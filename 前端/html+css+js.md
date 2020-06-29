# 1. html

> W3C标准

- 结构化标准语言 （HTML、XML）
- 表现标准语言（CSS）
- 行为标准（DOM，ECMAScript）



> 网页基本信息

- DOCTYPE：告诉浏览器，我们要使用的规范是什么。默认 **html**。
- meta：描述性标签。**<font color='red'>描述网站的元信息</font>**。



> 基本标签

- 段落标签：<p></p>
- 换行标签：<br/>
- 水平线标签：<hr/>
- 字体样式：
    - 粗体：<strong></strong>
    - 斜体：<em></em>
- 注释和特殊符号
    - 注释： <!-- -->
    - 空格：%nbsp；
    - &gt；
    - &lt；
    - &copy；



> 图像标签

```html
<img src="path" alt="text" title="text" width="x" height=""/>
- path: url
- 图像的代替文件
- 鼠标悬停提示文字
- 图像宽度
- 图像高度
```



> 链接标签



```html
<a href="path" target="">文字</a>
- targer： 表示窗口在哪里打开
		 _blank： 在新标签打开；默认是当前页面打开
		 _self： 在当前网页下打开
```

> > 锚链接



```html
<!-- 定义 锚链接的点： 用name 定义-->
<a name="here">锚链接展示</a>

<!-- 加 # 跳转锚链接 -->
<a href="图像标签.html#here">跳转到锚链接</a>
```

> >功能性链接



邮箱链接：

```html
<a href="mailto:948209147@qq.com">发邮件给我</a>
```

qq链接：

```html
<a target="_blank" href="http://wpa.qq.com/msgrd?v=3&uin=&site=qq&menu=yes"><img border="0" src="http://wpa.qq.com/pa?p=2::53" alt="给我发消息把！" title="给我发消息把！"/></a>
```

