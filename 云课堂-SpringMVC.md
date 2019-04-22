![1542370144402](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542370144402.png)

![1542373889425](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542373889425.png)

1.**scanFile(**):获取包路径：this.getClass.getClassLoader().getResource("/"+packageName.replaceAll(".","/"))

然后递归调用遍历文件夹，File.list()可获得一个文件夹下所有文件资源。然后找到class文件（File.indexOf("class")>0）,需要将HelloController.class ---->com.wyatt.controller.HelloController,获取到了所有类的包路径，包括xxxService类，用于后面@Autowire的自动注入，然后调用**instance()**进行实例化。

2.**instance()**:反射生成controller的实例，需要筛选出@Controller注解的类，即clazz.isAnnotationPresent(Controller.class)进行判断，RequestMapping类似，RequestMapping annotation=clazz.getAnnotation(RequetMapping.class)获得注解信息，然后放Map.put(annotation.value(),clazz.newInstance)放入**beanMap**

3.**ioc()**:对map进行遍历，拿到每一个value(类实例)获取到其中定义的**所有属性进行注入**，即Field[] fields=entry.getValue.getClass.getDeclaredFields()。

![1542375132491](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542375132491.png)

4.**urlMapping**:建立url和mehod的映射关系

![1542375183878](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542375183878.png)

5.然后即可在doPost()方法中进行反射调用

![1542375333821](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542375333821.png)



也就是在初始化的时候，需要先获得所有controller类的全限定名，并利用Class.forName()反射获得对应的实例，并保存下来，放在一个Map中。

**DispatcherServlet：**去寻找一个可用的HandlerMapping

**HandlerMapping**:用于找到处理方法的位置，也就是对一个handlerMapping的一个list进行遍历，找到一个对应的Handler同时返回一个封装的Handler（如UserController）和调用拦截器链，顺序执行则下一步。实现URL和method映射关系。

**HandlerAdapter**:如何调用对应的方法（即如何调用sayHello()这个方法），然后调用处理器对应的处理方法，即执行Handler返回一个ModelAndView。（包含model域这个map中的数据以及视图名，如success）。即进行反射调用处理方法

**ViewResolver**:调用ViewResolver进行视图解析，即success->success.jsp，即把一个逻辑视图名转化为实际视图，然后根据**Model域**的数据进行视图渲染，生成一个**View**对象并返回

转发核心方法：.**doDispatch**(HttpServletRequest, HttpServletResponse)

```
//getHandler方法中迭代遍历handlerMappings（list类型），返回一个对应的HandlerExecutionChain对象，其中包含了封装的真实handler对象

mappedHandler = this.getHandler(processedRequest);

//返回一个对应的handlerAdapter对象
HandlerAdapter err1 = this.getHandlerAdapter(mappedHandler.getHandler());

//根据返回的handlerexecutionChain获取定义的拦截器并执行

if(!mappedHandler.applyPreHandle(processedRequest, response)) {
                        return;
                    }
                    
//err是一个ModelAndView对象，通过反射调用真正的业务方法，handle方法等于调用HanlderAdater实现类
//RequestMappingHandlerAdapter.handleInternal()方法,见下       

err = err1.handle(processedRequest, response, mappedHandler.getHandler());
if(asyncManager.isConcurrentHandlingStarted()) {
                        return;
                    }
                    
//设置默认的view,即mv.setViewName(this.getDefaultViewName(request));
this.applyDefaultViewName(processedRequest, err);

//再执行拦截器链的postHandler（）方法
mappedHandler.applyPostHandle(processedRequest, response, err);
                    
//处理跳转返回值 ，找到对应的view
this.processDispatchResult(processedRequest, response, mappedHandler, err, (Exception)dispatchException);

//调用了DispatcherServlet.class中的resolveViewName（）和render()方法
//该方法获得了对应的视图解析器并对viewname进行解析得到实际的视图对象View
view = this.resolveViewName(mv.getViewName(), mv.getModelInternal(), locale, request);

//该方法调用了View接口的默认实现类进行填充model数据即渲染，然后转发个客户端
view.render(mv.getModelInternal(), request, response);

//然后调用view的抽象类方法AbstractView.render()
public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        if(this.logger.isTraceEnabled()) {
            this.logger.trace("Rendering view with name \'" + this.beanName + "\' with model " + model + " and static attributes " + this.staticAttributes);
        }

        Map mergedModel = this.createMergedOutputModel(model, request, response);
        this.prepareResponse(request, response);
        
        
        //为指定的模型指定视图，如果有必要的话，合并它静态的属性和RequestContext中的属性，renderMergedOutputModel() 执行实际的渲染。
        this.renderMergedOutputModel(mergedModel, this.getRequestToExpose(request), response);
    }
    
 //其中的renderMergedOutputModel()默认使用其实现类
 //org.springframework.web.servlet.view.InternalResourceView
 String dispatcherPath = prepareForRendering(requestToExpose, response);
 rd.forward(requestToExpose, response);

```

#### getHandler()方法

```
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    Iterator var2 = this.handlerMappings.iterator();

    HandlerExecutionChain handler;
    do {
        if(!var2.hasNext()) {
            return null;
        }

        HandlerMapping hm = (HandlerMapping)var2.next();
        if(this.logger.isTraceEnabled()) {
            this.logger.trace("Testing handler map [" + hm + "] in DispatcherServlet with name \'" + this.getServletName() + "\'");
        }

        handler = hm.getHandler(request);
    } while(handler == null);

    return handler;
}
```

#### applyPreHandle()方法

```
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HandlerInterceptor[] interceptors = this.getInterceptors();
    if(!ObjectUtils.isEmpty(interceptors)) {
        for(int i = 0; i < interceptors.length; this.interceptorIndex = i++) {
            HandlerInterceptor interceptor = interceptors[i];
            if(!interceptor.preHandle(request, response, this.handler)) {
                this.triggerAfterCompletion(request, response, (Exception)null);
                return false;
            }
        }
    }

    return true;
}
```

#### RequestMappingHandlerAdapter.handleInternal()方法

```
//第三个参数从handler变为handlerMethod是在其父类中AbstractHandlerMethodAdapter进行强转
// return this.handleInternal(request, response, (HandlerMethod)handler);

protected ModelAndView handleInternal(HttpServletRequest request, HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
    this.checkRequest(request);
    ModelAndView mav;
    if(this.synchronizeOnSession) {
        HttpSession session = request.getSession(false);
        if(session != null) {
            Object mutex = WebUtils.getSessionMutex(session);
            synchronized(mutex) {
            
            //调用invokeHandlerMethod（）中的invokeAndHandler（）进行了反射调用
            //ModelMap数据存放model域的值，都放在一个mavContainer对象中
                mav = this.invokeHandlerMethod(request, response, handlerMethod);
            }
        } else {
            mav = this.invokeHandlerMethod(request, response, handlerMethod);
        }
    } else {
        mav = this.invokeHandlerMethod(request, response, handlerMethod);
    }

    if(!response.containsHeader("Cache-Control")) {
        if(this.getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
            this.applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
        } else {
            this.prepareResponse(response);
        }
    }

    return mav;
}
```