# SpringBoot 和 Swing



## 使用上下文管理 Swing 的 JFrame 对象



注册上下文，装配 Bean

```java
@Configuration
public class SwingConfig {

    @Bean
    public JFrame loginJFrame() {
        return new JFrame;
    }
}
```



直接启动 SpringBoot 会报 java.awt.HeadlessException 异常，需要改造 SpringBoot 的 Main 方法

https://my.oschina.net/u/3768341/blog/1819081

```java
@SpringBootApplication
public class SwingApplication {

    public static void main(String[] args) {
        SpringApplicationBuilder builder = new SpringApplicationBuilder(SwingApplication.class);

        builder.headless(false);
        ConfigurableApplicationContext context = builder.run(args);
    }

}
```



## 项目启动后，直接弹出可视化窗口

https://blog.csdn.net/wohaqiyi/article/details/80571073

```java
@SpringBootApplication
public class CloudUtilsApplication {

    public static void main(String[] args){
     //SwingUtilities.invokeLater作用
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                createAndShowGUI();
            }
        });
        SpringApplication.run(CloudUtilsApplication.class,args);
    }
    private static void createAndShowGUI(){
        // 创建 JFrame 实例
        JFrame frame = new JFrame("Login Example");
        // Setting the width and height of frame
        frame.setSize(350, 200);
        
        //窗口关闭，springboot项目就会关掉
//        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.dispose(); //窗口关闭，springboot项目仍运行
        
        /* 
         * 创建面板，这个类似于 HTML 的 div 标签
         * 面板中我们可以添加文本字段，按钮及其他组件。
         */
        JPanel panel = new JPanel();
        
        frame.add(panel);// 添加面板
        
        placeComponents(panel);// 调用用户定义的方法并添加组件到面板

        frame.setVisible(true);// 设置界面可见
    }
    
}
```











