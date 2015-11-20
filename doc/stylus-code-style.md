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

所有 Stylus 文件一律采用 UTF-8 字符集。

所有 Stylus 文件内部一律不标记 `@charset`。如果页面本身没有采用 UTF-8 字符集，则在引用样式时需要在 `<link>` 标签上注明字符集。

#### 模块 <a name="file-system--module">&nbsp;</a>

与 JavaScript 模块类似，每个样式模块对应一个物理文件。模块**必须**以 mixin 的方式组织，而不是以代码片断的方式组织。比如：

```stylus
// GOOD
// module.styl
my-mixin()
	color red
	border 1px solid
```

```stylus
// BAD
// module.styl
#wrapper
	color red
	border 1px solid
```

这意味着在导入模块之后，需要手动调用模块中的 mixin。比如，模块的内容是这样的：

```stylus
// module.styl
my-mixin()
	color red
	border 1px solid
```

而入口文件是这样调用模块的：

```stylus
// entry.styl
@import './module'

#wrapper
	my-mixin()
```

或这样的（推荐）：

```stylus
// entry.styl
#wrapper
	@import './module'
	my-mixin()
```

每个模块可包含一个或多个 mixin，建议只包含一个。不相关的多个 mixin 不应组织到同一个模块中。

#### 文件命名 <a name="file-system--filename">&nbsp;</a>

文件名必须为全小写，扩展名为 `.styl`。

入口文件的文件名建议与所在页面的功能（或 URL）相同或相近，比如首页的样式入口文件取名为 `home.styl`，用户中心各页面的样式入口文件取名为 `wo_*.styl`。

如果模块文件只包含一个 mixin 的话，模块的文件名建议取其包含的 mixin 的名字，以增强文件名的可读性。

## 代码风格 <a name="code-style">&nbsp;</a>

#### 基本风格 <a name="code-style--basic">&nbsp;</a>

* 声明结尾无分号（这意味着所有每条声明均独立一行）
* 属性名与属性值之间无冒号，只留一个空格
* 代码块（声明块和其它代码块）无花括号，一律采用缩进来表示层级关系

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

减号与变量、数值、函数、mixin 之间需要用空格间隔，以便于连字符区分；取负运算符同理。在取负值时，最好将取负的部分用括号括起来：

```stylus
#wrapper
	$size = 100px
	margin-left -$size * 0.5  // ERROR
	margin-left - $size * 0.5  // OK
	margin-left -($size * 0.5)  // GOOD
	margin-left (- $size * 0.5)  // GOOD
```

#### 其它字符 <a name="code-style--other-char">&nbsp;</a>

* 引号一律使用单引号。

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

群组选择符建议分行写。当分行时，群组内的多个选择符之间通常可以省略逗号。

```stylus
// GOOD
div
	p.warning,
	h3.highlight
		color red

// OK
div
	p.warning
	h3.highlight
		color red
```

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

**注意**：当分行的群组选择符与属性声明（或表达式）之间产生歧义时，不可省略分号，以免产生意外的解析结果（或解析错误）：

```stylus
// GOOD
div
	foo bar,
	h3.highlight
		color red

// BAD
div
	foo bar  // => foo: bar;
	h3.highlight
		color red
```

```stylus
// GOOD
div
	foo > bar,
	h3.highlight
		color red

// ERROR (ParseError on Stylus v0.x, maybe accepted on v1.x)
div
	foo > bar
	h3.highlight
		color red
```

因此，**建议在群组内的多个选择符之间总是写上逗号**。


## 变量 <a name="variable">&nbsp;</a>

#### 命名 <a name="variable--naming">&nbsp;</a>

全小写字母，单词之间以连字符分隔。

变量必须以 `$` 开头。比如：

```stylus
$color-bg = white
```
#### 作用域 <a name="variable--scope">&nbsp;</a>

变量是有作用域的。

作为公开 API 提供的变量必须放置在全局作用域，即成为全局变量。

* CMUI 核心层提供的公开的全局变量必须以 `$cm-` 开头，比如 `$cm-color-fg`。

* CMUI 主题层提供的公开的全局变量必须以具备模块特征的前缀开头，比如对 `Baixing` 主题来说，`$bx-color-gray` 就是个不错的变量名。

非公开的变量应该被限定在一定的作用域内（成为局部变量），且建议使用  `$-` 前缀。如果不得不暴露到全局作用域，则**必须**使用 `$-` 前缀，并应该避免过于宽泛的命名（加上模块名作为辅助前缀是个不错的主意，比如 `$-btn-height`）。

## Mixin <a name="mixin">&nbsp;</a>

#### 调用 <a name="mixin--invoke">&nbsp;</a>

Mixin 在调用时必须使用括号。比如：

```stylus
.foo
    my-mixin()
```

注意：“[透明 mixin](http://stylus-lang.com/#transparent-mixins)” 不在此列，仍以类似属性声明的方式书写。

#### 参数 <a name="mixin--argument">&nbsp;</a>

Mixin 在定义时，其参数必须以 `$` 开头。

#### 作用域 <a name="mixin--scope">&nbsp;</a>

Mixin 是有作用域的。

模块内部使用的 mixin（即不是作为公开 API 存在的 mixin）应该被限定在一定的作用域内（成为局部 mixin），且建议使用 `-` 前缀，比如 `-my-temp-mixin()`。

如果不得不暴露到全局作用域，则**必须**使用 `-` 前缀，并应该避免过于宽泛的命名（加上模块名作为辅助前缀是个不错的主意，比如 `-btn-style()`）。

#### 接口 <a name="mixin--api">&nbsp;</a>

如果某个 mixin 的作用等同于某个作为公开 API 存在的类名，则应该与该类名同名，比如用 `cmBtn()` 对应 `.cmBtn`。

## 函数 <a name="function">&nbsp;</a>

#### 参数 <a name="function--argument">&nbsp;</a>

函数在定义时，其参数必须以 `$` 开头。

#### 作用域 <a name="function--scope">&nbsp;</a>

函数是有作用域的。

模块内部使用的函数（即不是作为公开 API 存在的函数）应该被限定在一定的作用域内（成为局部函数），且建议使用 `-` 前缀，比如 `-my-temp-fn()`。

如果不得不暴露到全局作用域，则**必须**使用 `-` 前缀，并应该避免过于宽泛的命名（加上模块名作为辅助前缀是个不错的主意，比如 `-btn-get-type()`）。

***

#### 相关阅读

* [Stylus 官方网站](http://stylus-lang.com/)

***

（如有任何意见，请直接回复。）
