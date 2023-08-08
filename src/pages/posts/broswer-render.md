---
layout: '../../layouts/MarkdownPost.astro'
title: 'How browser renders a page'
pubDate: 2023-08-06
description: 'Introduction to how browser works'
author: 'AyaSono'
cover:
  url: 'https://img1.baidu.com/it/u=1251894689,705030244&fm=253&fmt=auto&app=138&f=JPEG?w=750&h=360'
  square: 'https://img1.baidu.com/it/u=1251894689,705030244&fm=253&fmt=auto&app=138&f=JPEG?w=750&h=360'
  alt: 'cover'
tags: ["handle write", "JavaScript", "broswer"]
theme: 'light'
featured: false
---
> Chinese follows English

## How does the browser render a page?

When the browser's web thread receives an HTML document, it generates a render task and passes it to the message queue of the render master thread.

Under the event loop mechanism, the rendering master thread takes out the rendering task from the message queue and starts the rendering process.

-------

The entire rendering process is divided into several stages, namely: HTML parsing, style calculation, layout, layering, drawing, chunking, rasterisation, painting, and so on.

Each stage has clear inputs and outputs, and the output of the previous stage becomes the input of the next stage.

In this way, the entire rendering process forms a well-organised production pipeline.

-------

The first step in rendering is **parsing HTML**.

During the parsing process, CSS is parsed, and JS is executed; in order to improve parsing efficiency, the browser will start a pre-parsing thread before parsing, and it will download the external CSS files and external JS files in the HTML first.

If the main thread parses to the `link` position, and the external CSS file has not yet been downloaded and parsed, the main thread will not wait and will continue to parse the subsequent HTML, because the downloading and parsing of the CSS is done in the pre-parsing thread. This is because the downloading and parsing of the CSS is done in the pre-parsing thread. This is the fundamental reason why CSS does not block HTML parsing.

If the main thread reaches the `script` position, it stops parsing HTML and waits for the JS file to be downloaded and the global code to be parsed before continuing to parse the HTML. this is because the execution of the JS code may modify the current DOM tree, so the generation of the DOM tree has to be paused. This is the fundamental reason why JS blocks HTML parsing.

After the first step, you will get the DOM tree and CSSOM tree, and the browser's default style, internal style, external style, and in-line style will be included in the CSSOM tree.

-------

The next step in rendering is **style computation**.

The main thread traverses the resulting DOM tree and computes the final style for each node in the tree in turn, called Computed Style.

During this process, many predefined values become absolute values, such as `red` becomes `rgb(255,0,0)`, and relative units become absolute units, such as `em` becomes `px`.

After this step, you will get a DOM tree with styles.

--------

The next step is **layout**, after the layout is completed, you will get a layout tree.

The layout stage iterates through each node of the DOM tree in turn, calculating geometric information for each node. For example, the width and height of the node, and its position relative to the containing block.

Most of the time, the DOM tree and the layout tree are not one-to-one.

For example, `display:none` nodes do not have geometric information, so they will not be generated into the layout tree; another example is the use of pseudo-element selectors, although these pseudo-element nodes do not exist in the DOM tree, they have geometric information, so they will be generated into the layout tree. There are also anonymous rowboxes, anonymous blockboxes, and so on, which can cause the DOM tree and the layout tree to not correspond to each other.

-----------

The next step is **layering**

The main thread will use a complex set of strategies to layer in the entire layout tree.

The benefit of layering is that in the future, when a layer is changed, only that layer will be processed subsequently, thus improving efficiency.

Styles such as scrollbars, stacking contexts, transforms, and opacity all affect the layering results to a greater or lesser extent, and can also be used to affect the layering results to a greater extent through the `will-change` attribute.

---------

Then the next step is **drawing**

The main thread generates a separate set of drawing instructions for each layer, which are used to describe how the content of that layer should be drawn.

------

Once the drawing is complete, the main thread submits the drawing information for each layer to the compositing thread, which will do the rest of the work.

The compositing thread first chunks each layer, dividing it into more small regions.

It will take multiple threads from the thread pool to do the chunking.

----

Once the chunking is complete, it enters the **Rasterisation** phase.

The compositing threads will give the block information to the GPU process to complete the rasterisation at a very high speed.

The GPU process will open multiple threads to complete the rasterisation and prioritise the blocks close to the viewport area.

The result of rasterisation is a block of bitmap

---------

The last stage is the **drawing**.

The compositing thread gets the bitmaps for each layer and each block and generates a "quad" of information.

The quad identifies where on the screen each bitmap should be drawn, and takes into account deformations such as rotation and scaling.

Transformations occur in the compositing thread, not in the main rendering thread, which is essentially why `transform` is so efficient.

The compositing thread then submits the quad to the GPU process, which generates a system call that is submitted to the GPU hardware for the final screen image.

## What is reflow?

The essence of reflow is to recalculate the layout tree.

Layout is triggered when the layout tree needs to be recalculated after an operation that affects the layout tree.

In order to avoid multiple consecutive operations that cause the layout tree to be recalculated, the browser will merge these operations and recalculate the layout tree once all the JS code has been completed. Therefore, the reflow caused by changing attributes is done asynchronously.

Also because of this, when JS fetches layout properties, it may not get the latest layout information.

The browser weighs the options and decides to get an immediate reflow of the property.

## What is repaint?

The essence of repaint is that the drawing instructions are recalculated based on the layering information.

When a visible style is changed, it needs to be recalculated, which triggers a repaint.

Since an element's layout information is also part of the visible style, reflow will always cause repaint.

