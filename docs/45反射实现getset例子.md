# 反射实现getset例子



```java
/**
 * 反射set方法
 * @param target target object
 * @param fieldName the name of the field
 * @param fieldValue the object of the value
 */
public static void setField(Object target, String fieldName, Object fieldValue) {
    Class cls = target.getClass();
    //判断该属性是否存在
    Field field;
    try {
        field = cls.getField(fieldName);
        if(field == null) {
            field = cls.getDeclaredField(fieldName);
        }
        if(field == null) {
            return;
        }
    }catch (NoSuchFieldException e) {
        e.printStackTrace();
    }
    
    String methodName = "set"+fieldName.substring(0, 1).toUpperCase()+(fieldName.length()>1?fieldName.substring(1):"");
    try{
        Method method = cls.getMethod(methodName, fieldvalue.getClass());
        method.invoke(target, fieldValue);
    }catch(NoSuchMethodException e) {
        e.printStackTrace();
    }catch(IllegalAccessException e) {
        e.printStackTrace();
    }catch(InvocationTargetException e) {
        e.printStackTrace();
    }
    
    
}

/**
 * 反射get方法
 * @param target target object
 * @param fieldName the name of the field
 * @param <T> the class of the field
 * @return the object of the field
 */    
public static <T> T getField(Object target, String fieldName) {
    Class cls = target.getClass();
    //判断属性是否存在
    //判断该属性是否存在
    Field field;
    try {
        field = cls.getField(fieldName);
        if(field == null) {
            field = cls.getDeclaredField(fieldName);
        }
        if(field == null) {
            return null;
        }
    }catch(NoSuchFieldException e) {
        e.printStackTrace();
    }
    
    String methodName = "set" + fieldName.substring(0, 1).toUpperCase() + (fieldName.length()>1?fieldName.substring(1):"");
    try {
        Method method = cls.getMethod(methodName);
        return (T)method.invoke(target);
    }catch (NoSuchMethodException e) {
        e.printStackTrace();
    }catch (IllegalAccessException e) {
        e.printStackTrace();
    }catch (InvocationTargetException e) {
        e.printStackTrace();
    }
    
    return null;
}
```



