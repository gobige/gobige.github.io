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
servlet作为服务器和客户端之间交互，响应请求，基于组件，独立于平台的服务端常用的程序，大多数servlet只用来扩张http协议的web服务器

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

针对HttpSession提供 HttpSessionActivationListener，HttpSessionAttributeListener，HttpSessionBindingEvent，HttpSessionBindingListener，HttpSessionIdListener，HttpSessionListener等**事件监听器**
 