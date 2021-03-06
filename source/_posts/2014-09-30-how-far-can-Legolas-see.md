---
layout: post
title: Legolas 究竟能看多远？
date: 2014-09-30 20:54:40
categories: 怪哉
tags: [指环王,眼睛,艾里斑]
comments: true
---
指环王的设定中，精灵族被描述为英俊潇洒、武功高强、视力敏锐的类人类（human-like）形象。别的不说，但就视力敏锐这一项，就让人望不可及。在护戒小分队中，精灵王子 Legolas 扮演侦察员的角色，经常留意远处突然出现的危险。“the Riders of Rohan”一章中，Legolas 在 5 leagues（约24km。 1 leaue等于3mile,1 mile 约1.6km）开外就发现了 Rohan 的骑兵队伍，不仅看清他们的发色、装备、人数，还从中认出了对方的指挥官。多么了不起的眼力！ Legolas 真能看那么远么？
<!--more-->

这里就要引入一个概念：艾里斑（Airy disk）。学术来讲，就是光束通过一个圆形孔径的镜头所能得到的最好光点的描述。因为光作为一种电磁波，在通过狭缝时候会发生衍射，这样会得到一个明亮的中心与外部明暗交替的圆环。这个中心就被称为艾里斑（airy disk），它与周边的圆环一道被叫做艾里图（airy pattern）。他们都得名于George Biddell Airy，因为他对这一现象做了理论解释。

艾里斑可以看作是对原目标点的模糊，因为光线的衍射会丧失一部分细节。如果有两个点距离很近，他们通过透镜后的艾里斑的边缘圆环重合的时候，就无法区分这两个点。这样，我们就会说看不清目标了。那么人眼睛视力的极限是多少呢？

如上，艾里斑的大小确定了透镜的分辨率，当然数值越小的艾里斑代表着更好的分辨率。它由这个公式确定

    sin(θ) ≈ 1.22×（λ/d）
λ代表波长，d代表孔径直径。在θ极小的时候，sin(θ)∽θ，所以上式也可简化为
    
    θ ≈ 1.22×（λ/d）
由上可知，透镜分辨率由波长与孔径大小决定。为了获得更好（小）的分辨率，可以通过减少波长（换光）与增大孔径（大镜头）来达成。在θ小的情况下，还有一个更简易的公式
    
    θ ≈ x/f = h/D = 1.22×（λ/d）
x代表物体在视网膜（幕布）的投影大小，f为眼睛焦距,h为物体高度，D为物距。对于人眼睛来讲，瞳孔大小5毫米，光线波长500纳米，很容易算出角度为

    θ ≈ 1.22×（λ/d）× 180/π ≈ 0.007°
    h = θ×D = 1.22×（λ/d）× 100 = 1.22×10^-2(m) //设距离100米
这个是人眼睛分辨的极限了，相当于在100m处看一1厘米的物体，要知道到这个程度人眼睛看到的物体在视网膜上的投影就会被模糊掉了。

当 Legolas 在24千米外观察骑兵时候，其能分辨的极限可从上面公式得处为2.96米，也即3米高的物体在Legolas看来就已经开始模糊不清了，或许他能从模糊中数出骑兵数目，可要看清每人装束就不可能了。除非————

## what if
上面的公式告诉我们，要想让θ小。可以通过减小λ,增大d,增大f来实现。

如果Legolas能够看到波长更短的光线，比如紫外线。如果他能看到波长200纳米的紫外线，那么在24千米外他能够刚刚分清

    h = 1.22×（200×10^-9/(5×10^-3)）×24000 = 1.2米
这样大小的物体，区分人物已经够了。可是，要知道空气会吸收紫外线，紫外线在大气中并不能有效传播，意味着Legolas很可能生活在一片黑暗当中！

Legolas眼睛构造与人不同，比如说，焦距长，好比一个望远镜。这样对于Legolas简直就是灾难，因为长焦的镜头一般来讲视场小，对于抖动很敏感。玩过望远镜的同学就能感觉到，人裸眼看到的一片星空，通过望远镜只能看到皮毛大的一块，通过望远镜来巡视一片星空是很费时费力的。这对善于狩猎的精灵族人来讲，要有进化出强健的结构来支撑脖子（防抖），同时还能快速扫描目标区域。那么精灵族人的头部可能是这样的：

![Chicken Powered Camera Stabilization](/images/Chicken-Powered-Image-Stabilization.jpg)

或者，Legolas的瞳孔异于常人。比如这样子：

![gollum](/images/gollum.jpg)

当然，还可以靠魔法嘛。
