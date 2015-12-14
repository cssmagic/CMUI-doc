---
title: "CMUI 之 Stylus 编码规范（草案）"
---

## 基本概念 <a name="basis">&nbsp;</a>

对网站项目来说，所有 Stylus 文件分为两类：

* **入口文件**：被页面直接引用的文件称为入口文件。
* **模块文件**：相对独立的样式代码块可以以模块的方式抽象出来，以便被入口文件或其它模块引用（`@import`）。

显然，只有入口文件是需要被编译成 CSS 文件的。

#### CMUI <a name="basis--cmui">&nbsp;</a>

可以将 CMUI 各主题的 `index.styl` 文件视为入口文件，也可以视为模块文件（由众多小模块构成的大模块）。这完全取决于使用者是直接使用编译好的 `dist/*.css` 文件，还是以引用模块的方式来使用各主题的 `index.styl` 文件。（百姓网手机站采用的是后一种方式。）


## CSS 编码规范 <a name="css-guideline">&nbsp;</a>

#### 声明 <a name="css-guideline--declaration">&nbsp;</a>

声明块中的各条声明需要以一定的顺序排列，以便快速浏览和定位。推荐顺序如下：

* 显示方式与布局方式
* 外边距与内边距
* 尺寸
* 文本样式
* 背景与边框
* 其它样式

```stylus
#my-element
	// display and layout
	display block
	position relative
	z-index 10
	float left
	clear both
	
	// margin and padding
	margin-top 10px
	padding 10px
	
	// measurement
	height 100px
	min-width 500px
	
	// text and font
	line-height 1.5
	font-weight 700
	color black
		
	// background and border
	background silver
	border-right 1px solid green
	
	// other
	animation-name my-anim
```


#### 其它事项 <a name="css-guideline--other">&nbsp;</a>

