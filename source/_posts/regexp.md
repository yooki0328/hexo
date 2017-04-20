---
title: 正则表达式
date: 2016-09-23 03:49:39
tags: [regexp]
---
### **什么是 RegExp？**
RegExp 是正则表达式的缩写。
当您检索某个文本时，可以使用一种模式来描述要检索的内容。RegExp 就是这种模式。
简单的模式可以是一个单独的字符。
更复杂的模式包括了更多的字符，并可用于解析、格式检查、替换等等。
您可以规定字符串中的检索位置，以及要检索的字符类型，等等。<!--more-->
<table><thead><tr><th>表达式</th><th>含义</th></tr></thead><tbody><tr><td>\b</td><td>代表着**_单词的开头或结尾(单词分界处)_**</td></tr><tr><td>\d</td><td>匹配一位数字</td></tr><tr><td>{n}</td><td>重复匹配n次</td></tr><tr><td>^</td><td>匹配输入字符串的开始位置</td></tr><tr><td>$</td><td>匹配字符串结束位置</td></tr><tr><td>*</td><td>匹配前面的表达式0次或多次</td></tr><tr><td>+</td><td>匹配前面的表达式一次或多次</td></tr><tr><td>?</td><td>匹配前面的表达式0次或一次,跟在任何一个其他限制符后面时,匹配模式是非贪婪的</td></tr><tr><td>{n,m}</td><td>至少匹配n次最多匹配m次</td></tr><tr><td>.</td><td>匹配除”\n”之外的任何单个字符</td></tr><tr><td>\w</td><td>匹配包括下划线的任何单词字符</td></tr><tr><td>\W</td><td>匹配任何非单词字符</td></tr></tbody></table>
(?0\d{2}[) -]?\d{8}
这个表达式可以匹配几种格式的电话号码，像(010)88886666，或022-22334455，或02912345678等。我们对它进行一些分析吧：首先是一个转义字符(,它能出现0次或1次(?),然后是一个0，后面跟着2个数字(\d{2})，然后是)或-或空格中的一个，它出现1次或不出现(?)，最后是8个数字(\d{8})。
### 分支条件|
(?=exp)也叫零宽度正预测先行断言
(?<=exp)也叫零宽度正回顾后发断言
零宽度负预测先行断言(?!exp)
零宽度负回顾后发断言(?<!exp)
(?<=<(\w+)>).*(?=<\/\1>)匹配不包含属性的简单HTML标签内里的内容
js 中，正则表达式后面可以跟三个 flag，比如 /something/igm

* i 的意义是不区分大小写
* g 的意义是，匹配多个
* m 的意义是，是 ^ 和 $ 可以匹配每一行的开头
```javascript
/a/.test(‘A’) // => false
/a/i.test(‘A’) // => true
‘hello hell hoo’.match(/h.?\b/) // => [ ‘hello’, index: 0, input: ‘hello hell hoo’ ]
‘hello hell hoo’.match(/h.?\b/g) // => [ ‘hello’, ‘hell’, ‘hoo’ ]
‘aaa\nbbb\nccc’.match(/^[\s\S]?$/g) // => [ ‘aaa\nbbb\nccc’ ]
‘aaa\nbbb\nccc’.match(/^[\s\S]?$/gm) // => [ ‘aaa’, ‘bbb’, ‘ccc’ ]
```
直接用 . 匹配不到 \n，所以我们需要找到一个原子，能匹配包括 \n 在内的所有字符。这个原子的惯用写法就是 [\s\S]
```javascript
var match1 = text.match(/^```[\s\S]+?^```/gm);
console.log(match1) // => [ '```\ncode code2 code3```\n```' ]

// 这里有一种很骚的写法，[^] 与 [\s\S] 等价
var match2 = text.match(/^```[^]+?^```/gm)
console.log(match2) // => [ '```\ncode code2 .code3```\n```' ]
```
