## 拦截器（Interceptor）和过滤器（Filter）的执行顺序和区别

- 拦截器是基于java的反射机制的,aop实现，可以在方法前后，而过滤器是基于函数回调，需要在web.xml中配置。

- 拦截器不依赖与servlet容器，过滤器依赖与servlet容器。

- 拦截器可以多次被调用，还可以访问当前作用于的栈和对象，可以详细到每个方法，而过滤器器只能在请求的前后过滤

- 过滤器能够修改reques，而拦截器不行

- Filter的执行顺序在Interceptor之前，具体的流程见下图

  ![img](https://img-blog.csdn.net/20180511120357731?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rlc3Rjc19kbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

也就是

过滤器：能够拿到原始的http请求，但是拿不到你请求的控制器和请求控制器中的方法的信息

拦截器：可以拿到你请求的控制器和方法，拿不到请求方法的参数