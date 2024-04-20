---
title: 一个复制粘贴的小技巧
date: 2020-02-26 01:08:42
author: repoog
excerpt: 使用Javascript在用户进行双击操作时候自动复制文本是很多网站具备的功能，本例来自对于百度一处复制操作的Bug，从而发现可以使用更为简单的办法实现相同的功能。
comments: true
tags:
  - clipboardData
  - font-size
  - Javascript
  - 复制粘贴
categories:
  - 软件开发
---

通过JS在复制操作时添加尾巴什么的，常见的思路是利用Selection.selectAllChildren()、window.clipboardData.setData()或者window.execcommand('copy')。比如下面这段代码：

``` Javascript
<p id="my_text">
this is a copy text.
</p>

<script>
document.getElementById("my_text").addEventListener('copy', function(e){
    const copy_obj = document.getSelection();
    e.clipboardData.setData('text/plain', copy_obj.toString() + "\nCopyright to 2020.");
    e.preventDefault();
});
</script>
```

但复制操作很多时候也会通过双击文本来自动选择，在Chrome浏览器中，双击文本会自动选择分词的文本或标签内的整段文本。在这个操作方式下，还有另外一种简单的办法来添加文本尾巴。上面的例子可以简单写为：

``` HTML
<p id="my_text">
this is a copy text.<span style="font-size:0"> Copyright to 2020.</span>
</p>
```

只要将添加的文本的font-size设置为0，便可实现文本不可见，双击复制粘贴时会自动添加尾巴的效果。但这种方式只适用于简单的文本添加，而无法实现HTML格式的尾巴。