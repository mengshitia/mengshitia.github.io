+++
title = '一些建站过程中的经历'
date = '2026-02-14T09:02:07+08:00'
description = '或许也是踩坑经历？'
draft = false
isCjkLanguage = true
categories = ['文章']
tags = ['记录']
[build]
  list = 'always'  # Use 'never' to exclude this page from all page collections.
+++

## 我对 Hugo 主题的理解

所以在我有了自制主题的想法之后，就遇到了三大终极问题中的一个：“我要做什么？”。

我在文档里能找到的唯一一句话就是：
> A theme is a module that delivers a complete set of components defining a site’s layout, presentation, and behavior. While every theme is a module, not every module is a theme. -- [Hugo Docs - Glossary](https://gohugo.io/quick-reference/glossary/#theme)。

大意是：“主题是一个模块，提供了一系列的的组件，这些组件定义了一个网站的布局、外观以及行为（或者说功能）。所有的主题都是 Hugo 模块，但反过来不一定。”

在对比了主题和站点的文件结构之后，我发现主题和站点本质上就是同一个东西。

其实按照文档创建一个站点之后不添加主题，直接构建，Hugo 会输出一些警告信息，像这样：
```text
WARN  found no layout file for "html" for kind "home": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN  found no layout file for "html" for kind "taxonomy": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
```

大意就是缺少一些模板文件。
解决方法也很简单，去 `layouts/` 文件夹下创建这些文件就行，空的也无所谓，因为 Hugo 的模板系统只需要找到这些文件并尝试处理，只要没出错就可以运行（空文件也能处理，因为没有内容）。

创建完这几个文件之后，再去翻阅 [Hugo 模板类型](https://gohugo.io/templates/types/)的相关文档，就可以理解为什么站点和主题其实是同一个东西：
因为当站点里没有对应的模板文件（和其他资源文件如样式表、脚本）时，Hugo 会去配置好的主题里寻找它的 `layouts/` 文件夹，看看里面有没有对应的模板文件，如果找到了，就正常生成网站。
而无论是把主题里的模板全部移动到站点的 `layouts/` 文件夹下，或者反过来，把站点的模板全部移动到已配置的主题中的 `layouts/` 文件夹下都是可以正常工作的。

所以，在我刚开始折腾主题的时候，其实根本没有主题（直接在站点内开工了）。
在把模板和功能做得差不多了之后才单独建了一个主题，然后把文件转移过去，再和站点整合。


## 深色浅色主题功能的原理

我决定自制主题的原因中最主要的就是实现深色浅色主题的功能：根据系统设置选择深色浅色，或让用户选择使用深色还是浅色主题。

一开始我并不会做这个功能，但是不要紧，可以先看看别人怎么做的嘛。
打开 [Paper-Mod 的示例网站](https://adityatelange.github.io/hugo-PaperMod/)，然后打开开发者工具，在点击深色浅色主题切换按钮的时候观察变化。我观察到了 [data-*](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Reference/Global_attributes/data-*) 属性：
```html
<html lang="en" dir="ltr" data-theme="dark">
<!-- 其他网页内容... --->
</html>
```

和 [color-scheme](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference/Properties/color-scheme) 属性：
```css
:root {
  color-scheme: light;
  /* 浅色主题样式... */
}

:root[data-theme="dark"] {
  color-scheme: dark;
  /* 深色主题样式... */
}
```

切换按钮很明显是有事件监听器的，这样才能完成主题的切换（下面的代码摘取自 [Paper-Mod 的示例网站](https://adityatelange.github.io/hugo-PaperMod/)）：
```javascript
document.getElementById("theme-toggle").addEventListener("click", () => {
  const html = document.querySelector("html");
  if (html.dataset.theme === "dark") {
    html.dataset.theme = 'light';
    localStorage.setItem("pref-theme", 'light');
  } else {
    html.dataset.theme = 'dark';
    localStorage.setItem("pref-theme", 'dark');
  }
})
```

那么主题切换的原理应该就是：
1. 定义好深色、浅色两套 CSS 样式表；
2. 利用自定义数据属性 `data-*`，这里是 `data-theme` 来启用对应的 CSS 样式；
3. 给切换按钮绑定事件监听器，并使用 `localStorage` 来保存用户偏好。

那么“自动”在哪里呢？

很可惜这一功能在 Paper-Mod 中并没有，以具体代码来说（下面的代码摘取自 [Paper-Mod 的示例网站](https://adityatelange.github.io/hugo-PaperMod/)）：
```javascript
if (localStorage.getItem("pref-theme") === "dark") {
  document.querySelector("html").dataset.theme = 'dark';
} else if (localStorage.getItem("pref-theme") === "light") {
 document.querySelector("html").dataset.theme = 'light';
} else if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
  document.querySelector("html").dataset.theme = 'dark';
} else {
  document.querySelector("html").dataset.theme = 'light';
}
```

存储在 `pref-theme` 中的值决定了是使用深色还是浅色。
没有时（例如初次加载网页时），才会使用 [matchMedia()](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/matchMedia) 来决定网页应该是深色还是浅色主题。

这里并不存在任何“自动”的选项，一旦用户点过主题切换，`localStorage` 中就会存储 `pref-theme`，并在下一次打开网页的时候使用。

这不是我所希望的，应该在让用户把玩主题配色功能的同时，允许用户有希望网页跟随浏览器设置的想法。

其实到这里，只需要对这部分代码进行改造即可实现我需要的功能，
但我被 [prefers-color-scheme](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference/At-rules/@media/prefers-color-scheme) 这个陌生的东西吸引了，
在简单地看了看文档之后，一拍脑袋直接用上了 [@media](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference/At-rules/@media) 规则。

当时我认为，自动选项就是去掉 `data-theme` 这个自定义数据属性，让 `@media` 发挥作用。

但这个想法完全错了。

以上面的 CSS 样式表为例。
假如我去掉 `data-theme` 属性，那么深色主题的样式表 `:root[data-theme="dark"]` 无法满足加载条件，也就不会被加载，主题永远都是浅色的。
而如果给浅色样式加上 `[data-theme="light"]`，那么任何一个主题样式都不会被加载。
那我再增加一个样式表，里面写……等下，这对吗？我这不是绕了一圈回来了？！

于是稀里糊涂地撞墙之后我终于回头了。
我对 Paper-Mod 的代码进行了一番修改，增加了 `auto`，并在获取到这个值之后根据系统设置给 `data-theme` 设置对应的值（`light` 或 `dark`）。

### 避免页面闪烁

测试完深色浅色切换功能之后，我开始琢磨配色了，打开开发者工具，直接在里面调整颜色查看效果。
改了一批颜色之后觉得不是很满意，于是点了下刷新想重新来，这一点不要紧，差点给我闪瞎。

刷新和切换页面时会快速地闪烁一次，就像有人用手电筒晃了你一下。

一开始我以为是 [transition](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference/Properties/transition) 的问题，于是增加了过渡时长。
一刷新，哇，还是爆闪！

在网络上一番搜索，了解到这个问题发生的原因：
> 如果用于确定深色浅色主题的代码在样式表应用之后才执行，那么页面加载的时候就会发生一次“闪烁”。

解决方法就是在 `<head>` 标签里面塞一段用于确定主题的脚本，
这样在页面加载的早期，样式表还未应用的时候就可以设置好该深色还是浅色主题了（下面的代码来自我的主题 Mage，这个例子中是设置了 `data-theme` 的值）：
```html
<head>
  <script>
    // 判断使用深色还是浅色主题。
    (() => {
      const savedThemeKey = 'savedTheme';
      const savedTheme = localStorage.getItem(savedThemeKey);
      if (savedTheme && savedTheme !== 'auto') {
        document.documentElement.dataset.theme = savedTheme;
      } else {
        const preferDarkMode = window.matchMedia('(prefers-color-scheme: dark)').matches === true;
        document.documentElement.dataset.theme = preferDarkMode ? 'dark' : 'light';
      }
    })()
  </script>
</head>
```


## 整合我的样式表

当我把整个页面模板拆分成单独的小部件时，为了方便理解和维护，我也将样式表文件一一对应地拆分了。

但显然如何加载这些样式表是个问题。
起初我想通过 [resources.GetMatch](https://gohugo.io/functions/resources/getmatch/) 获取样式表，
但我不想一股脑儿把这些样式表全部加载呀，而且我希望可以自行确定加载顺序，于是这个方案就被抛弃了。

之后我找到了 [@import](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference/At-rules/@import) 规则。
这看上去很符合我的需求，于是我创建了一个文件，里面只写 `@import` 规则来加载其他的样式表文件，再用 `resources.Get` 加载这个文件。

结果这样做根本不行！

[Hugo 的开发者给出了两个方案](https://discourse.gohugo.io/t/importing-css-files-from-a-css-file-not-working/47956/4)，要么像上面那样获取所有的样式表然后用 `resources.Concat` 整合，要么改用 Sass。

最后我选择改用 Sass，而这也引出了另一个问题。

### Hugo 并不会自动选择 Dart Sass

在换成 Sass 写样式表之后，自然是不熟悉，于是翻看文档，慢慢尝试。

而 Hugo 文档中警告 `resources.ToCSS` 已经在 `0.128.0` 版本过时，且内置的 LibSass 转译器也在 `0.153.0` 版本过时并且会在将来移除掉，同时要求用户使用 Dart Sass 转译器（我写的时候 Hugo 已经是 `0.155.3` 版本了）。

并且我在 [Hugo 里有关 Dart Sass 的章节](https://gohugo.io/functions/css/sass/#dart-sass)中看到了这么一段话：

> If you have been using Embedded Dart Sass with Hugo v0.113.0 and earlier, uninstall Embedded Dart Sass, then install Dart Sass. If you have installed both, Hugo will use Dart Sass. -- [Hugo Docs - Dart Sass#Installation overview](https://gohugo.io/functions/css/sass/#installation-overview)

“**If you have installed both, Hugo will use Dart Sass.**” 这句话导致我认为 Hugo 会在内置的 LibSass 和外部的 Dart Sass 转译器中优先选择使用后者。

所以我非常放心地直接用了 `toCSS`：
```go-html-template
{{ with resources.Get "scss/style.scss" | toCSS | minify | fingerprint }}
  <link rel="stylesheet" href="{{ .RelPermalink }}" integrity="{{ .Data.Integrity }}" crossorigin="anonymous">
{{ end }}
```

可是，在我逐渐应用上 Sass 的特性和语法之后，我感觉到了不对劲的地方。

在我完全[按照 Sass 的文档去使用 Maps](https://sass-lang.com/documentation/values/maps/#using-maps) 的时候，居然提示我遇到了语法错误？
一头雾水的我转到 [Sass 试验场](https://sass-lang.com/playground/)里尝试了半天也没发现哪里有问题。

接着我看到了[使用 css.Sass 的示例](https://gohugo.io/functions/css/sass/#example)中是这样的写：
```go-html-template
{{ with resources.Get "sass/main.scss" }}
  {{ $opts := dict
    "enableSourceMap" hugo.IsDevelopment
    "outputStyle" (cond hugo.IsDevelopment "expanded" "compressed")
    "targetPath" "css/main.css"
    "transpiler" "dartsass"
    "vars" site.Params.styles
    "includePaths" (slice "node_modules/bootstrap/scss")
  }}
  {{ with . | toCSS $opts }}
    {{ if hugo.IsDevelopment }}
      <link rel="stylesheet" href="{{ .RelPermalink }}">
    {{ else }}
      {{ with . | fingerprint }}
        <link rel="stylesheet" href="{{ .RelPermalink }}" integrity="{{ .Data.Integrity }}" crossorigin="anonymous">
      {{ end }}
    {{ end }}
  {{ end }}
{{ end }}
```

搞了半天，还是得配置参数啊，还得显式地给出使用 Dart Sass 转译器（`"transpiler" "dartsass"`）才可以嘛！

### 把 Sass 变量的值传递给 CSS 变量

有时候需要对不同选择器下的元素配置类似甚至相同的属性，完全复制粘贴显然不太合适，而且既然已经用上 Sass 了，为何不试试语言特性？

我的应用场景是：
只设置一个 CSS 变量，然后根据给定条件，把预先定义好的 Sass 变量值传递给 CSS 变量。
因为 CSS 变量值在改变后会立刻生效，这样就能达到动态应用样式的效果了。

起初一切顺利，直到我尝试做屏幕宽度响应，这段代码是没问题的：
```scss
@use "sass:map";

// 定义了不同宽度和对应宽度下的行为
$screenWidth: (
  "max": 1920px,
  "large": 1600px,
  "medium": 1200px,
  "small": 1024px,
);
$contentWidth: (
  "max": 1200px,
  "large": 960px,
  "medium": 768px,
  "small": 80%,
);
$contentPadding: (
  "max": 2rem,
  "large": 1.75rem,
  "medium": 1.5rem,
  "small": 10%,
);

// 最宽宽度限制
@media (min-width: map.get($screenWidth, "max")) {
  :root {
    --width-main: #{meta.inspect(map.get($contentWidth, "max"))};
    --padding-article: #{meta.inspect(map.get($contentPadding, "max"))};
  }
}
```

但是下面的代码就有问题了：
> 这段代码的结果不符合预期：
> ```scss
> /* 定义省略 */
> // 根据宽度设置对应内容宽度和文章缩进
> @each $size, $width in $screenWidth {
>   @media (max-width: $width) {
>     :root {
>       --width-main: map.get($contentWidth, $size);
>       --padding-article: map.get($contentPadding, $size);
>     }
>   }
> }
> ```
{ .block-critical }

我在 Sass 文档里翻来翻去，从找寻特定章节到漫无目的的一个章节一个章节浏览，可能过了半个多小时，就在我看得困了快睡着的时候突然找到了原因：
> Unfortunately, interpolation removes quotes from strings, which makes it difficult to use quoted strings as values for custom properties when they come from Sass variables. As a workaround, you can use the meta.inspect() function to preserve the quotes. -- [Sass: Property Declarations#Custom Properties](https://sass-lang.com/documentation/style-rules/declarations/#custom-properties)
{ .block-warning }

大意是：“插值会移除字符串的引号，因此对于含有引号的字符串的 Sass 变量是很难应用到 CSS 变量中的。一种解决方案是使用 `meta.inspect()` 函数保留引号。”。
在我这里的情况下，`$key` 作为含引号的字符串，传值进去的时候引号直接被吃掉了。
因此解决方案是：
> 使用 `meta.inspect()` 解决插值引号问题：
> ```scss
> /* 定义省略 */
> // 根据宽度设置对应内容宽度和文章缩进
> @each $size, $width in $screenWidth {
>   @media (max-width: $width) {
>     :root {
>       --width-main: #{meta.inspect(map.get($contentWidth, $size))};
>       --padding-article: #{meta.inspect(map.get($contentPadding, $size))};
>     }
>   }
> }
> ```
{ .block-sanity }

但我还要提一句，[meta.inspect() 的文档中提到](https://sass-lang.com/documentation/style-rules/declarations/#custom-properties)这个函数是用于调试的，无法保证不同版本和实现下的 Sass 中的输出结果是保持一致的。我还在寻找其他解决方法，因此目前没有将这个宽度响应的功能正式添加到主题中。


## 给网站增加多语言

“顺手的事。”，我这样想。

其实也没这么简单，因为我的菜单配置在站点，因此语言配置也在站点中。
但我同时也需要把主题做成多语言的，因此主题的配置在 `i18n/` 目录下。
这样，主题部分的 i18n 负责把模板内一些字符串替换成对应语言的文本。
而站点的 i18n 主要是站点默认语言和菜单这部分。

站点配置里的这两项比较绕：
```toml
# 此项控制设置好的默认语言所生成的文章是存放到对应语言文件夹内（true），还是直接丢到站点结构下（false）。
defaultContentLanguageInSubdir = true
# 此项控制是否禁用给默认语言添加一个跳转地址，
# 例如本站默认语言是"zh-Hans"，下面的配置是没有禁用，所以地址里会有一个"/zh-Hans"的小尾巴。
disableDefaultLanguageRedirect = false
```

吐槽：在看[这部分配置文档](https://gohugo.io/configuration/all/#disabledefaultlanguageredirect)的时候总感觉是在玩什么脑筋急转弯，为啥要用 `disable` 这么个需要反向理解的前缀啊……

总之 i18n 是配置好了，虽然这样的分离配置总感觉有点割裂什么的，至少能用，以后再看看有没有更好的方案。


## 让一些链接在点击时在新窗口打开

需要给这样的链接用上 [target](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Reference/Elements/a#target) 属性：
```html
<a href="https://gohugo.io/" title="Hugo" target="_blank">Hugo</a>
```

## 其他

~~或许还有，但是我记不清了，所以~~没有了（
