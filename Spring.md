# 简述

Spring框架的目的是降低开发的复杂性。
Spring最核心的东西就是bean，bean其实很简单，就是一个对象，可以保存不同的属性。Spring框架中，要用到的东西都会封装到bean当中，使用时，调用这个bean就可以了。
这又引申出两个方面，创建bean和使用bean。

  - 创建bean就是通过各种xml的配置设置bean的属性，Spring内部有很多种解析xml的方法，通过方法就可以获得bean的属性。
  - 使用bean时，我们通过Factory的方法调用bean，这是设计模式当中工厂模式。Spring内部通过标准的方式生成了一个bean来使用。

## IoC和DI

创建bean和使用bean的时候又涉及到了IoC（控制反转）和DI（依赖注入）。

  - IoC说的是，没有使用Spring时我们调用bean时，需要自己new一个对象，现在是Spring通过里面的容器，各种Factory来new这个对象，控制权归容器所有。自己new的话过程叫正转，容器帮忙叫反转。
  - DI是在容器new对象的时候，这个对象可能有一些依赖关系，Spring容器当中有多种方法可以添加这个依赖关系。
  
这些功能都是一种思想，用来降低系统的耦合度，提高系统地性能。比如说通过单例模式减少资源开销，反复利用同一个资源，不用反复的new。通过容器管理bean，我们在调用的时候只需要声明调用的是啥，容器就会返回给你想要的，提高了效率。至于bean之间的复杂关系，交给容器去处理，bean本身相对独立。

## bean的生命周期

  - 实例化
  - 属性设置
  - 初始化
  - 销毁


Spring有两种IoC容器BeanFactory和ApplicationContext。

  - BeanFactory在启动的时候不会去实例化Bean，中有从容器中拿Bean的时候才会去实例化
  - ApplicationContext在启动的时候就把所有的Bean全部实例化了。它还可以为Bean配置lazy-init=true来让Bean延迟实例化

Spring还有一个重要的功能就是AOP（面向切面编程）。
编程中，对象与对象之间，方法与方法之间，模块与模块之间都是一个个切面。当需要给分散的行为引入公共的功能时，AOP就应运而生。通过IoC容器，生成代理监控目标类。代理就是设计模式中的代理模式。

# AOP 

  面向切面编程，处理系统中分布于各个模块的横切关注点。比如事务管理、日志、缓存等等。
  
  AOP主要分为静态代理和动态代理
  
    - 静态代理：代表为AspectJ，是由程序员创建的或特定工具自动生成的源码，在程序运行前，代理类的.class文件就已经存在了
    - 动态代理：代表为Spring AOP，在程序运行时，运用反射机制动态创建
    
  ## Spring AOP
     
  Spring AOP的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理。
  
  - JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口，核心是InvocationHandler接口和Proxy类
  - 如果目标类没有实现接口，Spring AOP会选择使用CGLIB来代理目标类。CGLIB是一个代码生成的类库，可以再运行时动态生成某个类的子类，通过继承的方式来实现，如果某个类被标记为final，则无法使用CGLIB代理
  
  ### JDK动态代理
  
  定义一个接口
  
  ~~~
  public interface TestService {
    public void run();
  }
  ~~~
  
  实现接口的类
  
  ~~~
  public class TestServiceImpl implements TestService {
    System.out.println("running");
  }
  ~~~
  
  自定义TestInvocationHandler类实现InvocationHandler接口，实现接口中的invoke方法，该方法中加入切面逻辑，method.invoke(target,args)执行目标类方法。
  
  ~~~
  public class TestInvocationHandler implements InvocationHandler {
    private Object target；
    
    //构造函数
    TestInvocationHandler(Object target){
      super();
      this.target = target;
    }
    
    //切面逻辑
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
      //执行前逻辑
      System.out.println("before------------");
      //执行程序
      Object result = method.invoke(target,args);
      //执行后逻辑
      System.out.println("after-------------");
      return result;
    }
  }
  ~~~
  
  测试类

  ~~~
  public class Test {
    public static void main(String[] args){
      TestService service = new TestServiceImpl();
      TestIncationHandler handler = new TestIncationHandler(service);
      
      //创建代理实例
      TestService serviceProxy = (TestService)Proxy.newProxyInstance(service.getClass().getClassLoader(),service.getClass().getInterfaces(),handler);
      
      //运行动态代理的执行程序
      serviceProxy.run();
    }
  }
  ~~~
  
  ### CGLIB动态代理
  
  目标类，CGLIB不需要定义目标类的统一接口
  
  ~~~
  public class Base {
    public void run(){
      System.out.println("running");
    }
  }
  ~~~
  
  实现动态代理
  
  ~~~
  public class CglibProxy implements MethodInterceptor {
    public Object intercept(Object object, Method method,Object[] args, MethodProxy proxy) throws Throwable {
      //执行前逻辑
      System.out.println("before------------");
      //执行程序
      proxy.invoke(object,args);
      //执行后逻辑
      System.out.println("after-------------");
      return null;
    }
  }
  ~~~
  
  增强的目标类的工厂
  
  ~~~
  public class Factory {
    //将目标类添加切入逻辑
    public static Base getInstance(CglibProxy proxy){
      Enhancer enhancer = new Enhancer();
      enhancer.setSuperclass(Base.class);
      
      //增强目标类调用的是代理对象中的intercepet方法
      enhancer.setCallback(proxy);
      //生成增强类
      Base base = (Base)enhancer.creat();
      return base;
    }
  }
  ~~~
  
  测试类
  
  ~~~
  public class Test {
    CglibProxy proxy = new CglibProxy();
    Base base = Factory.getInstance(proxy);
    base.run();
  }
  ~~~
  
