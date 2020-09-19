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
  
