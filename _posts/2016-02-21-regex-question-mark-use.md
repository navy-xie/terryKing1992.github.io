---
layout: post
title: 正则表达式中问号的用法
date: 2016-02-21 10:18:50
tags: [正则表达式]
---

由于最近工作需要，要从网页链接中找到网页中有用的博客内容，大家都知道，基本使用正则表达式来匹配是最简单的一种做法，而一般都是div中有div，怎么才能匹配到那些内容的div而不是一直匹配到最后面的div呢?可能这样表述不是很清楚，下面看一下代码就知道怎么回事了

2、具体描述

	<div id="article_details" class="details">  
	    <div id="article_content" class="article_content">  
	        错误：1488 SQLSTATE: HY000 (ER_REORG_PARTITION_NOT_EXIST) 消息：重组的分区数超过了已有的分区数。  
	    </div>  
	</div>  

怎么才能匹配到中间的div配对标签呢?经查询资料发现，在java中如果使用来匹配的话，有一种贪婪匹配原则，就是如果你使用这个正则表达式来匹配的话
.那么肯定会匹配到最后面那个才会结束，代码如下

	public static void main(String[] args) {  
	    // TODO Auto-generated method stub  
	    // 创建包  
	    String str = "<div id=\"article_details\" class=\"details\"><div id=\"article_content\" class=\"article_content\">错误：1488 SQLSTATE: HY000 (ER_REORG_PARTITION_NOT_EXIST) 消息：重组的分区数超过了已有的分区数。</div></div>";  
	    String regex = "<div id=\"article_content\" class=\"article_content\">.*</div>";  
	    Pattern pattern = Pattern.compile(regex);  
	    Matcher mch = pattern.matcher(str);  
	    while (mch.find()) {  
	        System.out.println(mch.group());  
	    }  
	}

运行结果可以看到

从上图可以看到匹配到了最后面一个这种就是贪婪匹配，而如果想要表达式不匹配最后面的那个div，那么？就派上用场了。
将上面代码中的正则表达式变为

	String regex = "<div id=\"article_content\" class=\"article_content\">(.*?)</div>";  

然后再运行可以看到，通过非贪婪表达式已经完美解决了我所遇到的问题.

通常的非贪婪表达式有以下几种格式：

	*? 重复任意次，但尽可能少重复
	+? 重复1次或更多次，但尽可能少重复
	?? 重复0次或1次，但尽可能少重复
	{n,m}? 重复n到m次，但尽可能少重复
	{n,}? 重复n次以上，但尽可能少重复

以后在项目中应该还会用到，在这里先记录一下