（关于 CSS 的各种最佳实践，请参阅[《CSS 编码技巧 · CSS Secrets》](https://github.com/cssmagic/CSS-Secrets/issues/8)。）


## 文件系统 <a name="file-system">&nbsp;</a>

#### 字符集 <a name="file-system--charset">&nbsp;</a>

所有 Stylus 文件一律采用 UTF-8 字符集，文件无 BOM 头。

所有 Stylus 文件内部一律不标记 `@charset`。如果页面本身没有采用 UTF-8 字符集，则在引用样式文件时需要在 `<link>` 标签上注明字符集：

```html
<link rel="stylesheet" href="cmui.css" charset="utf-8">
```

#### 文件命名 <a name="file-system--filename">&nbsp;</a>

Stylus 文件的扩展名为 `.styl`。

文件名由小写英文字母和数字组成，且必须以英文字母开头；单词之间以连字符分隔。比如 `text-style.styl`。

## 模块 <a name="module">&nbsp;</a>

#### 模块的组织 <a name="module--org">&nbsp;</a>

与 JavaScript 模块类似，每个样式模块对应一个物理文件。模块**必须**以 mixin 的方式组织，而不是以代码片断的方式组织。比如：

```stylus
// [module.styl]
// GOOD
my-mixin()
	color red
	border 1px solid
```

```stylus
// [module.styl]
// BAD
#wrapper
	color red
	border 1px solid
```

这意味着在导入模块之后，需要手动调用模块中的 mixin。比如，模块的内容是这样的：

```stylus
// [module.styl]
my-mixin()
	color red
	border 1px solid
```

而入口文件是这样调用模块的：

```stylus
// [entry.styl]
@import './module'

#wrapper
	my-mixin()
```

或这样的（推荐这种方式，因为这会将模块内的 mixin 导入到局部作用域，可以有效避免同名 mixin 可能引起的冲突）：

```stylus
// [entry.styl]
#wrapper
	@import './module'
	my-mixin()
```

每个模块可包含一个或多个 mixin，建议只包含一个。不相关的多个 mixin 不应组织到同一个模块中。

#### 模块的导入 <a name="module--import">&nbsp;</a>

仅使用 `@import` 来导入模块，不使用 `@require`。

模块的路径与文件名需要用引号包住。文件路径总是以 `./` 开头，文件名不需要包含 `.styl` 扩展名：

```stylus
@import './modules/ui/index'
```


## 代码风格 <a name="code-style">&nbsp;</a>

#### 基本风格 <a name="code-style--basic">&nbsp;</a>

大小写：

* 所有类型选择符一律小写。
* 所有属性名和关键字一律小写。

```stylus
// GOOD
div, p, a
	color red

// BAD
DIV, P, A
	COLOR RED
```

代码块：

* 每条声明独占一行，行尾不写分号。
* 属性名与属性值之间无冒号，只留一个空格。
* 声明块（以及其它代码块）采用无花括号的风格，一律采用缩进来表示层级关系。

```stylus
// GOOD
div
	border 1px solid
	a
		color red

// BAD
div {
	border: 1px solid;
	a {
		color: red; 
	}
}
```

#### 空白符 <a name="code-style--whitespace">&nbsp;</a>

常规设置：

* 换行符采用 `LF`。
* 缩进采用一个 tab。

关于空格：

* 括号内侧不加空格。
* 函数名与调用括号之间不加空格。
* Mixin 名与调用括号之间不加空格。

```stylus
// GOOD
my-mixin(arg)

// BAD
my-mixin ( arg )
```

* 减号的两侧需要用空格间隔，以便与连字符区分（取负运算符同理）：

```stylus
#wrapper
	$size = 100px
	margin-left $size-10  // ERROR
	margin-left ($size - 10)  // GOOD
```

#### 括号 <a name="code-style--parenthesis">&nbsp;</a>

在需要传入一个值的地方使用表达式时，需要把表达式用括号括起来：

```stylus
#wrapper
	$size = 100px
	margin-left - $size * 0.5  // BAD
	margin-left (- $size * 0.5)  // GOOD
	
	padding-left $size / 0.5  // BAD
	padding-left ($size / 0.5)  // GOOD
```

#### 其它字符 <a name="code-style--other-char">&nbsp;</a>

引号一律使用单引号。

写在 `url()` 函数内的 URL 是不需要包一层引号的：

```stylus
// GOOD
background-image url(http://file.baixing.net/logo.png)

// BAD
background-image url('http://file.baixing.net/logo.png')
```


## 选择符 <a name="selector">&nbsp;</a>

#### 嵌套 <a name="selector--nesting">&nbsp;</a>

尽可能利用选择符嵌套，来把代码归纳为树形结构。

```stylus
// GOOD
div
	border 1px solid
	a
		color red

// BAD
div
	border 1px solid
div a
	color red
```

#### 父级引用 <a name="selector--parent-ref">&nbsp;</a>

除了类、伪类、伪元素、属性选择符等情况之外，父级引用往往是不需要的，建议精简：

```stylus
div
	border 1px solid

	// GOOD
	> a
		color red

	// BAD
	& > a
		color red

```

#### 群组选择符 <a name="selector--group">&nbsp;</a>

单纯由类型选择符所构成的群组选择符可以写在一行；但复杂的群组选择符必须分行：

```stylus
// OK
div
	a, p, span
		color red

// BAD
div
	p.warning, h3.highlight
		color red
```

当分行时，群组内的多个选择符之间建议总是写上逗号：

```stylus
// GOOD
div
	p.warning,
	h3.highlight
		color red
```

仅当确定不会产生歧义或解析错误时，才可以省略逗号：

```stylus
// OK
div
	p.warning
	h3.highlight
		color red
```

> 注：以下情况存在歧义或解析错误：

> ```stylus
> // BAD
> div
> 	foo bar  // => foo: bar;
> 	h3.highlight
> 		color red
> 	
> // ERROR (ParseError on Stylus v0.x, maybe accepted on v1.x)
> div
> 	foo > bar
> 	h3.highlight
> 		color red
> ```


## 变量 <a name="variable">&nbsp;</a>

#### 命名 <a name="variable--naming">&nbsp;</a>

变量必须以 `$` 作为前缀。除前缀外，变量名由全小写英文字母和数字组成，且前缀后的第一个字符必须是英文字母；单词之间以连字符分隔。比如：`$color-bg`。

#### 作用域 <a name="variable--scope">&nbsp;</a>

变量是有作用域的。

作为公开 API 提供的变量必须放置在全局作用域，即成为全局变量。

* CMUI 核心层提供的公开的全局变量必须以 `$cm-` 开头，比如 `$cm-color-fg`。

* CMUI 主题层提供的公开的全局变量必须以具备模块特征的前缀开头，比如对 `Baixing` 主题来说，`$bx-color-gray` 就是个不错的变量名。

非公开的变量应该被限制在一定的作用域内（成为局部变量），且建议使用  `$-` 前缀：

```stylus
my-mixin()
	$-var1 = 10px  // limited in a mixin

#wrapper
	$-var2 = 20px  // limited in a selector
```

## Mixin <a name="mixin">&nbsp;</a>

#### 命名 <a name="mixin--naming">&nbsp;</a>

Mixin 名由全小写英文字母和数字组成，且必须以英文字母开头；单词之间以连字符分隔。比如 `my-mixin()`。

#### 调用 <a name="mixin--invoke">&nbsp;</a>

Mixin 在调用时必须使用括号。比如：

```stylus
.foo
    my-mixin()
```

（注意：“[透明 mixin](http://stylus-lang.com/#transparent-mixins)” 不在此列，仍以类似属性声明的方式书写。）

#### 参数 <a name="mixin--argument">&nbsp;</a>

Mixin 在定义时，其参数必须以 `$` 开头。

参数名由英文字母和数字组成，采用小驼峰拼写方式。比如 `$myParam`。

#### 作用域 <a name="mixin--scope">&nbsp;</a>

Mixin 是有作用域的。

作为公开 API 提供的 mixin 必须暴露到全局作用域，即成为全局 mixin。具体实现方法如下：

```stylus
// [module.styl]
my-mixin()
	color red
	border 1px solid
```

```stylus
// [entry.styl]
@import './module'  // `my-mixin()` will be a global mixin

another-mixin()  // defined as a global mixin
	color green
```

非公开的（即只在内部使用的）mixin 应该被限制在一定的作用域内（成为局部 mixin），且使用 `-` 前缀，比如 `-my-temp-mixin()`。示例如下：

```stylus
// [entry.styl]
// BAD
-icon-size($size = 16px)  // leaked to global scope
	size $size

.icon
	-icon-size()
	&.large
		-icon-size(32px)

// GOOD
.icon
	-icon-size($size = 16px)  // defined as a local mixin
		size $size

	-icon-size()
	&.large
		-icon-size(32px)
```

#### 接口 <a name="mixin--api">&nbsp;</a>

如果某个 mixin 的作用等同于某个作为公开 API 存在的类名，则应该与该类名同名，比如用 `cmBtn()` 对应 `.cmBtn`。

## 函数 <a name="function">&nbsp;</a>

#### 命名 <a name="function--naming">&nbsp;</a>

（参见 [mixin 的命名](#mixin--naming)。）

#### 参数 <a name="function--argument">&nbsp;</a>

（参见 [mixin 的参数](#mixin--argument)。）

#### 作用域 <a name="function--scope">&nbsp;</a>

（参见 [mixin 的作用域](#mixin--scope)。）

***

#### 相关阅读

* [Stylus 官方文档](http://stylus-lang.com/)

***

（如有任何意见，请直接回复。）
