![1543843637384](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543843637384.png)

![1543843856274](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543843856274.png)

![1543839541638](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543839541638.png)

![1543839640970](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543839640970.png)

![1543839935122](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543839935122.png)

![1543841843480](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543841843480.png)

![1543841856358](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543841856358.png)

![1543842067782](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543842067782.png)



然后使用AnnotationConfigContext导入配置类Appconfig.class

Spring aop是在配置文件中进行介入，需要xml中进行复杂的配置

AspectJ是在编译时介入的，二者是aop的两种不同实现



**BeanPostProcessor(Bean后置处理器)，是一个接口**

在bean实例化之前调用，使得可以介入bean的实例化过程。AspectJ注解自动实现aop就是使用后置处理器实现，**将实际的bean进行调换，处理，返回一个代理的对象**。”狸猫换太子“

![1543842540517](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543842540517.png)





**ImportBeanDefinitionRegistar**

用于保证动态的加载注册一个bean，确定是否加载，使注解是否生效，即“动态开关”

定义一个类实现**ImportBeanDefinitionRegistar**接口即可，然后交给EnableAop注解确定是否生效。删掉@EnableAop注解即不生效

![1543843461317](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543843461317.png)

![1543843581176](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543843581176.png)





**IOC容器**

微观上本身相当于是一个**ConcurrentHashMap**，所有bean都放在这个map中

xxx.getBean()方法默认为getSingleton，即单例，直接从map.get()中直接获取这个bean，因为在初始化的时候就创建所有bean并放入了map中。调试的时候使用条件断点匹配相应的bean，找到singletonObjects.put方法

**aop底层机制**



**aop和ioc如何结合**