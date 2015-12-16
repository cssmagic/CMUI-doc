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


## CSS 编码规范 <a name="css">&nbsp;</a>

#### 选择符 <a name="css--selector">&nbsp;</a>

* 选择符最末层不应该使用通配选择符（`*`）。

	> Stylint 配置：`{universal: 'never'}`

	```stylus
	// BAD
	#wrapper
		*
			display block

	// GOOD
	#wrapper
		div, p, a, span
			display block
	```

#### 值 <a name="css--value">&nbsp;</a>

* 当**长度值**为零时，应省略单位。

	> Stylint 配置：`{zeroUnits: 'never'}`

* `z-index` 的值必须是 10 的倍数。

	> Stylint 配置：`{zIndexNormalize: 10}`

* 尽可能精简那些可以自动展开的属性值：

	> Stylint 配置：`{efficient: 'always'}`
	
	```stylus
	#wrapper
		margin 0 0  // BAD
		margin 0 0 0 0  // BAD
		margin 0  // GOOD
		border-width 0 20px 0 20px  // BAD
		border-width 0 20px  // GOOD
	```

* 小数点前的零不省略：

	> Stylint 配置：`{leadingZero: 'always'}`
	
	```stylus
	#wrapper
		opacity .5  // BAD
		opacity 0.5  // GOOD
	```

* 优先使用 `none` 来关闭边框或描边样式：

	> Stylint 配置：`{none: 'always'}`
	
	```stylus
	#wrapper
		border 0  // BAD
		border none  // GOOD
	```

#### 声明 <a name="css--declaration">&nbsp;</a>

声明块中的各条声明需要以一定的顺序排列，以便快速浏览和定位。推荐顺序如下：

* 显示方式与布局方式
* 外边距与内边距
* 尺寸
* 文本样式
* 背景与边框
* 其它样式

```stylus
#wrapper
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
	
	// text, font and foreground color
	line-height 1.5
	font-weight 700
	color black
		
	// background and border
	background silver
	border-right 1px solid green
	
	// other
	cursor pointer
	animation-name my-anim
	backface-visibility hidden
```

#### 其它事项 <a name="css--other">&nbsp;</a>

