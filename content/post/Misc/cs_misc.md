---
title: "技术杂项"
date: 2021-11-01T15:20:01+08:00
categories:
- 计算机科学与技术
- 杂项
tags:
- 杂项
- 滚动更新
---
一些尚未形成体系，但是可能很有用的技术杂项整理。
<!--more-->
# Web
1. PixiJS：纹理重复加载问题
    - 问题描述：使用Pixi在加载纹理的时候，经常出现重复加载的警告。显然可以优化。
    - 基础写法
        ```javascript
        textureList = ['a.jpg','b.jpg'];
        loader.add(textureList);
        loader.load((loaders, resources) => {
            //balabala
        });
        ```
    - 问题原因：加载的时候，会将纹理加载进缓存，警告也是提示这一点，缓存中已经有重名的纹理了。
    - 修改写法：使用默认的loader，并使用PIXI.utils获取缓存，加载前进行去重判断
        ```javascript
        app.loader = PIXI.Loader.shared;
        for (let texture of textureSet) { // textureSet是存储纹理加载路径的集合
            if (PIXI.utils.BaseTextureCache[texture]) {
                textureSet.delete(texture);
            }
        }
        let textureList = Array.from(iconSet);
        if (textureList.length != 0) {
            app.loader.add(textureList).loader((loader, resources) => {
                // balabala
            });
        }
        ```
    - 参考:[精灵加载去缓存](https://segmentfault.com/a/1190000022280843)