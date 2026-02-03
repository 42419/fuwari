---
title: Markdown 示例
published: 2023-10-01
description: 一个简单的 Markdown 博客文章示例。
tags: [Markdown, 博客, 演示]
category: 示例
draft: true
---

# 一个 h1 标题

段落之间用空行分隔。

第二段。_斜体_，**粗体**，和 `等宽字体`。无序列表看起来像这样：

- 这一项
- 那一项
- 另一项

注意 --- 不考虑星号 --- 实际文本内容从第 4 列开始。

> 块引用是这样写的。
>
> 如果你愿意，它们可以跨越多个段落。

使用 3 个破折号表示 em-dash。使用 2 个破折号表示范围（例如，"一切都在第 12--14 章"）。三个点 ... 将被转换为省略号。
支持 Unicode。☺

## 一个 h2 标题

这里有一个有序列表：

1. 第一项
2. 第二项
3. 第三项

再次注意实际文本从第 4 列开始（距离左侧 4 个字符）。这里有一个代码示例：

    # 让我重申一下 ...
    for i in 1 .. 10 { do-something(i) }

正如你可能猜到的，缩进 4 个空格。顺便说一下，如果你愿意，可以使用分隔块而不是缩进块：

```
define foobar() {
    print "Welcome to flavor country!";
}
```

（这样复制和粘贴更容易）。你也可以选择标记分隔块以供 Pandoc 进行语法高亮：

```python
import time
# 快点，数到十！
for i in range(10):
    # （但不要 *太* 快）
    time.sleep(0.5)
    print i
```

### 一个 h3 标题

现在是一个嵌套列表：

1. 首先，获取这些食材：

    - 胡萝卜
    - 芹菜
    - 扁豆

2. 烧一些水。

3. 把所有东西倒进锅里，并按照以下算法操作：

        找到木勺
        揭开锅盖
        搅拌
        盖上锅盖
        把木勺平衡地放在锅柄上
        等待 10 分钟
        转到第一步（或者完成后关闭炉灶）

    不要碰木勺，否则它会掉下来。

再次注意文本如何始终以 4 空格缩进对齐（包括继续第 3 项的最后一行）。

这里有一个链接到 [网站](http://foo.bar)，到 [本地文档](local-doc.html)，以及到 [当前文档中的章节标题](#an-h2-header)。这里有一个脚注 [^1]。

[^1]: 脚注文本放在这里。

表格可以看起来像这样：

size material color

---

9 leather brown
10 hemp canvas natural
11 glass transparent

Table: Shoes, their sizes, and what they're made of

（上面是表格的标题。）Pandoc 也支持多行表格：

---

keyword text

---

red Sunsets, apples, and
other red or reddish
things.

green Leaves, grass, frogs
and other things it's
not easy being.

---

水平规则如下。

---

这里有一个定义列表：

apples
: 适合做苹果酱。
oranges
: 柑橘！
tomatoes
: tomatoe 里没有 "e"。

同样，文本缩进 4 个空格。（在每个术语/定义对之间放一个空行以使内容更分散。）

这里有一个"行块"：

| Line one
| Line too
| Line tree

图片可以这样指定：

[//]: # (![example image]&#40;./demo-banner.png "An exemplary image"&#41;)

行内数学方程式像这样：$\omega = d\phi / dt$。显示数学应该独占一行，并放在双美元符号中：

$$I = \int \rho R^{2} dV$$

$$
\begin{equation*}
\pi
=3.1415926535
 \;8979323846\;2643383279\;5028841971\;6939937510\;5820974944
 \;5923078164\;0628620899\;8628034825\;3421170679\;\ldots
\end{equation*}
$$

并且注意你可以反斜杠转义任何希望字面显示的标点字符，例如：\`foo\`，\*bar\* 等。