（关于 CSS 的各种最佳实践，请参阅[《CSS 编码技巧 · CSS Secrets》](https://github.com/cssmagic/CSS-Secrets/issues/8)。）


## 文件系统 <a name="file-system">&nbsp;</a>

#### 文件命名 <a name="file-system--filename">&nbsp;</a>

Stylus 文件的扩展名为 `.styl`。

文件名由小写英文字母和数字组成，且必须以英文字母开头；单词之间以连字符分隔。比如 `text-style.styl`。

#### 字符集 <a name="file-system--charset">&nbsp;</a>

所有 Stylus 文件一律采用 UTF-8 字符集，文件无 BOM 头。

> EditorConfig 配置：`charset = utf-8`

所有 Stylus 文件内部一律不标记 `@charset`。如果页面本身没有采用 UTF-8 字符集，则在引用样式文件时需要在 `<link>` 标签上注明字符集：

```html
<link rel="stylesheet" href="cmui.css" charset="utf-8">
```


## 模块 <a name="module">&nbsp;</a>

#### 模块的组织 <a name="module--org">&nbsp;</a>

与 JavaScript 模块类似，每个样式模块对应一个物理文件。模块**必须**以 mixin 的方式组织，而不是以代码片断的方式组织。比如：

```stylus
// [module.styl]
// BAD
#wrapper
	color red
	border 1px solid
```

```stylus
// [module.styl]
// GOOD
my-mixin()
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

#### 大小写 <a name="code-style--letter-case">&nbsp;</a>

* 所有类型选择符一律小写。
* 所有属性名和关键字一律小写。

```stylus
// BAD
DIV, P, A
	COLOR RED

// GOOD
div, p, a
	color red
```

#### 代码块 <a name="code-style--code-block">&nbsp;</a>

* 每条声明独占一行，行尾不写分号。

	> Stylint 配置：`{stackedProperties: 'never', semicolons: 'never'}`

* 属性名与属性值之间无冒号，只留一个空格。

	> Stylint 配置：`{colons: 'never'}`

* 声明块（以及其它代码块）采用无花括号的风格，一律采用缩进来表示层级关系。

	> Stylint 配置：`{brackets: 'never'}`

	```stylus
	// BAD
	div {
		border: 1px solid;
		a {
			color: red; 
		}
	}

	// GOOD
	div
		border 1px solid
		a
			color red
	```

#### 缩进 <a name="code-style--indentation">&nbsp;</a>

* 采用一个 tab。

	> Stylint 配置：`{indentPref: false, mixed: true}`

	> EditorConfig 配置：`indent_style = tab`

#### 换行符 <a name="code-style--line-feed">&nbsp;</a>

* 换行符采用 `LF`。

	> EditorConfig 配置：`end_of_line = lf`

* 文件末尾至少要保留一个换行符。

	> EditorConfig 配置：`insert_final_newline = true`

#### 空格 <a name="code-style--space">&nbsp;</a>

* 行末不留空格。

	> Stylint 配置：`{trailingWhitespace: 'never'}`
	
	> EditorConfig 配置：`trim_trailing_whitespace = true`

* 括号内侧不加空格。

	> Stylint 配置：`{parenSpace: 'never'}`

* Mixin 名（以及函数名）与调用括号之间不加空格。

	```stylus
	// BAD
	my-mixin ( arg )

	// GOOD
	my-mixin(arg)
	```

* 减号的两侧需要用空格间隔，以便与连字符区分（取负运算符同理）：

	```stylus
	#wrapper
		$size = 100px
		margin-left $size-10  // ERROR
		margin-left ($size - 10)  // GOOD
	```

* 逗号在用作分隔符时，其后必须加一个空格：

	> Stylint 配置：`{commaSpace: 'always'}`

	```stylus
	#wrapper
		color rgba(0,0,0,0.5)  // BAD
		color rgba(0, 0, 0, 0.5)  // GOOD
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

> Stylint 配置：`{quotePref: 'single'}`

> EditorConfig 配置：`quote_type = single` 

写在 `url()` 函数内的 URL 是不需要包一层引号的：

```stylus
#wrapper
	background-image url('http://file.baixing.net/logo.png')  // BAD
	background-image url(http://file.baixing.net/logo.png)  // GOOD
```


## 注释 <a name="comment">&nbsp;</a>

#### 常规注释 <a name="comment--general">&nbsp;</a>

代码中的常规注释优先选择**单行注释**（`// comment`），而不是原生 CSS 中的多行注释风格（`/* comment */`）。

单行注释的双斜杠之后空一格。

> Stylint 配置：`{commentSpace: 'always'}`

针对代码块的注释独占一行，写在代码块的顶部；针对某个声明（或选择符）的注释写在声明（或选择符）的右侧，与声明之间用一个 tab 间隔（即 Elastic tabstops 风格）。

```stylus
// comment to a code block
#wrapper
	color red
	
	// comment to a code block
	.foo,	// comment to a selector
	.bar
		color green
		border 1px solid	// comment to this declaration
```

#### 特殊注释 <a name="comment--special">&nbsp;</a>

分隔线是一种特殊的注释，用于把文件划分为多个区段；或者说，它用于把多个代码块分组。分隔线有两种层级，两者的配合使用可以提高代码的组织能力。（尽管分隔线很有用，但我们应该优先通过纵深化的树形结构来体现代码块之间的独立关系。）

区段标记（`/** section mark **/`）是另一种特殊的注释，用于描述各个区段的名称或作用。

分隔线和区段标记的使用示例如下：

```stylus
/* ============================================= */
/** icon **/
.icon
	...
/* --------------------------------------------- */
/** icon size **/
.icon
	...
	&.large
		...
	&.small
		...
/* --------------------------------------------- */
/** icon image **/
...

/* ============================================= */
/** btn **/
...
```

建议在 IDE 或编辑器中将上述特殊注释设置为可以快速输入的代码片断（比如 WebStorm 中的 Live template）。


## 选择符 <a name="selector">&nbsp;</a>

#### 嵌套 <a name="selector--nesting">&nbsp;</a>

尽可能利用选择符嵌套，来把代码归纳为树形结构。

```stylus
// BAD
#wrapper
	border 1px solid
#wrapper a
	color red

// GOOD
#wrapper
	border 1px solid
	a
		color red
```

#### 父级引用 <a name="selector--parent-ref">&nbsp;</a>

除了类、伪类、伪元素、属性选择符等情况之外，父级引用往往是不需要的，建议精简：

```stylus
#wrapper
	border 1px solid

	// BAD
	& > a
		color red

	// GOOD
	> a
		color red
```

#### 群组选择符 <a name="selector--group">&nbsp;</a>

单纯由类型选择符所构成的群组选择符可以写在一行；但复杂的群组选择符必须分行：

```stylus
// BAD
#wrapper
	p.warning, h3.highlight
		color red

// OK
#wrapper
	a, p, span
		color red
```

当分行时，群组内的多个选择符之间建议总是写上逗号：

```stylus
// GOOD
#wrapper
	p.warning,
	h3.highlight
		color red
```

仅当确定不会产生歧义或解析错误时，才可以省略逗号：

```stylus
// OK
#wrapper
	p.warning
	h3.highlight
		color red
```

> 注：以下情况存在歧义或解析错误：

> ```stylus
> // BAD
> #wrapper
> 	foo bar  // => foo: bar;
> 	h3.highlight
> 		color red
> 	
> // ERROR (ParseError on Stylus v0.x, maybe accepted on v1.x)
> #wrapper
> 	foo > bar
> 	h3.highlight
> 		color red
> ```


## 变量 <a name="variable">&nbsp;</a>

#### 命名 <a name="variable--naming">&nbsp;</a>

变量必须以 `$` 作为前缀。除前缀外，变量名由全小写英文字母和数字组成，且前缀后的第一个字符必须是英文字母；单词之间以连字符分隔。比如：`$color-bg`。

> Stylint 配置：`{prefixVarsWithDollar: 'always'}`

#### 作用域 <a name="variable--scope">&nbsp;</a>

变量是有作用域的。

作为公开 API 提供的变量必须放置在全局作用域，即成为全局变量。

* CMUI 核心层提供的公开的全局变量必须以 `$cm-` 开头，比如 `$cm-color-fg`。

* CMUI 主题层提供的公开的全局变量必须以具备模块特征的前缀开头，比如对 `Baixing` 主题来说，`$bx-color-gray` 就是个不错的变量名。

非公开的变量应该被限制在一定的作用域内（成为局部变量），且建议使用 `$-` 前缀：

```stylus
my-mixin()
	$-var1 = 10px  // limited in a mixin

#wrapper
	$-var2 = 20px  // limited in a selector
```

如果某个变量不属于公开 API，但由于需要被多个根级代码块共享而不得不暴露到全局作用域，则**必须**使用 `$-` 前缀：

```stylus
$-shared-var = 10px

my-mixin()
	size $-shared-var

another-mixin()
	line-height $-shared-var
```


## Mixin <a name="mixin">&nbsp;</a>

#### 命名 <a name="mixin--naming">&nbsp;</a>

Mixin 名由全小写英文字母和数字组成，且必须以英文字母开头；单词之间以连字符分隔。比如 `my-mixin()`。

#### 定义 <a name="mixin--define">&nbsp;</a>

Mixin 内部的选择符**不写**不必要的父级引用：

```stylus
// BAD
my-mixin()
	// unnecessary
	&
		color red
	// unnecessary
	& .foo
		color green
	// Note: this parent reference is necessary
	&.bar
		color yellow

// GOOD
my-mixin()
	color red
	.foo
		color green
	&.bar
		color yellow
```

#### 调用 <a name="mixin--invoke">&nbsp;</a>

Mixin 在调用时必须使用括号。比如：

```stylus
.foo
	my-mixin()
```

（注意：“[透明 mixin](http://stylus-lang.com/#transparent-mixins)” 不在此列，仍以类似属性声明的方式书写。）

Mixin 在调用时必须位于当前代码块的最顶部：

```stylus
my-mixin()
	color red

// BAD
.bar
	font-weight bold
	my-mixin()
	
// GOOD
.foo
	my-mixin()
	font-weight bold
```

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

非公开的（即只在内部使用的）mixin 应该被限制在一定的作用域内（成为局部 mixin），且建议使用 `-` 前缀：

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
	-icon-size($size = 16px)  // defined in local scope
		size $size

	-icon-size()
	&.large
		-icon-size(32px)
```

如果某个 mixin 不属于公开 API，但由于需要被多个根级代码块共享而不得不暴露到全局作用域，则**必须**使用 `-` 前缀。


#### 接口 <a name="mixin--api">&nbsp;</a>

如果某个 mixin 的作用等同于某个作为公开 API 存在的类名，则应该与该类名同名，比如用 `cmBtn()` 对应 `.cmBtn`。


## 函数 <a name="function">&nbsp;</a>

#### 命名 <a name="function--naming">&nbsp;</a>

（参见 [mixin 的命名](#mixin--naming)。）

#### 参数 <a name="function--argument">&nbsp;</a>

（参见 [mixin 的参数](#mixin--argument)。）

#### 作用域 <a name="function--scope">&nbsp;</a>

（参见 [mixin 的作用域](#mixin--scope)。）


## 不建议使用的功能 <a name="deprecated">&nbsp;</a>

#### Extend <a name="deprecated--extend">&nbsp;</a>

Extend 功能会重新组织代码顺序，可能会导致无法预料的结果。因此禁用此功能，改用 mixin 来实现类似的代码组织功能：

```stylus
// BAD
.class-a
	font-weight bold
	...
.class-b
	@extend .class-a
	color green
	...

// OK
.class-a,
.class-b
	font-weight bold
	...
.class-b
	color green
	...

// GOOD
my-mixin()
	font-weight bold
	...
.class-a
	my-mixin()
.class-b
	my-mixin()
	color green
	...
```

在上面的示例中，第一段代码使用了 extend 功能，最终生成的 CSS 代码量较少；第二段代码生成的结果与第一段相同，相当于通过人工预先归并代码的方式来减少冗余度，但不易阅读和维护；第三段代码最为清晰，但生成的代码存在冗余部分。

最终，我们选择第三种方式，因为代码冗余会在 Gzip 阶段消化掉，而代码的可维护性永远是第一位的。

#### Placeholder <a name="deprecated--placeholder">&nbsp;</a>

Placeholder 并不是一个独立的功能，它实际上是 extend 的一种高级应用形式。基于相同的原因，禁用此功能，改用 mixin 来实现类似的代码组织功能：

```stylus
// BAD
$my-placeholder
	font-weight bold
	...
.class-a
	@extend $my-placeholder
.class-b
	@extend $my-placeholder
	color green
	...

// GOOD
my-mixin()
	font-weight bold
	...
.class-a
	my-mixin()
.class-b
	my-mixin()
	color green
	...
```


#### Block

Block 所能提供的功能是 mixin 的子集，没有额外优点。为降低复杂度，禁用此功能。

```stylus
// BAD
my-block =
	font-weight bold

#wrapper
	{my-block}

// GOOD
my-mixin()
	font-weight bold

#wrapper
	my-mixin()
```

#### CSS Literal <a name="deprecated--css-literal">&nbsp;</a>

即 CSS 字面量，由 `@css` 代码块来定义，其内部代码不会被 Stylus 引擎处理，只会原样输出。实际开发中暂未发现必须使用此功能的场景，为降低复杂度，禁用此功能。

> Stylint 配置：`{cssLiteral: 'never'}`

如果存在大段遗留的 CSS 代码需要整合到 Stylus 文件中，建议先将 CSS 文件转换为 Stylus 文件（以下操作会在当前目录下得到 `foo.styl` 文件）：

```sh
$ stylus -C foo.css
```


## 其它 <a name="other">&nbsp;</a>

#### 其它 Stylint 配置 <a name="other--stylint-config">&nbsp;</a>

* `{valid: true}` - 属性名、值、选择符必须是有效值。
* `{mixins: ['...', '...']}` - 指定自定义的透明 mixin。

#### 代码压缩 <a name="other--compression">&nbsp;</a>

静态资源服务器在响应所有 CSS 文件时必须做 Gzip 压缩。

由于风险太大，收益不高，在 minify 阶段放弃任何形式的 “CSS 代码高级压缩” 功能，包含规则合并、声明去重等等。

***

#### 相关阅读

* [Stylus 官方文档](http://stylus-lang.com/)

***

（如有任何意见，请直接回复。）
