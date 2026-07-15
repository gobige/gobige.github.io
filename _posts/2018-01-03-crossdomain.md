---
layout: post
title: '开发中的跨域问题'
subtitle: ''
date: 2018-01-03
categories: 其他
author: yates
cover: 'http://cctv.com'
tags: 
---

### 前言
现实工作开发中经常会有跨域的情况，因为公司会有很多项目，也会有很多子域名，各个项目或者网站之间需要相互调用对方的资源，避免不了跨域请求，那么跨域时怎么产生的，又应该怎么解决这类问题呢

首先，先来了解几个概念

### 同源
所谓同源是指，域名，协议，端口相同。当页面在执行一个脚本时会检查访问的资源是否同源，如果非同源，那么在请求数据时，浏览器会在控制台中报一个异常，提示拒绝访问。

### 同源策略
由 Netscape 公司提出的一个著名的安全策略，所有支持 JavaScript 的浏览器都会使用这个策略
- DOM同源策略：禁止对不同源页面DOM进行操作。这里主要场景是iframe跨域的情况，不同域名的iframe是限制互相访问的。
- XmlHttpRequest同源策略：禁止使用XHR对象向不同源的服务器地址发起HTTP请求。

### 跨域
跨域问题是由浏览器的同源策略造成的，当从一个域名去请求另外一个域名的资源，浏览器不能执行其他域名网站的脚本，被拒绝访问。跨域的严格一点来说就是只要协议，域名，端口有任何一个的不同，就被当作是跨域。

### 跨域解决方案

**跨域时，请求其实已经发出去了，服务器也接收并正常响应了，只是当数据传回浏览器时，被浏览器给拦截并报错了。**

**CORS（跨域资源共享） 最标准的做法**
浏览器说“除非后端（B 小区）的响应头里，加上这张“同意书”（Header）。
```java
# 允许来自 http://localhost:8080 的网页请求我（也可以写 * 代表允许所有人，但不安全）
Access-Control-Allow-Origin: http://localhost:8080
# 允许他们使用 GET, POST, OPTIONS 等方法
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
# 允许他们携带 Cookie
Access-Control-Allow-Credentials: true
``` 

```java
Spring 框架，只需在 Controller 类或方法上加一个注解即可：
@RestController
@RequestMapping("/api")
@CrossOrigin(origins = "http://localhost:8080") // ⭐ 允许该源跨域访问
public class UserController {
    @GetMapping("/info")
    public User getInfo() {
        return new User("Mr.Chen");
    }
}
``` 

**nginx代理跨域**
- 前端地址：http://my-app.com
- 真正后端：http://actual-api.com:9000
- 我们用 Nginx 统一代理到 http://my-app.com 下：

、、、java
server {
    listen 80;
    server_name my-app.com;

    # 1. 分发前端静态资源
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    # 2. 💡 拦截所有带 /api/ 的请求，悄悄转发给后端，浏览器毫不知情！
    location /api/ {
        proxy_pass http://actual-api.com:9000/; # 后端真实地址
        proxy_set_header Host $host;
    }
}
、、、
 

### 未完待续....
