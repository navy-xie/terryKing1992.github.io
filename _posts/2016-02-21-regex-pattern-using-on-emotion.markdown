---
layout: post
title: 如何用正则表达式 解析 文本中的 表情标签并且渲染到界面上
date: 2016-02-21 15:10:42
tags: [NSRegularExpression, 正则表达式, 表情]
---

最近在使用JSQMessage定制聊天界面的的时候发现，JSQMessage目前不支持表情的显示，所以要自己把表情的文本解析出来，并且显示在TextView上面。思路主要是 使用正则表达式解析，然后通过NSAttributeString进行重新拼装。

表情文本大概长这个样子，让我们有个直观的认识

	HAHFHDJKSAHFDA[大哭]JFDSAJFKDSA[大笑]DJFLAJLFD[大哭]HAHFDKJASFHDASHFDJASHFKDAHSJFDASHFJD[大笑][大笑][大笑]HFDHAJKFHDJSAHFDSJKA

其实仔细想想 并不是很复杂。以下为思路部分：

	1、将字符串转换成可变字符串，然后逐个找出其中的表情文本
	2、将找出来的表情符号，跟表情前面的非表情符号 放入到数组中去。
	3、等待解析结束之后，通过NSMutableAttributeString将字符串拼接，如果是表情，则使用NSTextAttachment来组装NSAttributeString
	4、拼接完成之后，创建一个不可滚动 不可交互的UITextView，使用textStorage进行拼接
	5、使用sizeThatSize来适配 自己定义的矩形范围
	6、添加到View上即可显示

以下是代码部分：

	UITextView *textView = [[UITextView alloc] init];
	textView.scrollEnabled = NO;
	textView.selectable = NO;
	textView.userInteractionEnabled = NO;
	textView.backgroundColor = [UIColor blueColor];

	NSMutableString *emojiString = [NSMutableString stringWithFormat:@"%@", @"HAHFHDJKSAHFDA[大哭]JFDSAJFKDSA[大笑]DJFLAJLFD[大哭]HAHFDKJASFHDASHFDJASHFKDAHSJFDASHFJD[大笑][大笑][大笑]HFDHAJKFHDJSAHFDSJKA"];

	emojiString = [NSMutableString stringWithFormat:@"%@", @"[大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭][大哭]"];

	NSMutableAttributedString *mutableAttribute = [[NSMutableAttributedString alloc] init];
	NSMutableArray *attributes = [NSMutableArray arrayWithCapacity:10];

	NSString *regex = @"\\[[^\\[|^\\]]+\\]";
	NSRegularExpression *expression = [NSRegularExpression regularExpressionWithPattern:regex options:NSRegularExpressionCaseInsensitive error:nil];

	NSTextCheckingResult *result = [expression firstMatchInString:emojiString options:NSMatchingReportProgress range:NSMakeRange(0, emojiString.length)];


	while (result) {
	    NSRange range = [result rangeAtIndex:0];

	    NSString *prefixNotEmojiString = [emojiString substringToIndex:range.location];
	    NSString *currentEmojiString = [emojiString substringWithRange:range];

	    if (prefixNotEmojiString.length) {
	        NSAttributedString *attributeString = [[NSAttributedString alloc] initWithString:prefixNotEmojiString attributes:@{NSAttachmentAttributeName:[UIFont fontWithName:@"Arial" size:16]}];
	        [attributes addObject:attributeString];
	    }

	    if (currentEmojiString.length) {
	        NSTextAttachment *attachment = [[NSTextAttachment alloc] init];
	        attachment.image = [UIImage imageNamed:@"ebg"];
	        attachment.bounds = CGRectMake(-5, -5, 20, 20);

	        NSAttributedString *attributeString = [NSAttributedString attributedStringWithAttachment:attachment];
	        [attributes addObject:attributeString];
	    }

	    [emojiString replaceCharactersInRange:NSMakeRange(0, range.location) withString:@""];
	    [emojiString replaceCharactersInRange:NSMakeRange(0, range.length) withString:@""];
	    result = [expression firstMatchInString:emojiString options:NSMatchingReportProgress range:NSMakeRange(0, emojiString.length)];
	}

	[attributes enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
	    [mutableAttribute appendAttributedString:obj];
	}];
	[textView.textStorage appendAttributedString:mutableAttribute];

	CGSize size = [textView sizeThatFits:CGSizeMake(240, CGFLOAT_MAX)];
	CGRect frame = CGRectMake(100, 100, size.width, size.height);
	textView.frame = frame;
	[textView sizeToFit];

	[self.view addSubview:textView];

上面这段代码放到某一个ViewController中的ViewDidLoad中即可。但是现在代码就完了么?当然不是，写代码的时候一定想着代码的复用。我们可以为UITextView写一个关于表情的分类，也可以写一个工具类来完成表情字符串的解析，然后返回NSAttributeString，这样子代码就会比较好看一些了。
