# SpringMVC源码分析

主要类型和关系：

DispatcherServlet —> HandlerMapping 

DispatcherServlet —> HandlerAdapter —> HttpMessageConveter



DispatcherServlet ---> Servlet 1.init(ServletConfig)

​													2.service(ServletRequest, ServletResponse)

​													3.destroy

## 执行流程：

用户请求 DispatcherServlet

`DispatcherServlet` 查找处理器映射 `HandlerMapping`（一个RequestMapping就是一个HandlerMapping），

通过 `HandlerMapping` 请求Handler，即调用处理适配器 `HandlerAdapter` ，

`HandlerAdapter` 调用 `Controller` 返回 `ModelAndView` 。

DispatcherServlet 拿到 MAV 后，调用视图解析器 ViewResovlter，获得View。

最后 解析View，渲染视图信息并返回页面。



关于 @RequestMapping 和 HandlerMapping：

`@RequestMapping` 表示一个请求地址，注解的类名、方法名、注解参数（uri）组成 `HandlerMapping` ，并注入到 DispatcherServlet 存 HandlerMapping的List 中。

请求时遍历找到和请求一致的HandlerMapping。



## 源码分析：

起点是DispatcherServlet.doService。

核心是DispatcherServlet.**doDispatch**方法

```java
/**
	 * Process the actual dispatching to the handler.
	 * <p>The handler will be obtained by applying the servlet's HandlerMappings in order.
	 * The HandlerAdapter will be obtained by querying the servlet's installed HandlerAdapters
	 * to find the first that supports the handler class.
	 * <p>All HTTP methods are handled by this method. It's up to HandlerAdapters or handlers
	 * themselves to decide which methods are acceptable.
	 * @param request current HTTP request
	 * @param response current HTTP response
	 * @throws Exception in case of any kind of processing failure
	 */
	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

        // 判断是否异步请求
		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
                // 获得对应HandlerMapping
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

                // interceptors
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}
```



### doDispatch 的逻辑：

####  getHandler(request)

`getHandler(request)` 获得 request 对应的 HandlerMapping。

```java
/**
 * Return the HandlerExecutionChain for this request.
 * <p>Tries all handler mappings in order.
 * @param request current HTTP request
 * @return the HandlerExecutionChain, or {@code null} if no handler could be found
 */
@Nullable
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
   if (this.handlerMappings != null) {
      for (HandlerMapping mapping : this.handlerMappings) {
         HandlerExecutionChain handler = mapping.getHandler(request);
         if (handler != null) {
            return handler;
         }
      }
   }
   return null;
}
```

然后，

判断 `HandlerExecutionChain` （包含Controller类和具体方法），

如果没找到，`noHandlerFound(){response.sendError(404);}` ，返回404错误。

如果找到，

**获取 HandlerAdapter** = `getHandlerAdapter(mappedHandler.getHandler());` 并返回。

**然后处理拦截器** `mappedHandler.applyPreHandle(processedRequest, response)` 

通过 `HandlerAdapter` 处理得到 `ModelAndView` 



#### HandlerAdapter.handle(request,response,handler) 

 `mv = ha.handle(processedRequest, response, mappedHandler.getHandler());`

通过 `HandlerAdapter` 处理得到 `ModelAndView` 。



`ha.handle(processedRequest, response, mappedHandler.getHandler());` 调用真正的处理逻辑：

`RequestMappingHandlerAdapter` 的 `handleInternal` 方法，源码如下：

```java
@Override
	protected ModelAndView handleInternal(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

		ModelAndView mav;
		checkRequest(request);

		// Execute invokeHandlerMethod in synchronized block if required.
		if (this.synchronizeOnSession) {
			HttpSession session = request.getSession(false);
			if (session != null) {
				Object mutex = WebUtils.getSessionMutex(session);
				synchronized (mutex) {
					mav = invokeHandlerMethod(request, response, handlerMethod);
				}
			}
			else {
				// No HttpSession available -> no mutex necessary
				mav = invokeHandlerMethod(request, response, handlerMethod);
			}
		}
		else {
			// No synchronization on session demanded at all...
			mav = invokeHandlerMethod(request, response, handlerMethod);
		}

		if (!response.containsHeader(HEADER_CACHE_CONTROL)) {
			if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
				applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
			}
			else {
				prepareResponse(response);
			}
		}

		return mav;
	}
```



##### invokeHandlerMethod

执行 HandlerMethod ：`mav = invokeHandlerMethod(request, response, handlerMethod);` 

 HandlerMethod：存Controller类和对应的方法名。

