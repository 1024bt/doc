#### @PostConstruct
##### 定义：
  @PostConstruct是Java自带的注解，在方法上加该注解会在项目启动的时候执行该方法，也可以理解为在spring容器初始化的时候执行该方法。

  从Java EE5规范开始，[Servlet](https://so.csdn.net/so/search?q=Servlet&spm=1001.2101.3001.7020)中增加了两个影响Servlet生命周期的注解，@PostConstruct和@PreDestroy，这两个注解被用来修饰一个非静态的void（）方法。