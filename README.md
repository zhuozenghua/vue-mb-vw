### Vue2.0 中使用vw完成移动端页面适配

如果你还对使用vw做移动端页面适配不了解，这里推荐大漠老师的两篇文章

[再聊移动端页面的适配](https://www.w3cplus.com/css/vw-for-layout.html)和[如何在Vue项目中使用vw实现移动端适配](https://www.w3cplus.com/mobile/vw-layout-in-vue.html)

随着视口单位被众多浏览器所支持，我们现在完全可以使用vw来做移动端的适配问题。下面就介绍一下Vue2.0 中使用vw实现移动端页面适配的步骤。[代码](https://github.com/zhuozenghua/vue-mb-vw)
***

#### 前提
你已经了解vue cli来构建项目。无论使用vue cli 2.x 还是3.x 版本，如果你了解webpack配置，那么过程都是大同小异。首先我们初始化一个项目:
```
vue init webpack vue-mb-vw
```
进入项目：
```
cd vue-mb-vw
```
启动项目：
```
npm run dev
```

***

#### 1. 安装一些PostCSS插件
在项目的根目录下有一个.postcssrc.js，默认情况下已经安装了以下几个插件：
*   [postcss-import](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fpostcss%2Fpostcss-import)
*   [postcss-url](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fpostcss%2Fpostcss-url)
*   [autoprefixer](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fpostcss%2Fautoprefixer)

###### postcss-import
postcss-import主要功有是解决@import引入路径问题。使用这个插件，可以让你很轻易的使用本地文件、node_modules文件

###### postcss-url
该插件主要用来处理文件，比如图片文件、字体文件等引用路径的处理。在Vue项目中，[ ` vue-loader ` ](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-loader)已具有类似的功能

##### autoprefixer
` autoprefixer `插件是用来自动处理浏览器前缀的一个插件

> 为了完成vw的布局兼容方案并简化我们的工作，还需要安装配置下面的几个PostCSS插件：

*   [postcss-px-to-viewport](https://github.com/evrone/postcss-px-to-viewport)
*   [postcss-cssnext](https://github.com/MoOx/postcss-cssnext)
*   [cssnano](https://github.com/cssnano/cssnano)
*   [postcss-aspect-ratio-mini](https://github.com/yisibl/postcss-aspect-ratio-mini)
*   [postcss-write-svg](https://github.com/jonathantneal/postcss-write-svg)
*   [postcss-viewport-units](https://github.com/springuper/postcss-viewport-units)

```
npm i  postcss-px-to-viewport postcss-cssnext cssnano   postcss-aspect-ratio-mini  postcss-write-svg  postcss-viewport-units --save-dev
```

接下来在.postcssrc.js文件对新安装的PostCSS插件进行配置：

```
module.exports = {
  "plugins": {
    "postcss-import": {},
    "postcss-url": {},
    // "autoprefixer": {},
    "postcss-aspect-ratio-mini": {}, 
    "postcss-write-svg": {
        utf8: false
    },
    "postcss-cssnext": {},
    "postcss-px-to-viewport": {
            viewportWidth: 750,     // (Number) The width of the viewport.
            unitPrecision: 3,       // (Number) The decimal numbers to allow the REM units to grow to.
            viewportUnit: 'vw',     // (String) Expected units.
            propList:['*','!font','!font-size'],
            selectorBlackList: ['.ignore', '.hairlines'],  // (Array) The selectors to ignore and leave as px.
            minPixelValue: 1,       // (Number) Set the minimum pixel value to replace.
            mediaQuery: false       // (Boolean) Allow px to be converted in media queries.
    }, 
    "postcss-viewport-units":{},
     "cssnano": {
            preset: "advanced",
            autoprefixer: false,
            "postcss-zindex": false
        }
  }
}
```

注意：由于cssnext和cssnano都具有autoprefixer。所以需要把默认的autoprefixer删除掉，然后把cssnano中的autoprefixer设置为false，我们使用cssnext的autoprefixer。

将viewportWidth设为我们视觉稿的尺寸，这里使用750px的视觉稿

##### postcss-px-to-viewport

[postcss-px-to-viewport](https://github.com/evrone/postcss-px-to-viewport)插件主要用来把`px`单位转换为`vw`、`vh`、`vmin`或者`vmax`这样的视窗单位，也是vw适配方案的核心插件之一。配置我们参考官网


##### postcss-cssnext

[postcss-cssnext](https://github.com/MoOx/postcss-cssnext)其实就是[cssnext](https://cssnext.github.io/)。该插件可以让我们使用CSS未来的特性，其会对这些特性做相关的兼容性处理。

##### cssnano
[cssnano](https://github.com/cssnano/cssnano)主要用来压缩和清理CSS代码。在Webpack中，` cssnano `和 [css-loader](https://github.com/webpack-contrib/css-loader)捆绑在一起，所以不需要自己加载它。不过你也可以使[postcss-loader](https://github.com/postcss/postcss-loader)显式的使用`cssnano`。

在`cssnano`的配置中，使用了`preset: "advanced"`，所以我们需要另外安装cssnano-preset-advanced：

```
npm i cssnano-preset-advanced --save-dev

```

`cssnano`集成了一些[其他的PostCSS插件](https://cssnano.co/guides/optimisations/)，如果你想禁用`cssnano`中的某个插件的时候，可以像下面这样操作：

```
"cssnano": {
    autoprefixer: false,
    "postcss-zindex": false
}

```
上面的代码把`autoprefixer`和`postcss-zindex`禁掉了。前者是有重复调用，后者只要启用了，`z-index`的值就会重置为`1`,我们需要禁用。

##### postcss-aspect-ratio-mini
[postcss-aspect-ratio-mini](https://github.com/yisibl/postcss-aspect-ratio-mini)主要用来处理元素容器宽高比。在实际使用的时候，具有一个默认的结构，用法可以去github查看。

##### postcss-write-svg
[postcss-write-svg](https://github.com/jonathantneal/postcss-write-svg)插件主要用来处理移动端`1px`的解决方案。该插件主要使用的是`border-image`和`background`来做`1px`的相关处理。

##### postcss-viewport-units
[postcss-viewport-units](https://github.com/springuper/postcss-viewport-units)插件主要是给CSS的属性添加`content`的属性，配合[viewport-units-buggyfill](https://github.com/rodneyrehm/viewport-units-buggyfill)库给`vw`、`vh`、`vmin`和`vmax`做适配的操作。

这是实现`vw`布局必不可少的一个插件，因为少了这个插件，这将是一件痛苦的事情。
***

#### 2. 兼容vw
让浏览器兼容视口单位的最终的解决方案就是使用[viewport-units-buggyfill](https://github.com/rodneyrehm/viewport-units-buggyfill)

[viewport-units-buggyfill](https://github.com/rodneyrehm/viewport-units-buggyfill)主要有两个JavaScript文件：viewport-units-buggyfill.js和viewport-units-buggyfill.hacks.js。

#####（1）在vue项目中的index.html引入它们：
```
   <script src="//g.alicdn.com/fdilab/lib3rd/viewport-units-buggyfill/0.6.2/??viewport-units-buggyfill.hacks.min.js,viewport-units-buggyfill.min.js"></script>
```
当然你也可以使用其他CDN，或者使用npm安装

##### （2）调用viewport-units-buggyfill：
```
<script>
    window.onload = function () {
        window.viewportUnitsBuggyfill.init({
            hacks: window.viewportUnitsBuggyfillHacks
        });
    }
</script>
```

#####（3）在使用了视口单位地方，添加content

在你的CSS中，只要使用到了`viewport`的单位（`vw`、`vh`、`vmin`或`vmax` ）地方，需要在样式中添加`content`：

```
ul {
   display: flex;
   flex-wrap: wrap;
   justify-content: space-between;
   font-size: 18px;
   list-style-type: none;
   padding:20px;
    /* hack to engage viewport-units-buggyfill */
    content: "viewport-units-buggyfill; padding: 2.667vw";
}

```
如果每次都需要手动书写，会极大地增加我们的工作量。这个时候就需要前面提到的` postcss-viewport-units` 插件。这个插件将让你无需关注`content`的内容，插件会自动帮你处理。

[viewport-units-buggyfill](https://github.com/rodneyrehm/viewport-units-buggyfill)还提供了其他的功能，详细的这里不阐述了。
***

#### 3.常见问题
##### （1）遇到不想px转换为vw的地方，可以添加指定的类名像.ignore。然后在 "postcss-px-to-viewport"插件配置中的selectorBlackList属性添加这个类名即可

##### （2）使用了ui框架的，需要避免px转vw。可以在 "postcss-px-to-viewport"插件配置中的exclude属性添加ui框架的目录，将其排除在外

##### （3）[Viewport Units Buggyfill](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Frodneyrehm%2Fviewport-units-buggyfill)添加的`content`也会引起一定的副作用。比如
>  The `content` hack may not work well on `<img>` and other replaced elements, even though it should [compute to `content: normal;` on regular elements](https://developer.mozilla.org/en-US/docs/Web/CSS/content). If you find yourself in such a situation, this may be a way out:
 ```source-css
 img {
   content: normal !important;
 }
```

>  This buggyfill only works on stylesheets! viewport units used in `style` attributes are *not* resolved.

>  The buggyfill can easily trip over files host on different origins (requiring CORS) and relative URLs to images/fonts/… within stylesheets. [#11](https://github.com/rodneyrehm/viewport-units-buggyfill/issues/11)