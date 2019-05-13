# 使用 Pandoc 将 markdown 文档生成幻灯片

## 目标

生成一份 [Pandoc 2 Slide](http://slides.101.camp/#/title-slide) 这样的幻灯片

幻灯片能往水平和垂直两个方向运动

## 环境

- macOS

## 依赖

- Pandoc
- Reveal.js

## 安装

- 安装 Pandoc
  - brew install pandoc
  - pandoc --version
- 安装 Reveal.js
  - 从 [Releases · hakimel/reveal.js](https://github.com/hakimel/reveal.js/releases) 下载解压, 将目录名改为 `reveal.js`

## 使用

新建一个幻灯片目录 `slides`

将 `reveal.js` 目录拷贝到 `slides` 目录下

进入 `slides` 目录

新建一个 markdown 文档 `test.md`

> pandoc test.md -o test.html -t revealjs -s -V theme=moon

```
test.md 是待转换文档
-o 是输出选项, test.html 是输出文本
-t 是格式选项, 可以是s5,slidy,slideous,dzslides或者revealjs
-s 选项能够将所有依赖的文件(linked scripts, stylesheets, images, and videos)都转换输出到同一个文件中
-V theme=moon 是指定主题
    black：黑色背景，白色文本，蓝色链接（默认主题）
    white：白色背景，黑色文本，蓝色链接
    league：灰色背景，白色文本，蓝色链接（reveal.js 3.0.0 之前版本的默认主题）
    beige：米黄色背景，暗色（#333）文本，棕色链接
    sky：蓝色背景，暗色文本，蓝色链接
    night：黑色背景，亮色（#eee）文本，橙色链接
    serif：咖啡色背景，灰色文本，褐色链接
    simple：白色背景，黑色文本，蓝色链接
    solarized：奶油色背景，深绿色文本，蓝色链接

```

markdown 文档转换成幻灯片是有一套规则的

在 mardkown 文档的开头使用 `%` 来标识 标题, 作者, 日期, 如

```

% 使用 Pandoc 将 markdown 文档生成幻灯片
% damao
% 2019-05-14

```

中间不能有空行

这 3 行将转换成演示文稿的第一张幻灯片, 即封面幻灯

其余幻灯片将根据 markdown 的标题来进行转换

顶部幻灯片对应 markdown 文档中的 1 号标题 `#`, 若有多个 1 号标题, 则生成水平方向上的多张顶部幻灯片

2 号标题 `##` 及以下的内容则生成 对应 1 号标题垂直向下方向的幻灯片

如以下 markdown 文档:

```markdown
% 使用 Pandoc 将 markdown 文档生成幻灯片
% damao
% 2019-05-14

# 1

## 1.1

## 1.2

# 2

## 2.1

## 2.2

## 2.3

# 3

## 3.1
```

此文档将生成 1 个封面幻灯片, 3 个顶部幻灯片, 第 1 张顶部幻灯下有 2 张幻灯, 第 2 张顶部幻灯下有 3 张幻灯片, 第 3 张顶部幻灯片下有 1 张幻灯片

此规则是默认, 可以通过设置 `--slide-level` 选项自定义规则

## 参考链接

- [Pandoc 2 Slide](http://slides.101.camp/#/title-slide)
- [Pandoc - Installing pandoc](https://pandoc.org/installing.html)
- [技术|在命令行使用 Pandoc 进行文件转换](https://linux.cn/article-10228-1.html)
- [桌面应用|Markdown+Pandoc→HTML幻灯片速成](https://linux.cn/article-4080-1.html)
- [hakimel/reveal.js: The HTML Presentation Framework](https://github.com/hakimel/reveal.js#installation)
- [Reveal.js：把你的 Markdown 文稿变成 PPT - 少数派](https://sspai.com/post/40657)
- [reveal.js教程 - LFhacks.com](https://www.lfhacks.com/s/revealjs.html)