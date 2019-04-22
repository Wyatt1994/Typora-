**mybatis mapper文件配置方式有3种：**

1.@selectProvider;

2.xml

3.@Select

**mybatis默认开启一级缓存，但结合spring就会失效**

spring和mybatis有哪些关联bean?

@MapperScan：扫描所有mapper接口类，先变为beanDefinition（在一个beanDefinitions集合中），其记录了bean的结构信息，配置信息；然后在processBeanDefiniton()方法中把**所有**beandefinition的class类型改为同一个mapperfactorybean类型（setbeanclass（）方法），通过传入各自的接口进行有参构造方法创建各自接口对应的mapperfactorybean对象。然后根据mapperfactorybean创建一个mapper对象，即调用factory.getObject()方法通过动态代理产生的对象；

SqlSessionFactorybean  



![1543408399406](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543408399406.png)

![1543409051236](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543409051236.png)

![1543409212136](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543409212136.png)

![1543409990505](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1543409990505.png)

先扫描mapperscan——>然后将接口类转为beandefinition(BeanDefinitionBuilder.getBeanDefinition()),并注册这个beanDefiniton(registry.registerBeanDefiniton()进行注册)——>lih





https://blog.csdn.net/qq_21441857/article/details/83015543

mapper接口的定义在bean加载阶段会被替换成**MapperFactoryBean类型**，在spring容器初始化的时候会给我们生成**MapperFactoryBean类型的对象**，在该对象生成的过程中调用afterPropertiesSet()方法，为我们生成了一个
MapperProxyFactory类型的对象存放于Configuration里的MapperRegistry对象中，同时**解析了mapper接口对应的xml文件，把每一个方法解析成一个MappedStatement对象**，存放于Configuration里的mappedStatements

###### 这个Map集合中。 



#### factorybean原理

我们调用getBean(Class requiredType)方法根据类型来获取容器中的bean的时候，对应我们的例子就是：根据类型com.zkn.spring.learn.service.FactoryBeanService来从Spring容器中获取Bean(**首先明确的一点是在Spring容器中没有FactoryBeanService类型的BeanDefinition**。但是却有一个Bean和FactoryBeanService这个类型有一些关系)。Spring在根据type去获取Bean的时候，会先获取到beanName。获取beanName的过程是：**先循环Spring容器中的所有的beanName，然后根据beanName获取对应的BeanDefinition，如果当前bean是FactoryBean的类型，则会从Spring容器中根据beanName获取对应的Bean实例，接着调用获取到的Bean实例的getObjectType方法获取到Class类型，判断此Class类型和我们传入的Class是否是同一类型**。如果是则返回测beanName，对应到我们这里就是：根据factoryBeanLearn获取到FactoryBeanLearn实例，调用FactoryBeanLearn的getObjectType方法获取到返回值FactoryBeanService.class。和我们传入的类型一致，所以这里获取的beanName为factoryBeanLearn。换句话说这里我们把factoryBeanLearn这个beanName映射为了：FactoryBeanService类型。即FactoryBeanService类型对应的beanName为factoryBeanLearn这是很重要的一点。在这里我们也看到了FactoryBean中三个方法中的一个所发挥作用的地。



如下，获取FactoryBeanService时没有匹配到，则扫描factoryBean，通过getObjectType成功匹配，然后返回mybatis的代理对象。

public class **FactoryBeanLearn** implements FactoryBean {

    @Override
    public Object getObject() throws Exception {
        //这个Bean是我们自己new的，这里我们就可以控制Bean的创建过程了
        return new FactoryBeanServiceImpl();
        //在mybatis中即根据mapper动态代理工厂返回一个动态代理对象
        mapperProxyFactory.getProxy()
    }
    
    @Override
    public Class<?> getObjectType() {
        return FactoryBeanService.class;
    }
    
    @Override
    public boolean isSingleton() {
        return true;
    }
