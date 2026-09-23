# qwen-image-21-prompting

[English](README.md) | [繁體中文](README.zh-Hant.md) | [简体中文](README.zh-Hans.md)

替 **Qwen-Image 2.1**（文生图与图片编辑）写提示词的 skill，照官方 rewriter 规格走，
并且把中文文本渲染这件事处理好。

这是**提示词层**。它产出一段提示词与一个建议画幅，你可以贴进云端平台、送进 API、
或喂给本地后端——skill 不在乎是哪一种，也刻意不谈模型档、权重、量化与运行参数。

## 安装

把整个文件夹放进 skills 目录：

```
~/.claude/skills/qwen-image-21/
```

## 里面有什么

```
SKILL.md                          入口——路由，以及唯一值得重复的那条规则
references/
  official_rewriter_t2i.txt         官方 rewriter 系统提示词（文生图），原文未改
  official_rewriter_edit.txt        官方 rewriter 系统提示词（图片编辑），原文未改
  prompt-writing.md                 两份规格的精要，加上实测出来的写法规则
  text-generation.md                画面文本，尤其是中文
  community-findings.md             宣称的能力实测——包含【不成立】的那些
  delivery-and-verification.md      送出前的接线核对、拿回图之后的验收、A/B 纪律
```

**参考文档只留英文，这是刻意的。** 它们原本有三种语言，而同一条结论一旦被修正，
三份副本就会开始漂移——而且是无声的，因为没有人会去 diff 一份翻译。
改成翻译 README：README 是人读一次的东西，参考文档是 agent 每次都要读的东西。

## 浓缩版

**用一个问题挑规格——有没有输入图？**
没有输入图 → 走文生图 rewriter；有输入图 → 走编辑 rewriter。
把对应的文件整份读完，照顺序运行：那份规格是一串有序步骤，**后面的步骤不会回头修改前面的**。

**画面里要有文本时，只做两件事。** 在提示词里用「占画面的比例」讲出最小字级，
并且告诉调用者：文本若长坏了就说一声，我换一个词重生。
固定 seed ＋ 换掉坏掉的那个词，是唯一稳定划算的文本技巧。

**没有任何自动化的东西能验中文本形。** LLM、OCR、VLM 都用上下文消解歧义——
而那正是「错字刚好符合预期词义」得以藏住的同一个机制。
文本非对不可的时候，要嘛让人去看，要嘛用真字体合成上去。

## 实测结论都有日期

`community-findings.md` 记的是量了什么、什么时候量的、怎么量的——
包含那些被宣称、但实际上站不住的能力（360° 全景接缝合不起来；
透明背景对一般对象干净，对文本会劣化）。
在你自己的组态上重验过再依赖它，**尤其是否定结论**——后续版本很可能已经修好了。

## 提示词不是全部的工作

`delivery-and-verification.md` 讲提示词前后那两段：
模型有某个能力，不代表你正在调用的那个端点把它接出来了
（实例：同一台机器，文生图实际输出 1152×2048，它自己的编辑路径却无声忽略了送进去的尺寸）；
以及图回来之后要在**未经修饰的原图**上验收，不是看预览。
**已经准备好的请求，不是已经生成的图片。**

## 出处

`references/official_rewriter_*.txt` 是 Qwen-Image 2.1 官方 rewriter 系统提示词，原文收录未修改。
其余都是自己动手测出来的笔记，另有一份清楚标注出处的第三方 benchmark 摘要在 `community-findings.md`。

`delivery-and-verification.md` 借鉴了 Craft Skills 的
[`qwen-image-gen`](https://github.com/ZSeven-W/craft-skills/tree/main/skills/qwen-image-gen)
在交付面的整理框架（MIT，© Fini.Yang）。里面的实测数据是我们自己的。