```java
/**
	 * Invoke the {@link RequestMapping} handler method preparing a {@link ModelAndView}
	 * if view resolution is required.
	 * @since 4.2
	 * @see #createInvocableHandlerMethod(HandlerMethod)
	 */
	@Nullable
	protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

		ServletWebRequest webRequest = new ServletWebRequest(request, response);
		try {
			WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
			ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

			ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
			if (this.argumentResolvers != null) {
				invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
			}
			if (this.returnValueHandlers != null) {
				invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
			}
			invocableMethod.setDataBinderFactory(binderFactory);
			invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);

			ModelAndViewContainer mavContainer = new ModelAndViewContainer();
			mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
			modelFactory.initModel(webRequest, mavContainer, invocableMethod);
			mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);

			AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
			asyncWebRequest.setTimeout(this.asyncRequestTimeout);

			WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
			asyncManager.setTaskExecutor(this.taskExecutor);
			asyncManager.setAsyncWebRequest(asyncWebRequest);
			asyncManager.registerCallableInterceptors(this.callableInterceptors);
			asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

			if (asyncManager.hasConcurrentResult()) {
				Object result = asyncManager.getConcurrentResult();
				mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
				asyncManager.clearConcurrentResult();
				LogFormatUtils.traceDebug(logger, traceOn -> {
					String formatted = LogFormatUtils.formatValue(result, !traceOn);
					return "Resume with async result [" + formatted + "]";
				});
				invocableMethod = invocableMethod.wrapConcurrentResult(result);
			}

            // 执行逻辑后，设置好mavContainer
			invocableMethod.invokeAndHandle(webRequest, mavContainer);
			if (asyncManager.isConcurrentHandlingStarted()) {
				return null;
			}

			return getModelAndView(mavContainer, modelFactory, webRequest);
		}
		finally {
			webRequest.requestCompleted();
		}
	}
```



###### invocableMethod.invokeAndHandle

核心方法是： `invocableMethod.invokeAndHandle(webRequest, mavContainer);`

```java
/**
	 * Invoke the method and handle the return value through one of the
	 * configured {@link HandlerMethodReturnValueHandler HandlerMethodReturnValueHandlers}.
	 * @param webRequest the current request
	 * @param mavContainer the ModelAndViewContainer for this request
	 * @param providedArgs "given" arguments matched by type (not resolved)
	 */
	public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

        // 执行Controller的方法后的返回值
		Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
		setResponseStatus(webRequest);

		if (returnValue == null) {
			if (isRequestNotModified(webRequest) || getResponseStatus() != null || mavContainer.isRequestHandled()) {
				disableContentCachingIfNecessary(webRequest);
				mavContainer.setRequestHandled(true);
				return;
			}
		}
		else if (StringUtils.hasText(getResponseStatusReason())) {
			mavContainer.setRequestHandled(true);
			return;
		}

		mavContainer.setRequestHandled(false);
		Assert.state(this.returnValueHandlers != null, "No return value handlers");
		try {
            // 返回值：处理的方式和名称（例如：Method,login），Model and View的container，uri 
			this.returnValueHandlers.handleReturnValue(
					returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
		}
		catch (Exception ex) {
			if (logger.isTraceEnabled()) {
				logger.trace(formatErrorForReturnValue(returnValue), ex);
			}
			throw ex;
		}
	}
```

invocableMethod.invokeAndHandle 的逻辑：

核心方法是：`Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);` 

方法剩下的逻辑是处理 returnValue 的。



invokeForRequest 源码：

```java
/**
	 * Invoke the method after resolving its argument values in the context of the given request.
	 * <p>Argument values are commonly resolved through
	 * {@link HandlerMethodArgumentResolver HandlerMethodArgumentResolvers}.
	 * The {@code providedArgs} parameter however may supply argument values to be used directly,
	 * i.e. without argument resolution. Examples of provided argument values include a
	 * {@link WebDataBinder}, a {@link SessionStatus}, or a thrown exception instance.
	 * Provided argument values are checked before argument resolvers.
	 * <p>Delegates to {@link #getMethodArgumentValues} and calls {@link #doInvoke} with the
	 * resolved arguments.
	 * @param request the current request
	 * @param mavContainer the ModelAndViewContainer for this request
	 * @param providedArgs "given" arguments matched by type, not resolved
	 * @return the raw value returned by the invoked method
	 * @throws Exception raised if no suitable argument resolver can be found,
	 * or if the method raised an exception
	 * @see #getMethodArgumentValues
	 * @see #doInvoke
	 */
	@Nullable
	public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

        //找到传进来的参数信息
		Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
		if (logger.isTraceEnabled()) {
			logger.trace("Arguments: " + Arrays.toString(args));
		}
		return doInvoke(args);
	}

    /**
	 * Invoke the handler method with the given argument values.
	 */
	@Nullable
	protected Object doInvoke(Object... args) throws Exception {
		ReflectionUtils.makeAccessible(getBridgedMethod());
		try {
            // 核心：调用 Method的invoke方法
			return getBridgedMethod().invoke(getBean(), args);
		}
		catch (IllegalArgumentException ex) {
			assertTargetBean(getBridgedMethod(), getBean(), args);
			String text = (ex.getMessage() != null ? ex.getMessage() : "Illegal argument");
			throw new IllegalStateException(formatInvokeError(text, args), ex);
		}
		catch (InvocationTargetException ex) {
			// Unwrap for HandlerExceptionResolvers ...
			Throwable targetException = ex.getTargetException();
			if (targetException instanceof RuntimeException) {
				throw (RuntimeException) targetException;
			}
			else if (targetException instanceof Error) {
				throw (Error) targetException;
			}
			else if (targetException instanceof Exception) {
				throw (Exception) targetException;
			}
			else {
				throw new IllegalStateException(formatInvokeError("Invocation failure", args), targetException);
			}
		}
	}
```

