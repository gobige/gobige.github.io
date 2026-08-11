---
layout: post
title: 'AI agent初窥'
subtitle: 'AI agent初窥'
date: 2026-04-01
categories: aiagent
author: yates
cover: 'http://cctv.com'
tags: aiagent
---

## RAG检索


### 为什么向量检索得分是 0.88，但 Re-rank 出来依然是 0？

- 向量检索是 Vector Store在召回数据时自动注入到 Metadata 里的向量余弦相似度。
- Re-rank 阶段根本还没打上分/或把低分覆盖：
DefaultContentAggregator 在拿到这 15 条带 SCORE=0.88 的向量检索结果后，会把它们送进 bge-reranker ONNX 模型。
- 模型对这 15 条文本进行了 Cross-Attention 重新打分。
- 模型算出来的真正的 Re-rank Score 极其低（甚至是负数，比如 -2.1）。
- 低于阈值的数据在 Re-rank 内部被全部删除了，导致最终输出为 0。
    - 在 500 字符下被过滤，300 字符下能留存 1 条，说明 ONNX 模型输出的是 Raw Logit（未经归一化的原始分）。**超长或语义稍有偏差**时，原始 Logit 会直接跌成负数（如 -1.5）。
    
**能不能缩小chunk的大小来解决呢？**

- 缩小 Chunk（比如从500字符缩到200字符）确实有显着好处：
    - 减少无关噪音：Chunk 越小，无关文本越少，语义越精细，向量检索（Embedding）和 Cross-Encoder（Re-ranker）的相关性评分倾向于变高。
    - 避免 Truncation（截断）：避免超长文本超过模型的 512 max_seq_length。

缩小 Chunk 只能“缓解”命中率，但无法解决 Logit 绝对值无界引发的“过滤机制失效”；只有归一化才能彻底修复 Re-ranker 的逻辑机制。

解决负数问题：数据归一化处理

归一前后rank得分
![](https://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/agent-design/WeChat5d814932631bd0bae44922014ba22baa.jpg)

**中文建议采用 200 ~ 400 字符 的 Chunk 尺寸，带 15% ~ 20% 的 Overlap（重叠度）。**


### OpenTelemetry日志上报问题，langfuse无法看到上报日志

在 OpenTelemetry 规范中，如果不调用span.end()，Span就永远处于内存未完成状态，Exporter绝对不会把数据打包发送给后台。这就是为什么控制台能看到日志，但 Langfuse 页面永远没消息。


### RAG检索相似度设置为0.9 基本一模一样的句子都检索不到

1. 切片太大，语义偏差
2. 大量重复语句或高度相似片段
3. 重复切割(本次根源)：如果同一份文档或更新后的版本被重复导入/多次写入到了向量数据库（没有做id幂等去重），就会存在内容完全相同的冗余片段。
4. 知识库源文档本身包含大量的冗余信息
5. 向量数据库未做 MMDP / 多样性打散标准的向量余弦相似度检索只关注 “Query 与 Document 的相似度”，而不关注 “Document 与 Document 之间的独立性”。如果知识库中某个主题集中在某一两段话中，Top-15 检索就会把该主题附近最相似的上下文全部揽入，从而形成“近亲繁殖”。

