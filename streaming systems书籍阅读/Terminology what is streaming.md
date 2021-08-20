**Terminology: what is streaming?**

Before going any further, I’d like to get one thing out of the way: what is streaming? The term streaming is used today to mean variety of different things (and for simplicity I’ve been using it somewhat loosely up until now), which can lead to misunderstandings about what streaming really is, or what streaming systems are actually capable of. As a result, I would prefer to define the term somewhat precisely.

```
在进一步讨论之前，我想先弄清楚一件事：什么是streaming？streming这个术语今天被用来表示各种不同的事物（为了简单起见，我一直在使用它直到现在有点松散），这可能会导致对streaming到底是什么，或者streaming systems实际上能够做什么的误解。因此，我更倾向于对这个术语进行精确的定义。
```



The crux of the problem is that many things that ought to be described by what they are (e.g. unbounded data processing, approximate results, etc.), have come to be described colloquially by how they historically have been accomplished (i.e., via streaming execution engines). This lack of precision in terminology clouds what streaming really means, and in some cases burdens streaming systems themselves with the implication that their capabilities are limited to characteristics historically described as “streaming”, such as approximate or speculative results. 

Given that well- designed streaming systems are just as capable (technically more so) of producing correct, consistent, repeatable results as any existing batch engine, I prefer to isolate the term “streaming” to a very specific meaning: 

a type of data processing engine that is designed with infinite datasets in mind. Nothing more. (For completeness, it’s perhaps worth calling out that this definition includes both true streaming as well as micro- batch1implementations)

```
问题的关键在于，许多应该用它们的本来面目来描述的事情（例如，无界数据处理、近似结果等），已经用它们在历史上是如何完成的（即，通过流执行引擎）来口语化地描述了。术语中缺乏精确性模糊了streaming的真正含义，在某些情况下，流媒体系统本身也会受到影响，即它们的功能仅限于历史上描述为“流媒体”的特征，例如近似或推测性结果。
考虑到设计良好的流式处理系统与任何现有的批处理引擎一样能够（技术上更是如此）产生正确、一致、可重复的结果，我更倾向于将术语“流式处理”隔离为一个非常具体的含义：一种在设计时考虑到无限数据集的数据处理引擎。没别的了。（为了完整起见，也许值得一提的是，这个定义既包括真正的流式处理，也包括微批处理1实现）
streaming定义：一种数据处理引擎，其在被设计时考虑无限数据集。
```

**UnboundedData**:Atypeofever-growing,essentiallyinfinitedataset. These are often referred to as “streaming” data or “streams”. However, the terms streaming or batch are problematic when applied to datasets, because as noted above, they imply the use of a certain type of execution engine for processing those datasets. The key distinction between the two types of datasets in question is, in reality, their finiteness, and it’s thus preferable to characterize them by terms that capture this distinction. As such, I will refer to infinite “streaming” datasets as unbounded data, and finite “batch” datasets as bounded data.



**UnboundedorContinuousDataProcessing**:Anongoingmodeof data processing, applied to the aforementioned type of unbounded data. As much as I personally like the use of the term “streaming” to describe this type of data processing, its use in this context again implies the employment of a streaming execution engine, which is at best misleading; repeated runs of batch engines have been used to process unbounded data since batch systems were first conceived (and conversely, well-designed streaming systems are more than capable of handling “batch” workloads over bounded data). As such, for the sake of clarity, I will prefer to simply refer to this as continuous data processing.



Low-Latency,Approximate,and/orSpeculativeResults:Thesetypesof results are most often associated with streaming engines. The fact that batch systems have traditionally not been designed with low-latency or speculative results in mind is a historical artifact, and nothing more. And of course, batch engines are perfectly capable of producing approximate results if instructed to. Thus, as with the terms above, it’s far better describing these results as what they are (low-latency, approximate, and/or speculative) than by how they have historically been manifested (via streaming engines).







