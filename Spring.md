#### IOC容器初始化

IOC容器初始化是在refresh()方法中启动，实现了beanDefinition的Resource**定位、载入和注册**三个过程。

**定位**：即将xml文件读入并转为Resource类型。

**载入**：解析xml文件，生成BeanDefinition对象

**注册**：即将所有的BeanDefinition与其定义的id，name等一一对应放入hashMap中

#### 依赖注入

依赖注入在调用getBean()时才开始，也可以通过设置lay-init属性在ioc初始化时完成

依赖注入时，即getBean()方法调用，然后调用**createBean()**，相应的注入对象bean根据BeanDefinition的定义进行生成，同时进行了初始化，例如初始方法和后置处理器配置。

#### Spring中如何由BeanDefinition实例化为bean对象呢？

1.若定义了工厂方法，则调用工厂方法实例化；

2.若定义了构造器注入，则调用相应构造方法。

3.否则使用默认的实例化策略：1）**CGlib的字节码技术**生成和转换java字节码，实现bean实例化；2）通过beanUtlls，使用**jvm的反射功能**获取构造器进行实例化。

#### ***Aware接口

某个bean实现了该接口，则可以通过这个bean获取到其在ioc容器中的真实beanName,applicationcontext等**容器内部属性**