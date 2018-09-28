title: webpack中文文档摘要
author: Peng Fang
tags:
  - webpack
categories:
  - webpack
date: 2017-07-31 17:49:00
---
首先给出webpack中文文档的[地址](https://doc.webpack-china.org/guides/getting-started/#-)
这是我目前见过的翻译最好的webpack文档，清晰明了。

## webpack
在webpack之前，前端开发人员会使用grunt和gulp等工具来处理资源，并将它们从/src文件夹移动到/dist或/build目录中。同样的方式也被用于Javascript模块。但是像webpack这样的工具，将动态打包所有的依赖项（创建所谓的依赖图），避免打包未使用的模块。webpack最出色的功能之一就是除了JavaScript，还可以通过loader因人员任何其他类型的文件。
webpack打包命令默认选择使用一个配置文件叫webpack.config.js，改文件存在于根目录下。配置文件具有更强的灵活性，我们可以通过配置方式指定loader规则(loader rules)、插件(plugins)、解析选项(resolve options)，以及许多其他增强功能。
初始化npm (npm init -y)会生成一个package.json，其中可以配npm 脚本和项目所需的依赖等。使用npm的scripts，我们可以通过模块名，来引用本地安装的npm包，而不是写出完整路径。例如：
``` javascript
{
  ...
  "scripts": {
    "build": "webpack"
  },
  ...
}
```
我们直接调用webpack这样的别名，而不是去调用./node_modules/.bin/webpack
通过向npm run build命令和我们的参数之间添加两个中横线，可以将自定义参数传递给webpack，例如: npm run build -- --colors

## 管理输出

### 加载CSS
为了从JavaScript模块中import一个css文件，你需要在webpack.config.js的module配置中安装并添加style-loader和css-loader。webpack根据正则表达式，来确定应该查找哪些文件，并将其提供给指定的loader。这样可以在依赖此样式的文件中import某个样式文件，当模块运行时，含有CSS字符串的`<style>`标签将被插入到HTML文件的`<head>`中。
> 可以生产环境中进行[CSS分离](https://doc.webpack-china.org/plugins/extract-text-webpack-plugin)达到节省加载时间的目的。

### 加载图片
试想下载CSS的时候，如果其中有图片文件，那要如何处理呢？使用file-loader可以轻松地将内容混合到CSS中。
当我们import oneImage from './my-image.png'，该图像将被处理并添加到output目录，并且oneImage变量将包含该图片名在被处理后的最终URL。例如当使用css-loader， css中url('./my-image.png')中的路径被替换为输出目录中图片的最终路径。
> 查看[image-webpack-loader](https://github.com/tcoopman/image-webpack-loader)和[url-loader](https://github.com/tcoopman/image-webpack-loader)了解压缩和优化图像等功能。

### 加载字体
file-loader和url-loader可以接收并加载任何文件，然后将其输出到构建目录，当然也包括字体文件。

### 加载数据
数据文件一般包含JSON文件、CSV、TSV和XML。类似于NodeJS，JSON支持实际上是内置的，import一个json文件默认将正常运行。我们可以使用csv-loader和xml-loader来处理其余三类文件，csv-loader处理csv和tsv。这样import四类文件中的任何一类，所导入的变量将包含可直接使用的已解析JSON
>在使用d3等工具来实现某些数据可视化时，预加载数据会非常有用。构建过程中将数据提前载入并打包到模块中，以便浏览器加载模块后，可以立即从模块中解析数据。

### 全局资源
上述内容最出色之处是，以这种方式加载资源，可以更直观地将模块和资源组合在一起，无需依赖于含有全部资源的/asset目录，而是将资源与代码组合在一起。例如：类似这样的结构非常有用：
``` javascript
+ |– /components
+ |  |– /my-component
+ |  |  |– index.jsx
+ |  |  |– index.css
+ |  |  |– icon.svg
+ |  |  |– img.png
```
这种配置方式会使你的代码更具备可移植性，现有统一放置的方式会造成所有资源紧密耦合在一起。假如在想在另一个项目中使用/my-component，只需将其复制或移动到/components目录下。只要你安装了扩展依赖，并且配置过相同的loader，那么项目应该可以良好运行。
但是这样的缺点是无法使用新的开发方式或者在多个组件之间共享资源。仍然可以将这些资源存储在公共目录中，甚至配合使用alias来使它们更方便导入。