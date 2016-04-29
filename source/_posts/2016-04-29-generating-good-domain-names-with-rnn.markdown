---
layout: post
title: "使用RNN自动生成域名"
date: 2016-04-29 19:31:34 +0800
comments: true
categories: [Machine Learning]
---
如果有很多猴子分别使用打印机瞎打字，什么时候能打出一篇莎士比亚的作品？
好吧，我们不让猴子写巨著，给他一个简单的任务——帮我想出些不错的域名出来。域名已经是个非常饱和的市场了，
四位以下域名已经全部被注册了，七八位又太长，五位六位或许能淘到些宝贝。可是仅仅五位域名的组合个数已经突破了六千万个(36^5)，
从里面挑出一个简直是大海捞针，更有人使用单词、拼音规则已经从里面筛选了一通。那么有没有精准一点的方法筛选呢？
<!--more-->
当然是可以的。借助于RNN，我们可以轻松地自动生成成千上万的候选域名。RNN是一种特殊的神经网络，适合处理具有序列属性的数据。
Andrej Karpathy 在[The Unreasonable Effectiveness of Recurrent Neural Networks](http://karpathy.github.io/2015/05/21/rnn-effectiveness/)一文中
介绍了一种 char-rnn 自动内容生成的方法。他将内容生成当作是序列预测问题，以一段文本作为输入序列，其对应的输出序列为原文本序列的移位序列，
比如 "hello world"设定上下文长度为4,则得到序列为"hell"->"ello", "ello"->"llo "... 从而获得了预测一个字符的能力。

Karpathy 提供了一个基于python的mini实现，不过不是基于lstm，而是普通的rnn。网络结构中每个记忆单元含有两个隐含层，各层之间全连接，而不同的记忆单元共享权值。
对每个输入都进行正向计算与反向传播并更新权重。如下图所示：

{% img /images/char-rnn.png char-rnn %}

由于神经网络要求输入数据一致，所以需要对原数据进行调整，它以类似于词袋模型方法将每个字符向量化，将输入数据对齐。虽然Karpathy的方法基于字符，
不过也可以将其扩充为word-based，方法也十分类似，直接统计文档中的word，将每个word向量化。后面的处理逻辑一致。

本文边基于这个mini实现来进行网络训练。下一步要获取一批高质量域名数据，当然不存在这样的数据可供下载，我们退而求其次，可以收集一些知名公司的域名
作为训练样本。我们选择了一个国内的黄页网站以及alexa的top500站点，从两个站点抓取所有的域名。这些站点的抓取比较简单，根据url的索引规律就可以很容易
遍历所有的条目，使用类似for each ->fetch ->parse方式即可抓取。不过为了尝鲜，选择使用pyspider作为抓取工具。工具的安装配置不再本文的介绍范围，
在管理界面新建一个任务，其配置为:

	class Handler(BaseHandler):
		crawl_config = {
		}

		@every(minutes=24 * 60)
		def on_start(self):
			self.crawl('http://www.qkankan.com/all/index.html', callback=self.index_page)

		@config(age=10 * 24 * 60 * 60)
		def index_page(self, response):
			for each in response.doc('h2 &gt; a').items():
				if each.attr.href.find('qkan') &gt; -1:
					self.crawl(each.attr.href, callback=self.detail_page)
			for each in response.doc('.next a').items():
				self.crawl(each.attr.href, callback=self.index_page)

		@config(priority=2)
		def detail_page(self, response):
			return {
				"name":response.doc('h1').text(),
				"domain":response.doc('#sitelogo a[href^=&quot;http&quot;]').attr.href,
			}
			
同理配置Alexa :

	class Handler(BaseHandler):
		crawl_config = {
		}

		@every(minutes=24 * 60)
		def on_start(self):
			self.crawl('http://www.alexa.com/topsites', callback=self.index_page)

		@config(age=10 * 24 * 60 * 60)
		def index_page(self, response):
			for each in response.doc('.desc-paragraph &gt; a').items():
				self.crawl(each.attr.href, callback=self.detail_page)
			self.crawl(response.doc('.next').attr.href, callback=self.index_page)

		@config(priority=2)
		def detail_page(self, response):
			return {
				"name":response.doc('.compare-site &gt; a').attr.href,
				"domain":response.doc('.compare-site &gt; a').text(),
			}			
			
在pyspider配置界面导出json格式的结果，使用如下脚本解析出域名信息（去除www,com,org等前后缀）并输出到文件中.
使用了tldextract的包来解析域名，结果导出到names.txt，每行记录一个域名。

{% blockquote %}
strToDomain('domain.json','names.txt')	
{% endblockquote %}

	import json
	import tldextract
	def strToDomain(fileA,fileB):
		with open(fileA,'r') as fr,open(fileB, 'w') as fw:
			for line in fr.readlines():
				record = json.loads(line)
				domainStr = None
				try:
					domainStr = record['result']['domain']
				except:
					continue
				if domainStr != None:
					domainResult = tldextract.extract(domainStr)
					fw.write(domainResult.domain + '\n')	

最终一共获取了四千左右域名,共43K大小。虽然数据量不够大，姑且一试。

下文是一些自动生成的域名信息（剔除了四位以下八位以上），看起来似乎可以当上总经理出任CEO迎娶白富美走向人生巅峰呢。。。

{% blockquote %}
molal maisal liansu mngath telmjd gopzort inquar huathe shmbora kinlink dalma konten careb entonga fatilow ibacks steque borgai hfesyen sarice indoy sicin akith rebat jobeglm newingg
{% endblockquote %}