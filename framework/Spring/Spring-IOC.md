**1、Spring中Bean的作用域scope有哪些？**

（1）singleton（单例）：Spring IOC容器只会存在一个共享的bean【默认】

（2）prototype：每一次请求（即将其注入到另一个bean中）都会生成新的实例，相当于new的操作

（3）request：request表示针对每一次Http请求都会产生一个新的bean，同时该bean仅在当前Http request内有效

（4）session：session表示针对每一次Http请求都会产生一个新的bean，同时该bean仅在当前Http session内有效

（5）global session

