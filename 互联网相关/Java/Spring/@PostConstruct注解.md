2021-09-06
16:39:00
author:陈建浩


--- 

PostConstruct注解
在Spring项目经常遇到@**PostConstruct**注解，首先介绍一下它的用途: **被注解的方法，在对象加载完依赖注入后执行。**

>Java中该注解的说明：@PostConstruct该注解被用来修饰一个非静态的void（）方法。被@PostConstruct修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器执行一次。PostConstruct在构造函数之后执行，init（）方法之前执行。
此注解是在Java EE5规范中加入的，在Servlet生命周期中有一定作用，它通常都是一些初始化的操作，但初始化可能依赖于注入的其他组件，所以要等依赖全部加载完再执行。

与之对应的还有@**PreDestroy**，在对象消亡之前执行，原理差不多，这里不做过多介绍。

**总体概括如上，注意其中几个点**

1. 要在依赖加载后，对象使用前执行，而且只执行一次，原因在上面已经说了。

2. 所有支持依赖注入的类都要支持此方法。

首先，我们可以看到这个注解是在javax.annotation包下的，也就是java拓展包定义的注解，并不是spring定义的，但至于为什么不在java包下，是因为java语言的元老们认为这个东西并不是java核心需要的工具，因此就放到扩展包里（javax中的x就是extension的意思），而spring是支持依赖注入的，因此spring必须要自己来实现@PostConstruct的功能。

**PostConstruct注释规则**

1. 除了拦截器这个特殊情况以外，其他情况都不允许有参数，否则spring框架会报**IllegalStateException**；而且返回值要是void，但实际也可以有返回值，至少不会报错，只会忽略

2. 方法随便你用什么权限来修饰，public、protected、private都可以，反正功能是由反射来实现

3. 方法不可以是static的，但可以是final的

所以，综上所述，在spring项目中，在一个bean的初始化过程中，方法执行先后顺序为

**Constructor > @Autowired > @PostConstruct**

先执行完构造方法，再注入依赖，最后执行初始化操作，所以这个注解就避免了一些需要在构造方法里使用依赖组件的尴尬。

以上是对@PostConstruct的简单介绍。