# Gitbook 使用教程

## 1. 本地安装Gitbook环境

1. 本地通过 NPM 安装 GitBook 命令行工具

   ```shell
   npm install gitbook-cli -g
   ```

2. Gitbook初始化

   ```shell
   gitbook init
   ```

3. 插件安装

   ```shell
   gitbook install
   ```

4. 效果预览

   ```shell
   gitbook serve
   ```

## 2. 实用插件

在根目录下新建json格式文件 book.json。配置信息如下

```json
{
    "title": "0214",
    "description": "Life is Fantastic",
    "author": "Clarencezero",
    "language": "zh-hans",
    "root": ".",

    "plugins": [
        "splitter",
        "expandable-chapters",
        "chapter-fold",
        "code",
        "emphasize",
        "prism", 
        "flexible-alerts",
        "auto-scroll-table",
        "popup",
        "-po",
        "hide-element",
        "theme-comscore",
        "github",
        "flexible-alerts",
        "klipse",
        "-lunr", 
        "-search", 
        "search-pro",
        "tbfed-pagefooter",
        "ancre-navigation"
    ],
    "variables": {
        "themeFexa":{
            "nav":[
            ]
        }
    },
    "pluginsConfig": {
        "expandable-chapters":{},
        "prism": {
            "css": [
              "prismjs/themes/prism-tomorrow.css"
            ]
        },
        "tbfed-pagefooter": {
            "copyright":"Copyright &copy Clarencezero 2020",
            "modify_label": "修订时间：",
            "modify_format": "YYYY-MM-DD HH:mm:ss"
        },
        "github": {
            "url": "https://github.com/Clarencezero"
        },
        "hide-element": {
            "elements": [".gitbook-link"]
        },
        "theme-fexa":{
            "search-placeholder":"输入关键字搜索",
            "logo":"./logo.png",
            "favicon": "./favor.png"
        },
        "simple-page-toc": {
            "maxDepth": 3,
            "skipFirstH1": true
        }
    }
}
```

### 2.1 主题插件

| 插件名称         | 功能               | 备注                                                         |
| ---------------- | ------------------ | ------------------------------------------------------------ |
| theme-fexa       | 主题插件           | 比较丰富                                                     |
| theme-comscore   | 主题插件           | 好看                                                         |
| search-pro       | 高级搜索，支持中文 |                                                              |
| tbfed-pagefooter | 页面添加页脚       |                                                              |
| flexible-alerts  |                    | `[!NOTE]`是行匹配模式，默认情况下支持类型`NOTE`，`TIP`，`WARNING`和`DANGER`。 |
| popup            | 照片弹窗           | 另开一个tab                                                  |
| ancre-navigation | toc                |                                                              |
|                  |                    |                                                              |

## 3. 代码编写规范

## 4. 发布至Github

1. 新建一个Repository。用来存放源码
2. 新建一个Repository。用来存放编译后生成的HTML文件
3. 打开网页 https://clarencezero.github.io/book2即可

