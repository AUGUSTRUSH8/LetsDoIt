### Java日志使用姿势

应用程序日志对于代码维护有至关重要的作用，我们希望通过日志去发现问题、定位问题、解决问题，一般而言，日志分为两种，业务日志和异常日志，使用日志我们希望能达到以下目标：

- 对程序运行情况的记录和监控；
- 在必要时可详细了解程序内部的运行状态；
- 对系统性能的影响尽量小

#### Java日志框架

1. Log4j 或 Log4j 2 - Apache的开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件、甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等；用户也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，用户能够更加细致地控制日志的生成过程。这些可以通过一个配置文件（XML或Properties文件）来灵活地进行配置，而不需要修改程序代码。Log4j 2则是前任的一个升级，参考了Logback的许多特性；
2. Logback - Logback是由log4j创始人设计的又一个开源日记组件。logback当前分成三个模块：logback-core,logback- classic和logback-access。logback-core是其它两个模块的基础模块。logback-classic是log4j的一个改良版本。此外logback-classic完整实现SLF4J API使你可以很方便地更换成其它日记系统如log4j或JDK14 Logging；
3. java.util.logging - JDK内置的日志接口和实现，功能比较简单；
4. Slf4j - SLF4J是为各种Logging API提供一个简单统一的接口），从而使用户能够在部署的时候配置自己希望的Logging API实现；
5. Apache Commons Logging - Apache Commons Logging （JCL）希望解决的问题和Slf4j类似。
   选项太多了的后果就是选择困难症，一般的原则是没有最好的，只有最合适的。**在比较关注性能的地方，选择Logback或自己实现高性能Logging API可能更合适；在已经使用了Log4j的项目中，如果没有发现问题，继续使用可能是更合适的方式； 如果不想有依赖则使用java.util.logging或框架容器已经提供的日志接口**。

#### Java日志最佳实践

**定义日志变量：**

日志变量往往不变，最好定义成final static，变量名用大写。

**日志分级：**

Java的日志框架一般会提供以下日志级别，缺省打开info级别，也就是debug，trace级别的日志在生产环境不会输出，在开发和测试环境可以通过不同的日志配置文件打开debug级别。

- error - 其他错误运行期错误；
- warn - 警告信息，如程序调用了一个即将作废的接口，接口的不当使用，运行状态不是期望的 - 但仍可继续处理等；
- info - 有意义的事件信息，如程序启动，关闭事件，收到请求事件等；
- debug - 调试信息，可记录详细的业务处理到哪一步了，以及当前的变量状态；
- trace - 更详细的跟踪信息；

在程序里要合理使用日志分级:

```java
LOGGER.debug("entering getting content");
String content =CacheManager.getCachedContent();
if(content == null){

    //使用warn，因为程序还可以继续执行，但类似警告太多可能说明缓存服务不可用了，值得注意
    LOGGER.warn("Got empty content from cache,need perform database lookup");

    Connection conn = ConnectionFactory.getConnection();
    if (conn=null) {
    LOGGER.error("Can't get database connection, failed to return content");//尽量提供详细的信息，知道错误的原因，而不能简单的写logger.error("failed")       
    }else{
      try{    
    　　　content = conn.query(...);    
 　　　　}catch ( IOException e ){
                   //异常要记录错误堆栈
                   LOGGER.error("Failed to perform database lookup", e );
                }finally{
                   ConnectionFactory.releaseConnection(conn);
                 }
           }
}  
//调试的时候可以知道方法的返回了
LOGGER.debug("returning content: "+ content);
return content;
```

#### 基本的Logger编码规范

1.在一个对象中通常只使用一个Logger对象，Logger应该是static final的，只有在少数需要在构造函数中传递logger的情况下才使用private final。

```java
static final logger_LOG=loggerFactory.getLogger(Main.class);
```

2.输出Exceptions的全部Throwable信息，因为logger.error(msg)和logger.error(msg,e.getMessage())这样的日志输出方法会丢失掉最重要的StackTrace信息。

```java
void foo(){
     try {
            //  do something...
      }catch ( Exception e ){
          _LOG.error(e.getMessage()); // 错误
          _LOG.error("Bad things : ",e.getMessage()); // 错误
          _LOG.error("Bad things : ",e); // 正确
      }
}
```

3.不允许记录日志后又抛出异常，因为这样会多次记录日志，只允许记录一次日志。

```java
void foo() throws LogException{
     try{
         // do something...
     }catch ( Exception e ){
          _LOG.error("Bad things : ", e);
          throw new LogException("Bad things : ",e);
       }
}
```

4.不允许出现System print(包括System.out.println和System.error.println)语句。

```java
void foo() {
        try{
               // do something...
        }catch( Exception e ){ 
             System.out.println(e.getMessage()); // 错误
             System.err.println(e.getMessage()); // 错误
             _LOG.error("Bad things : ",e );  // 正确
          }
 }
```

5.不允许出现printStackTrace。

```java
void foo() {
     try {
             // do something...
     }catch ( Exception e ) {
            e.printStackTrace(); // 错误
            _LOG.error("Bad things : ", e ); //正确
         }
}
```

6.日志性能的考虑，如果代码为核心代码，执行频率非常高，则输出日志建议增加判断，尤其是低级别的输出

```java
if (LOGGER.isDebugEnabled ()) {
    LOGGER.debug("returning content: "+ content);
}
```

但更好的方法是Slf4j提供的最佳实践:

```java
LOGGER.debug("returning content: {}", content);
```

一方面可以减少参数构造的开销，另一方面也不用多写两行代码。

7.有意义的日志

通常情况下在程序日志里记录一些比较有意义的状态数据：程序启动，退出的时间点；程序运行消耗时间；耗时程序的执行进度；重要变量的状态变化。

除此之外，在公共的日志里规避打印程序的调试或者提示信息。