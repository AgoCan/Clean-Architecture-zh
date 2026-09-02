# 本地搭建文档

> 这是一个简单的介绍，不会很详细


创建5个文件：
- book.json:     搭建环境的配置文件
- Dockerfile：   生成镜像的文件，不重要
- README.md：    文档内容
- SUMMARY.md：   侧边栏的目录索引
- website.css：  自定义css

## 文件，必须存在的两个文件，book.json和SUMMARY.md

book.json,代码的高亮必须正确，例如不能使用 "```vue" ,不然会报错 。插件的功能，可以查看https://gitbook.zhangjikai.com/

由于honkit的使用，不能使用插件"prism", "prism-themes", 所以就使用默认的highlight

```json
{
  "title": "Clean-Architecture-zh",
  "keywords": "Keywords",
  "description": "Description",
  "author":"hankbook",
  "language": "zh-hans",
  "links" : {
    "sidebar" : {}
  },
  "styles": {
    "website": "styles/website.css"
  },

  "plugins": [
    "highlight", 
    "toggle-chapters", 
    "mermaid-fox",
    "codeblock-filename", 
    "splitter", 
    "-search",
	"-lunr", 
    "search-pro", 
    "theme-default", 
    "theme-comscore",
	"include", 
    "favicon", 
    "anchors", 
    "tbfed-pagefooter", 
    "hide-element"
  ],
    "hide-element": {
            "elements": [".gitbook-link"]
        },
    "tbfed-pagefooter": {
    "copyright":"Copyright &copy hankbook.cn 2020",
    "modify_label": "该文件修订时间：",
    "modify_format": "YYYY-MM-DD HH:mm:ss"
    },
    "search-pro": {
        "cutWordLib": "nodejieba",
        "defineWord" : ["Gitbook Use"]
    }
}
```

SUMMARY.md, 注意缩进

```
* [Clean-Architecture-zh](README.md)
    * [第 1 部分 概览](/docs/part1/README.md)
        * [第 1 章 设计与架构到底是什么](/docs/part1/ch1.md)
        * [第 2 章 两个价值维度](/docs/part1/ch2.md)
```

使用honkit搭建，如果安装honkit用-g，安装插件也必须加-g。

示例是没有-g的写法

```bash
# 配置 npm 使用阿里云镜像源
npm config set registry https://registry.npmmirror.com

npm install honkit

# 安装插件，自己去https://npmjs.org/进行查找
npm install \
        gitbook-plugin-toggle-chapters \
        gitbook-plugin-codeblock-filename \
        gitbook-plugin-splitter \
        gitbook-plugin-search-pro \
        gitbook-plugin-theme-default \
        gitbook-plugin-prism \
        gitbook-plugin-prism-themes \
        gitbook-plugin-theme-comscore \
        gitbook-plugin-include \
        gitbook-plugin-favicon \
        gitbook-plugin-anchors \
        gitbook-plugin-tbfed-pagefooter \
        gitbook-plugin-hide-element \
        gitbook-plugin-mermaid-fox
```

在本地启动

```bash
./node_modules/.bin/honkit serve
```

做成镜像,Dockerfile

```
FROM node:20-alpine AS builder

ARG https_proxy
ARG http_proxy

# 安装编译依赖（search-pro 需要 nodejieba 编译）
RUN apk add --no-cache python3 make g++

# 全局安装 honkit
RUN npm install honkit -g

WORKDIR /app
COPY . /app

# 全局安装所有插件（和 honkit 在同一个全局 node_modules 下）
RUN npm install -g \
        gitbook-plugin-toggle-chapters \
        gitbook-plugin-codeblock-filename \
        gitbook-plugin-splitter \
        gitbook-plugin-search-pro \
        gitbook-plugin-theme-default \
        gitbook-plugin-prism \
        gitbook-plugin-prism-themes \
        gitbook-plugin-theme-comscore \
        gitbook-plugin-include \
        gitbook-plugin-favicon \
        gitbook-plugin-anchors \
        gitbook-plugin-tbfed-pagefooter \
        gitbook-plugin-hide-element \
        gitbook-plugin-mermaid-fox

# 直接调用全局 honkit 构建
RUN honkit build

FROM nginx
COPY --from=builder /app/_book /usr/share/nginx/html


```

```
docker build -t book:v1 .
docker run -it -p 80:80 book:v1
```
