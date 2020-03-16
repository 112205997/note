# Spring Security学习



## 1.对于RestFul的理解

1. URL表示资源
2. 用http返回码表示状态
3. 使用Json交互
4. 是一种风格不是标准



## 2.springMVC与Mock相关

1.jsonpath表达式，github搜索

2.pageable与@PageableDefault

3.@pathVariable与@requestParam

4.URL声明使用正则   @requestMapping(value="/user/{id://d+}") 请求会返回4**状态码

5.@JsonView控制json输出内容 ,用于在不同的URL请求中屏蔽某些字段，例如密码

1. ​    使用接口声明多个视图，在实体类中声明

2. ​    在值对象的get方法上指定视图

3. ​    Controller方法上指定视图

   ![image-20200309111553376](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200309111553376.png)



![](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200309111640310.png

![image-20200309111716255](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200309111716255.png)

6 @RequestBody与日期类型参数处理  使用时间戳

7 @Valid注解与BindingResult验证请求参数的合法性  @notBlank这种注解要与@Valid结合使用才生效

​     BindingResult会将校验错误信息带进Controller方法中，即不直接拒绝请求4**，逻辑交由自己解决

![image-20200309114854072](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200309114854072.png)

8 rest的异常处理 自定义异常，统一异常处理

9 rest的切片拦截

1. ​	Filter 过滤器 实现servlet.Filter接口   方法有init()、destory()、doFilter()   

   ​	配置：在配置类加入FilterRegistrationBean 包括setFilter(),setUrlPatterns()设置过滤器和拦截URL

   ​    特点：只拦截http，并且不知道Spring控制器的处理

2. ​	Interceptor 拦截器 实现handlerInterceptor 方法有 preHandle() 、postHandle()异常不调用、afterCompletion()总是被调用

   配置：config类继承webMvcConfiguerAdapter,覆盖addInterceptors()方法，加入自定义的拦截器 

   ​      注意，同步和异步的拦截器处理不同，configureAsyncSupport()去配置异步拦截

   特点：比Filter多了个handler（强转HandlerMethod），即处理函数，但还是拿不到参数

   

3. ​	Aspect 切片 切片类（切入点、增强）使用@aspect注解

   特点：可以拿到处理函数的值和返回值，但拿不到原始的http请求数据

   ![image-20200310120553900](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200310120553900.png)

10 文件上传与下载

11 异步处理

​    Runnable--

![image-20200310171332219](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200310171332219.png)

​		使用Callable  副线程是主线程开启的



​	DeferredResult

​       业务逻辑不是在一个线程里处理的

​      ![image-20200310171408631](/Users/weijiniwei/Library/Application Support/typora-user-images/image-20200310171408631.png)

12 前后端开发效率工具 swagger2(文档生成)  wireMock(服务器模拟)