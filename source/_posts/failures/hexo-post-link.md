---
title: Hexo Post Link
---

## context

hexo post之间是有关联的，希望手动构建关系图。

但是markdown链接是会失效的。

## search plan

- [Hexo NexT troubleshooting](https://theme-next.js.org/docs/troubleshooting)
- Hexo link to another post

## stacktrace

trouble shooting没有提及这个细节问题了。

Hexo link to another post duckduckgo 第一条回复就是 [How to link a post with another post](https://github.com/hexojs/hexo/issues/2176)，点赞最多的comment就是 [post link tag plugin](https://hexo.io/docs/tag-plugins.html#Include-Posts) 了。

## solution

按照[post link tag plugin](https://hexo.io/docs/tag-plugins.html#Include-Posts) 以只需要添加 `{% post_link filenameWithoutExtension %}` 就可以了。

当然可能会有重名的问题，比如 index 这种文件（这里 [About Me](https://lslightly.github.io/about/) 是使用index的），就暂时先采用 `[About Me](/about)` 形式的链接了，因为 `about`目录位于`source`目录下。

