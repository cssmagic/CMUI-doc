# Stylus 编码规范（草案）

## CSS 编码规范

（请参考各种最佳实践，本文不再赘述。）

## 文件系统

#### 字符集

所有 Stylus 文件一律采用 UTF-8 字符集。

#### 模块

与 JavaScript 模块类似，每个样式模块对应一个物理文件。模块推荐以 mixin 的方式组织，而不是以代码片断的方式组织。（这意味着在导入模块之后，需要手动调用模块中的 mixin。）

示例：

```scss
// module.styl
my-mixin()
	color red
	border 1px solid
```

```scss
// page.styl
@import './module'

#wrapper
	my-mixin()
```

每个模块可包含一个或多个 mixin。不相关的多个 mixin 不应组织到同一个模块中。

## 代码风格

#### 基本风格

* 声明结尾无分号（这意味着所有每条声明均独立一行）
* 属性名与属性值之间无冒号，只留一个空格
* 代码块（声明块和其它代码块）无花括号，一律采用缩进来表示层级关系

示例：

```scss
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

#### 空白符

* 换行符采用 `LF`。
* 缩进采用一个 tab。

关于空格：

* 括号内侧不加空格。
* 函数名与调用括号之间不加空格。
* Mixin 名与调用括号之间不加空格。

示例：

```scss
// BAD
my-mixin ( arg )

// GOOD
my-mixin(arg)
```

#### 其它字符

* 引号一律使用单引号。

## 变量

全小写字母，单词之间以连字符分隔。

变量必须以 `$` 开头。比如：

```sass
$color-bg = white
```
### 作用域

变量是有作用域的。

作为公开 API 提供的变量必须放置在全局作用域，即成为全局变量。

* CMUI 核心层提供的公开的全局变量必须以 `$cm-` 开头，比如 `$cm-color-fg`。

* CMUI 主题层提供的公开的全局变量必须以具备模块特征的前缀开头，比如对 `Baixing` 主题来说，`$bx-color-gray` 就是个不错的变量名。

非公开的变量应该被限定在一定的作用域内（成为局部变量），且建议使用  `$-` 前缀。如果不得不暴露到全局作用域，则**必须**使用 `$-` 前缀，并应该避免过于宽泛的命名（加上模块名作为辅助前缀是个不错的主意，比如 `$-btn-height`）。

## Mixin

Mixin 在调用时必须使用括号。比如：

```sass
.foo
    my-mixin()
```

注意：“[透明 mixin](http://stylus-lang.com/#transparent-mixins)” 不在此列，仍以类似属性声明的方式书写。

### 参数

Mixin 的参数必须以 `$` 开头。

### 作用域

Mixin 是有作用域的。

模块内部使用的 mixin（即不是作为公开 API 存在的 mixin）应该被限定在一定的作用域内（成为局部 mixin），且建议使用 `-` 前缀，比如 `-my-temp-mixin()`。如果不得不暴露到全局作用域，则**必须**使用 `-` 前缀，并应该避免过于宽泛的命名（加上模块名作为辅助前缀是个不错的主意，比如 `-btn-style()`）。

如果某个 mixin 的作用等同于某个作为公开 API 存在的类名，则必须与该类名同名，比如 `cmBtn()`。

***

#### 相关阅读

* [Stylus 官方网站](http://stylus-lang.com/)

***

（未完待续）