Method的invoke：

```java
@CallerSensitive
    public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException
    {
        if (!override) {
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class<?> caller = Reflection.getCallerClass();
                checkAccess(caller, clazz, obj, modifiers);
            }
        }
        MethodAccessor ma = methodAccessor;             // read volatile
        if (ma == null) {
            ma = acquireMethodAccessor();
        }
        return ma.invoke(obj, args);
    }
```



handleReturnValue 源码：

处理 returnValue，设置 mavContainer 

```java
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
			ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {

		HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType);
		if (handler == null) {
			throw new IllegalArgumentException("Unknown return value type: " + returnType.getParameterType().getName());
		}
		handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);
	}

@Nullable
	private HandlerMethodReturnValueHandler selectHandler(@Nullable Object value, MethodParameter returnType) {
		boolean isAsyncValue = isAsyncReturnValue(value, returnType);
		for (HandlerMethodReturnValueHandler handler : this.returnValueHandlers) {
			if (isAsyncValue && !(handler instanceof AsyncHandlerMethodReturnValueHandler)) {
				continue;
			}
			if (handler.supportsReturnType(returnType)) {
                // 例如：ViewNameMethodReturnValueHandler
				return handler;
			}
		}
		return null;
	}
```



ViewNameMethodReturnValueHandler 的 handleReturnValue ：

具体的处理 returnValue，设置 mavContainer 逻辑。

```java
@Override
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
			ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {

		if (returnValue instanceof CharSequence) {
			String viewName = returnValue.toString();
			mavContainer.setViewName(viewName);
			if (isRedirectViewName(viewName)) {
                // 判断并执行 return "redirect:/index" ，跳转页面的情况
				mavContainer.setRedirectModelScenario(true);
			}
		}
		else if (returnValue != null) {
			// should not happen
			throw new UnsupportedOperationException("Unexpected return type: " +
					returnType.getParameterType().getName() + " in method: " + returnType.getMethod());
		}
	}
```



###### getModelAndView

RequestMappingHandlerAdapter#getModelAndView：

通过 mavContainer 获得 ModelAndView 并返回。完成 HandlerAdapter.handle 的逻辑。

```java
private ModelAndView getModelAndView(ModelAndViewContainer mavContainer,
			ModelFactory modelFactory, NativeWebRequest webRequest) throws Exception {

		modelFactory.updateModel(webRequest, mavContainer);
		if (mavContainer.isRequestHandled()) {
			return null;
		}
		ModelMap model = mavContainer.getModel();
		ModelAndView mav = new ModelAndView(mavContainer.getViewName(), model, mavContainer.getStatus());
		if (!mavContainer.isViewReference()) {
			mav.setView((View) mavContainer.getView());
		}
		if (model instanceof RedirectAttributes) {
			Map<String, ?> flashAttributes = ((RedirectAttributes) model).getFlashAttributes();
			HttpServletRequest request = webRequest.getNativeRequest(HttpServletRequest.class);
			if (request != null) {
				RequestContextUtils.getOutputFlashMap(request).putAll(flashAttributes);
			}
		}
		return mav;
	}
```



#### processDispatchResult

 `processDispatchResult` 处理结果。



















## 拦截器与过滤器

当一个请求进来的时候，会先执行各种filter，过滤掉最终需要的请求，然后会落到DispatcherServlet中的**doService()**方法。该方法是预先设置一些特殊请求参数，然后再转发给**doDispatch()**做真正的处理转发。

通过 handlerMapping获得 handleradapter 后，处理拦截器，再通过Class获得Method.invoke(controllerBean, args)。



TODO：源码分析

拦截器：

在 doDispatch() 的 getHandler 时，`AbstractHandlerMapping` 的 `getHandlerExecutionChain` 方法里，加载所有 HandlerInterceptor 的实现。自定义拦截器继承 `HandlerInterceptorAdapter`。



过滤器：

在 doService() 前调用 FilterChain的doFilter。自定义过滤器实现 `Filter`。



