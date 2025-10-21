title: "Thread by @AlchainHust2025-10-21T14:09:35+08:00"
source: "https://x.com/AlchainHust/status/1980492106801262955"
author:
  - "[[@AlchainHust]]"
published: 2025-10-21
created: 2025-10-21
description:
tags:
  - "clippings"
tags:
  - "readings"
---
**AI进化论-花生** @AlchainHust 2025-10-20

Andrej Karpathy提出了一个很激进的想法：所有LLM的输入都应该是图像，包括纯文本。  
  
什么意思？  
  
传统的大语言模型：文本 → tokenizer → LLM → 输出  
  
Andrej的vision：文本 → 渲染成图片 → LLM → 输出  
  
即使你要输入的就是纯文本，也先把它渲染成图片，再喂给模型。  
  
为什么这么做？  
  
他给了4个理由：  
  
1\. 信息压缩更高效  
  
这正是DeepSeek-OCR证明的。一页文档，传统方式可能需要2000个text tokens，用vision tokens只要64个。压缩率30倍。  
  
文本tokens很浪费，图像tokens更密集。  
  
2\. 更通用  
  
Text tokens只能表达文字。但现实世界的信息不只是文字：

\- 粗体、斜体

\- 彩色文字

\- 表格、图表

\- 任意图像  
  
全部渲染成图像输入，模型天然就能处理这些。  
  
3\. 可以用双向注意力  
  
这是技术细节。传统的text-to-text是自回归的（从左到右）。图像输入可以用双向注意力，看到全局信息，更强大。  
  
4\. 删除tokenizer（重点！）  
  
Andrej很讨厌tokenizer。  
Andrej 很讨厌 tokenizer。  
  
他的吐槽：

\- Tokenizer是一个丑陋的、独立的、非端到端的阶段

\- 它继承了Unicode、字节编码的所有历史包袱

\- 有安全风险（如continuation bytes攻击）

\- 两个看起来一样的字符，在tokenizer眼里可能完全不同

\- 😊这个emoji在tokenizer里只是一个奇怪的token，不是一张真正的笑脸图片  
  
他希望tokenizer消失。  
  
他的vision是什么  
  
\- 输入：全部是图像（即使原本是文本）

\- 输出：还是文本（因为输出像素不现实）  
  
OCR只是vision→text任务之一。很多text→text任务都可以变成vision→text。  
  
我的理解  
  
Andrej这个观点很激进，但确实有道理。  
  
从信息论角度，图像确实比文本更高效。DeepSeek-OCR证明了这一点：64个vision tokens就能表达2000个文本tokens的信息。  
  
从通用性角度，图像输入天然支持各种格式（粗体、颜色、图表），不需要tokenizer这个中间层。  
  
但问题是：  
  
1\. 计算成本：处理vision tokens比text tokens贵。虽然token数量少了，但每个vision token的计算量更大。

2\. 训练数据：现有的大部分训练数据都是纯文本。要全部渲染成图像，成本很高。

3\. 输出问题：他也承认，输出像素不现实。所以只能是图像输入→文本输出的混合模式。  
  
但长远看，这个方向可能是对的。  
  
特别是考虑到：

\- 人类的输入本来就是多模态的（文字、图片、视频）

\- Tokenizer确实有很多问题（安全、Unicode、历史包袱）

\- 未来的AI应该能直接理解像素，而不是把一切都变成token  
  
DeepSeek-OCR可能只是开始。它证明了"上下文光学压缩"是可行的。  
  
Andrej看到的是更远的未来：一个没有tokenizer的世界，所有输入都是图像，所有输出都是文本。  
  
这会不会成为现实？不知道。  
  
但至少，这个方向值得探索。

> 2025-10-20
> 
> I quite like the new DeepSeek-OCR paper. It's a good OCR model (maybe a bit worse than dots), and yes data collection etc., but anyway it doesn't matter.
> 
> The more interesting part for me (esp as a computer vision at heart who is temporarily masquerading as a natural language x.com/vllm\_project/s…  
> 我相当喜欢这篇新的 DeepSeek-OCR 论文。它是一个不错的 OCR 模型（可能比点阵略差），是的，数据收集等，但无论如何，这并不重要。
> 
> 对我来说更有趣的部分（尤其是作为一个内心深处是计算机视觉的人，现在暂时伪装成自然语言 x.com/vllm\_project/s…