---
layout: post
title: "c++ template机制浅析"
description: ""
category: 语言
tags: []
---
{% include JB/setup %}


最近写了一点点东西用到了模板，发现了模板的几个不易发觉的问题，涉及到C++模板的实现机制和编译链接的问题，下面简单分析下。

首先，模板并非一种具体的实现，而是产生具体实现的方式。这虽然是一种对语言表现能力的扩展，但同时也意味着对原有语言结构和编译链接模型的冲击。

对C语言而言，一个项目的结构可能是这样的：每个模块的头文件描述这个模块的接口，具体实现放在这个模块的.c文件中，每个模块分别编译，生成主程序时再进行链接即可。这里插一句，c语言的编译中，允许程序使用一个未“声明”的函数，而在c++的编译中，使用的函数必须先“声明”，但可以不“定义”，以上都是对一个编译单元而言的，这也算是c++对c的一种改良吧。所以不管是c还是c++，在每个编译单元里，我们不必知道每个函数的实现，只要在链接时保证能够找到这个函数的实现地址即可。这个过程对于大部分人而言，只要看过相关的背景知识，理解起来不难，而且函数的定义与使用都是程序员显式的写出来的，出现的错误也更“显然”一点。

但是加入模板之后问题不同了，我们某种程度上既难以保证模板能提供给我们一种需要的实现，而且也难以保证提供给我们的实现就是对的，至少它使以上两个问题复杂化了。

一个具体的函数实现可能是由编译器产生的，这本身带来了很大的复杂性。因为编译器必须能够看到模板的全部内容，而非只有接口，才能够知道该生成哪些实例化的代码。所以模板的全部内容对每个使用该模板的编译单元而言都应当是可见的，才能够保证所有该生成的代码都已生成，这使得模板的内容都必须放在头文件里，不能实现接口和定义的分离，这也是由模板的性质决定的。当然，解决这一问题也有很多其他的办法，如使用模板的显式实例化等等，但可能会带来更多的问题。总之，在链接的时候，“保证所有需要的函数实现都存在”这一问题变得更复杂了，一个常用的解决办法是把模板的全部内容写在一起，但这显然不是一个完美的方案。

另一方面，模板对函数实现的“唯一性”也有冲击。在c/c++中，链接时找到两个相同接口的函数会报“重复定义”的，这种唯一性在很多层面上都是必要的。但对于模板而言，每个模块可能都引用了一个模板的头文件，而如上所述，头文件里常常包含模板的全部内容，所以不同模块很可能生成了同一模板的同种实现，大部分情况下这些代码都是重复的，对于这种情况，c++编译器的选择不是报“重复定义”错，而是选择使用其中的一种，而将其他的丢弃。这有一定的道理，但带来了另一个问题，假如这几个接口相同的函数压根不是同样的函数，只不过接口相同罢了，而编译器会选择其中的一个来代替所有这些函数，显然会出错(可能会使一个程序员的低级错误到运行期才被发现)。另一方面，即使是引用的同一头文件，在不同的对齐方式下，或者在不同的编译参数下，可能也会生成不同的几种实现，即源代码层面相同，二进制层面却不同，按其中一份来链接显然也会出问题(这种错误就诡异的多了)。总之，“保证每个函数调用都链接到正确的实现上“这一问题变得更复杂了，需要程序员自己去解决，而这种错误一般到运行期才能被发现。

总体来说，c++引入模板显然还是有极大的好处的(stl多么好用)，带来的一些问题也都是考虑各种因素之后的一种折中。模板也是一种静态机制，靠编译器来做文章（需要动态特性的时候还要依赖虚函数和类继承等，那会更恐怖吧）。模板机制对原来的编译链接模型有了很大的冲击，使得一个c++程序不仅仅是一集相互无关的编译单位了，即各个编译单位在编译阶段就出现了耦合，一个程序更加强调整体性和中心化了。Bjarne提出了一个陈列台(repository)的概念，用于在编译时维护整个程序的信息。这种观念可看作是认清上述问题之后形成的一个c++程序的构建原则。

以上是我对模板机制的一个浅显的认识，更多相关讨论我建议看《c++的设计与演化》的“模板”一章，书里讲了模板设计时遇到的种种问题和解决方案。我觉得这书实在是了解c++设计的最好材料，值得反复的看。(每看一段都会感慨c++是如何的复杂。。。生命有限，且行且珍惜。。。)

