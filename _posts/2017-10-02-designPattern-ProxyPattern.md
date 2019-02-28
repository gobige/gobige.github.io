---
layout: post
title: '设计模式之代理模式'
subtitle: '代理模式'
date: 2017-10-2
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 设计模式
---

### 结构型模式
#### 代理模式
在代理模式（Proxy Pattern）中，一个类**代表**另一个类的**功能**

**优点** 

- 职责清晰。
- 高扩展性。
- 智能化。

**缺点**

- 由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢。 
- 实现代理模式需要额外的工作，有些代理模式的实现非常复杂。

**使用场景**：  

- 远程代理。- 虚拟代理。
- Copy-on-Write 代理。 
- 保护（Protect or Access）代理。
- Cache代理。
- 防火墙（Firewall）代理。
- 同步化（Synchronization）代理。
- 智能引用（Smart Reference）代理


示例
```java
public class proxyPattern {
    public static void main(String[] args) {
        ProxyVideo video = new ProxyVideo("Rick And Morty");
        video.display();
    }
}

class ProxyVideo implements video {
    private YoutuBe youtuBe;
    private String videoName;

    ProxyVideo(String videoName) {
        this.videoName = videoName;
    }


    public String getVideoName() {
        return videoName;
    }

    public void setVideoName(String videoName) {
        this.videoName = videoName;
    }

    @Override
    public void display() {
        if (youtuBe == null) {
            youtuBe = new YoutuBe(videoName);
        }
        youtuBe.display();
    }
}

class YoutuBe implements video {
    private String videoName;

    YoutuBe(String videoName) {
        this.videoName = videoName;
    }

    @Override
    public void display() {
        System.out.println("youtube playing " + videoName);
    }
}

interface video {
    void display();
}
```