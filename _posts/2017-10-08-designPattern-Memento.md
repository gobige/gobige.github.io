---
layout: post
title: '设计模式之备忘录模式'
subtitle: '备忘录模式'
date: 2017-10-08
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 备忘录模式
---

### 行为型模式
#### 备忘录模式
忘录模式（Memento Pattern）保存一个对象的某个状态，以便在适当的时候恢复对象

**优点** 

- 给用户提供了一种可以恢复状态的机制，可以使用户能够比较方便地回到某个历史的状态
- 实现了信息的封装，使得用户不需要关心状态的保存细节

**缺点**

- 消耗资源。如果类的成员变量过多，势必会占用比较大的资源，而且每一次保存都会消耗一定的内存

**使用场景**：  

- 需要保存/恢复数据的相关状态场景
- 提供一个可回滚的操作

示例
```java
public class MementoPattern {
    public static void main(String[] args) {
        ArchiveTool archiveTool = new ArchiveTool();
        ArchiveArea archiveArea = new ArchiveArea();

        archiveTool.setState("三打白骨精");
        archiveTool.setState("大战红孩儿");
        archiveArea.add(archiveTool.saveArchiveFile());
        archiveTool.setState("真假美猴王");
        archiveArea.add(archiveTool.saveArchiveFile());
        archiveTool.setState("三借芭蕉扇");
        archiveArea.add(archiveTool.saveArchiveFile());

        int i = 0;
        ArchiveFile file;
        while (true) {
            file = archiveArea.get(i);
            if (file == null) {
                return;
            }
            System.out.println("存档记录--" + file.getState());
            i++;
        }
    }
}

/**
 * 存档区
 */
class ArchiveArea {
    private List<ArchiveFile> archiveFiles = new ArrayList<>();

    public void add(ArchiveFile file) {
        archiveFiles.add(file);
    }

    public ArchiveFile get(Integer index) {
        if (index < archiveFiles.size()) {
            return archiveFiles.get(index);
        }

        return null;
    }
}

/**
 * 存档工具
 */
class ArchiveTool {
    private String state;

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }

    public ArchiveFile saveArchiveFile() {
        return new ArchiveFile(state);
    }

    public void getArchiveFileState(ArchiveFile archiveFile) {
        state = archiveFile.getState();
    }
}

/**
 * 存档文件
 */
class ArchiveFile {
    private String state;

    ArchiveFile(String state) {
        this.state = state;
    }

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }
}
```