## Why is transform more efficient?

Because transform doesn't affect either the layout or the draw instructions, it only affects the last "draw" stage of the rendering process.

Since the draw stage is in the compositing thread, changes to transform have little effect on the main rendering thread. Conversely, no matter how busy the main rendering thread is, it won't affect the transform.

## 浏览器是如何渲染页面的？

当浏览器的网络线程收到 HTML 文档后，会产生一个渲染任务，并将其传递给渲染主线程的消息队列。

在事件循环机制的作用下，渲染主线程取出消息队列中的渲染任务，开启渲染流程。

-------

整个渲染流程分为多个阶段，分别是： HTML 解析、样式计算、布局、分层、绘制、分块、光栅化、画

每个阶段都有明确的输入输出，上一个阶段的输出会成为下一个阶段的输入。

这样，整个渲染流程就形成了一套组织严密的生产流水线。

-------

渲染的第一步是**解析 HTML**。

解析过程中遇到 CSS 解析 CSS，遇到 JS 执行 JS。为了提高解析效率，浏览器在开始解析前，会启动一个预解析的线程，率先下载 HTML 中的外部 CSS 文件和 外部的 JS 文件。

如果主线程解析到`link`位置，此时外部的 CSS 文件还没有下载解析好，主线程不会等待，继续解析后续的 HTML。这是因为下载和解析 CSS 的工作是在预解析线程中进行的。这就是 CSS 不会阻塞 HTML 解析的根本原因。

如果主线程解析到`script`位置，会停止解析 HTML，转而等待 JS 文件下载好，并将全局代码解析执行完成后，才能继续解析 HTML。这是因为 JS 代码的执行过程可能会修改当前的 DOM 树，所以 DOM 树的生成必须暂停。这就是 JS 会阻塞 HTML 解析的根本原因。

第一步完成后，会得到 DOM 树和 CSSOM 树，浏览器的默认样式、内部样式、外部样式、行内样式均会包含在 CSSOM 树中。

-------

渲染的下一步是**样式计算**。

主线程会遍历得到的 DOM 树，依次为树中的每个节点计算出它最终的样式，称之为 Computed Style。

在这一过程中，很多预设值会变成绝对值，比如`red`会变成`rgb(255,0,0)`；相对单位会变成绝对单位，比如`em`会变成`px`

这一步完成后，会得到一棵带有样式的 DOM 树。

--------

接下来是**布局**，布局完成后会得到布局树。

布局阶段会依次遍历 DOM 树的每一个节点，计算每个节点的几何信息。例如节点的宽高、相对包含块的位置。

大部分时候，DOM 树和布局树并非一一对应。

比如`display:none`的节点没有几何信息，因此不会生成到布局树；又比如使用了伪元素选择器，虽然 DOM 树中不存在这些伪元素节点，但它们拥有几何信息，所以会生成到布局树中。还有匿名行盒、匿名块盒等等都会导致 DOM 树和布局树无法一一对应。

-----------

下一步是**分层**

主线程会使用一套复杂的策略对整个布局树中进行分层。

分层的好处在于，将来某一个层改变后，仅会对该层进行后续处理，从而提升效率。

滚动条、堆叠上下文、transform、opacity 等样式都会或多或少的影响分层结果，也可以通过`will-change`属性更大程度的影响分层结果。

---------

再下一步是**绘制**

主线程会为每个层单独产生绘制指令集，用于描述这一层的内容该如何画出来。

------

完成绘制后，主线程将每个图层的绘制信息提交给合成线程，剩余工作将由合成线程完成。

合成线程首先对每个图层进行分块，将其划分为更多的小区域。

它会从线程池中拿取多个线程来完成分块工作。

----

分块完成后，进入**光栅化**阶段。

合成线程会将块信息交给 GPU 进程，以极高的速度完成光栅化。

GPU 进程会开启多个线程来完成光栅化，并且优先处理靠近视口区域的块。

光栅化的结果，就是一块一块的位图

---------

最后一个阶段就是**画**了

合成线程拿到每个层、每个块的位图后，生成一个个「指引（quad）」信息。

指引会标识出每个位图应该画到屏幕的哪个位置，以及会考虑到旋转、缩放等变形。

变形发生在合成线程，与渲染主线程无关，这就是`transform`效率高的本质原因。

合成线程会把 quad 提交给 GPU 进程，由 GPU 进程产生系统调用，提交给 GPU 硬件，完成最终的屏幕成像。

## 什么是 reflow？

reflow 的本质就是重新计算 layout 树。

当进行了会影响布局树的操作后，需要重新计算布局树，会引发 layout。

为了避免连续的多次操作导致布局树反复计算，浏览器会合并这些操作，当 JS 代码全部完成后再进行统一计算。所以，改动属性造成的 reflow 是异步完成的。

也同样因为如此，当 JS 获取布局属性时，就可能造成无法获取到最新的布局信息。

浏览器在反复权衡下，最终决定获取属性立即 reflow。

## 什么是 repaint？

repaint 的本质就是重新根据分层信息计算了绘制指令。

当改动了可见样式后，就需要重新计算，会引发 repaint。

由于元素的布局信息也属于可见样式，所以 reflow 一定会引起 repaint。

## 为什么 transform 的效率高？

因为 transform 既不会影响布局也不会影响绘制指令，它影响的只是渲染流程的最后一个「draw」阶段

由于 draw 阶段在合成线程中，所以 transform 的变化几乎不会影响渲染主线程。反之，渲染主线程无论如何忙碌，也不会影响 transform 的变化。
