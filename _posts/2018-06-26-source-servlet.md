---
layout: post
title: '看源码之servlet'
subtitle: '看源码之servlet'
date: 2018-06-26
categories: source
author: yates
cover: ''
tags: java
---

### 前言
早期web应用主要浏览新闻等静态页面，http服务器向浏览器返回静态HTML。浏览器负责解析HTML，并将结果呈现
而后随着互联网发展，为了通过一些交互获取动态结果，需要HTTP服务器调用服务端程序，比如servlet，而servlet不能独立运行，必须部署到servlet容器中。
servlet作为服务器和客户端之间交互，响应请求，基于组件，独立于平台的服务端常用的程序，大多数servlet只用来扩张http协议的web服务器

Servlet容器用来加载和管理业务类。HTTP服务器不直接跟业务类打交道，而是把请求交给Servlet容器去处理，Servlet容器会将请求转发到具体的Servlet，
如果这个Servlet还没创建，就加载并实例化这个Servlet，然后调用这个Servlet的接口方法。因此Servlet接口其实是Servlet容器跟具体业务类之间的接口。


### Servlet请求流程：
资源请求-->HTTP服务器封装请求信息到ServletRequest对象-->servlet容器调用service方法-->根据请求URL和servlet映射关系，找到相应servlet（如果未加载则利用反射创建servlet，调用init初始化）
-->调用servlet的service处理请求-->返回ServletResponse对象给HTTP服务器-->服务器返回给客户端

Servlet容器会实例化和调用Servlet，以Web应用程序的方式来注册，部署Servlet到Servlet容器中。根据Servlet规范，Web应用程序有一定的目录结构用于加载Servlet

- MyWebApp
	- WEB-INF/web.xml 	
	- WEB-INF/lib
	- WEB-INF/classes
	- META-INF

Servlet规范servlet容器在启动时会加载web应用，创建ServletContext对象。ServletContext是一个全局对象，所有servlet可通过此共享数据（web应用初始化数据，文件资源，servlet请求转发等等）

## filter Listener
servlet规范是开发者专注于业务，但是由于规范导致的个性化需求受到限制，我们可以通过filter和 listener来实现

filter 工作流程：web应用部署完成后，servlet容器实例化Filter并把filter链接成一个filterchain，依次调用
listener 工作流程：web应用在servlet容器中运行，servlet容器内部发生各种事件，servlet容器提供一些默认监听器监听这些事件，也可以在web.xml中自定义监听器。

前者是干预过程的，是基于过程行为的；后者是基于状态，任何行为改变同一个状态，触发事件是一致的

**springMVC容器，spring容器，servlet容器**
tomcat等服务器启动时创建全局上下文环境servletcontext，期间触发容器初始化事件，Spring的ContextLoaderListener监听到后，调用contextInitialized方法初始化全局spring根容器（IOC容器）并放入servletcontext中。
tomcat还会扫描servlet，比如springmvc的dispatchservlet，dispatchservlet初始化时建立自己容器也就是springmvc容器，他可以通过servletcontext拿到spring根容器，并将其设置为springmvc父容器，这也就是为什么controller能访问service服务，而service不可以访问controller

https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/tomcat/1.jpg


servlet接口源码
```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
```

ServletConfig接口源码
```java
public interface ServletConfig {
    String getServletName();

    // 获取servlet上下文
    ServletContext getServletContext();

    String getInitParameter(String var1);

    Enumeration<String> getInitParameterNames();
}
```

ServletContext接口源码
```java
public interface ServletContext {
    String TEMPDIR = "javax.servlet.context.tempdir";
    String ORDERED_LIBS = "javax.servlet.context.orderedLibs";

    String getContextPath();

    ServletContext getContext(String var1);

    int getMajorVersion();

    String getMimeType(String var1);

    Set<String> getResourcePaths(String var1);

    URL getResource(String var1) throws MalformedURLException;

    InputStream getResourceAsStream(String var1);

    RequestDispatcher getRequestDispatcher(String var1);

    RequestDispatcher getNamedDispatcher(String var1);

    void log(String var1);
 
    void log(String var1, Throwable var2);

    String getRealPath(String var1);

    String getServerInfo();

    String getInitParameter(String var1);

    Enumeration<String> getInitParameterNames();

    boolean setInitParameter(String var1, String var2);

    Object getAttribute(String var1); 

    void setAttribute(String var1, Object var2);

    void removeAttribute(String var1);

    String getServletContextName();

    Dynamic addServlet(String var1, String var2); 

    <T extends Servlet> T createServlet(Class<T> var1) throws ServletException;

    ServletRegistration getServletRegistration(String var1); 

    javax.servlet.FilterRegistration.Dynamic addFilter(String var1, String var2);
 
    <T extends Filter> T createFilter(Class<T> var1) throws ServletException;

    FilterRegistration getFilterRegistration(String var1);

    Map<String, ? extends FilterRegistration> getFilterRegistrations();

    SessionCookieConfig getSessionCookieConfig();

    void setSessionTrackingModes(Set<SessionTrackingMode> var1);

    Set<SessionTrackingMode> getDefaultSessionTrackingModes();

    Set<SessionTrackingMode> getEffectiveSessionTrackingModes();

    void addListener(String var1);

    <T extends EventListener> void addListener(T var1);

    void addListener(Class<? extends EventListener> var1);

    <T extends EventListener> T createListener(Class<T> var1) throws ServletException;

    JspConfigDescriptor getJspConfigDescriptor();

    ClassLoader getClassLoader();

    void declareRoles(String... var1);

    String getVirtualServerName();
}
```

GenericServlet抽象类实现了servlet的接口，并实现了些通用的方法（**大多数方法都是通过调用ServletConfig方法**获取）；
```java
public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {
    private static final String LSTRING_FILE = "javax.servlet.LocalStrings";
    private static ResourceBundle lStrings = ResourceBundle.getBundle("javax.servlet.LocalStrings");
    private transient ServletConfig config;
 
    public String getInitParameter(String name) 
    Enumeration<String> getInitParameterNames()
    
    public ServletContext getServletContext() {
        ServletConfig sc = this.getServletConfig();
        if (sc == null) {
            throw new IllegalStateException(lStrings.getString("err.servlet_config_not_initialized");
        } else {
            return sc.getServletContext();
        }
    }
    
    public void init(ServletConfig config)
    public void log(String message, Throwable t)
    public String getServletName()
    
    // **强制继承类实现该方法**
    public abstract void service(ServletRequest var1, ServletResponse var2)
```

HttpServlet继承了GenericServlet，主要针对**http协议**交互的两端进行通信。**HttpServletRequest，HttpServletResponse，HttpSession**作为通信的载体

针对HttpSession，HttpServletRequest，HttpServletResponse提供listener和event,wrapper,attributeEvent,attributeListener等相关接口，类
 
我们在请求服务器或得到服务响应的时候有很多请求是需要进行过滤，把内容进行修改或拦截

filter接口源码 和servlet类似一样的生命周期,同样和servlet一样提供chain进行链式调用，提供config调用servletContext
```java
public interface Filter {
    void init(FilterConfig var1) throws ServletException;

    void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;

    void destroy();
}
```


Registration servlet3.0出现，用于动态向servletcontext注册servlet，filter实例，
```java
public interface Registration {
    String getName();

    String getClassName();

    boolean setInitParameter(String var1, String var2);

    String getInitParameter(String var1);

    Set<String> setInitParameters(Map<String, String> var1);

    Map<String, String> getInitParameters();

    public interface Dynamic extends Registration {
        void setAsyncSupported(boolean var1);
    }
}
```