filter 对 servlet进行request过滤，interceptor 对 mvc上下文过程的拦截。



具体方法和对应进度：

dofilter 对 servlet的request过滤，chain.doFilter(req, res)放行。

doService，doDispath，preHandle()拦截，handlerAdapter.handle执行controller。

postHandle()拦截，再返回ModeAndView。

afterCompletion()拦截，dofilter 对 servlet的response过滤。



Filter对servlet的请求和响应过滤，urlPatterns = "/*"设post置使用过滤器的url。

Interceptor 对 MVC上下文的拦截，分别是 执行controller前，preHandle；执行后，postHandle；

视图渲染后，afterCompletion。



参考资料：

手把手带你撕、拉、扯下 SpringMVC 的外衣: https://www.cnblogs.com/Java-Starter/p/10352802.html

https://www.cnblogs.com/Java-Starter/p/10444617.html#autoid-1-0-0

https://juejin.im/entry/6844903473960452104

SpringMVC源码阅读：异常解析器 https://www.cnblogs.com/Java-Starter/p/10356055.html

Spring Interceptor vs Filter: https://juejin.im/post/6844903828970553358

Servlet 到 Spring MVC 的简化之路: 

https://juejin.im/post/6844903570681135117

Tomcat和Spring什么关系？SpringMVC和Servlet什么关系？

https://zhuanlan.zhihu.com/p/65658315



### 2020.08.12 补充 DispatcherServlet前发生了什么？

### HandlerAdapter 和 Controller有哪些种类？

Web容器首先调用HttpServlet的service方法，并根据请求的类型调用doGet或doPost方法。

servlet: service(), doGet(), doPost(), doService(), doDispatch



#### 关于 servlet：

单例

创建一次，初始化一次，销毁一次



#### 关于 Tomcat 与 spring mvc

Tomcat 一个叫Mapper的类，每一个URL要交给哪个Servlet处理，具体的映射规则都由一个映射器决定。默认都配置了JSPServlet(*.jsp)和DefaultServlet(/)处理JSP和静态资源。

springmvc 的 DispatcherServlet配置/*，会覆盖处理所有 Tomcat请求。



filters -> dispatcherServlet -> handlerMapping(url:controllrt.method) -> interceptors -> handleradapter -> handler controller



#### 关于 handlermapping、handleradapter、handler

HandlerMapping 拿到 url对应 handler，

通过 handler搜索支持的 handleradapter，

通过 handleradapter 执行 handler。

实现 handler：

1. 实现 Controller接口：implements Controller
2. 实现 httpRequestHandler接口：implements HttpRequestHandler
3. @RequestMapping 



Spring MVC有三种映射策略：HandlerMapping

简单url映射 -> SimpleUrlHandlerMapping
BeanName映射 -> BeanNameUrlHandlerMapping
@RequestMapping映射 -> RequestMappingHandlerMapping



url映射：

```java
@Bean
public SimpleUrlHandlerMapping simpleUrlHandlerMapping() {
    SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
    Map<String, Object> urlMap = new HashMap<>();
    urlMap.put("/indexV2", "indexController");
    mapping.setUrlMap(urlMap);
    return mapping;
    }
```



BeanName映射：

@Component("/index/user")
public class AController implements Controller



@RequestMapping映射：

1. spring容器在启动的时候，会拿到所有的bean，判断这个bean上是否有@Controller或者@RequestMapping注解，如果有则执行后面的步骤
2. 解析类上的@RequestMapping注解，将其信息封装为RequestMappingInfo
3. 将RequestMappingInfo及其对应的HandlerMethod注册到mappingLookup中
4. 如果@RequestMapping指定的url没有通配符，则将url -> RequestMappingInfo注册到urlLookup中



HandlerAdapter：

当通过HandlerMapping找到Handler后，会依次调用handlerAdapters的supports方法，找到第一个返回true的HandlerAdapter，然后调用HandlerAdapter的handle方法，完成执行。



HttpRequestHandlerAdapter -> 执行实现了HttpRequestHandler接口的Handler
SimpleControllerHandlerAdapter -> 执行实现了Controller接口的Handler
RequestMappingHandlerAdapter -> 执行Handler类型是HandlerMethod及其子类的Handler。

> RequestMappingHandlerMapping返回的Handler是HandlerMethod类型



HttpRequestHandlerAdapter:

handle 方法 强转 handler为HttpRequestHandler，执行handler.handleRequest



SimpleControllerHandlerAdapter:

强转 handler为SimpleControllerHandlerAdapter



RequestMappingHandlerAdapter支持Handler类型是HandlerMethod

RequestMappingHandlerMapping返回的Handler是HandlerMethod







参考资料：

springmvc中handlermapping与handleradapter：

https://zhuanlan.zhihu.com/p/158226786



https://juejin.im/post/6844904048810803207#hhandlerhandlermapping





















