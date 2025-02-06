# 个人博客说明

![Hexo Deployment Status](https://github.com/orionxer/blog/workflows/pages%20build%20and%20deployment/badge.svg)
![Website](https://img.shields.io/website?url=https%3A%2F%2Forionxer.github.io/blog)
![GitHub repo size](https://img.shields.io/github/repo-size/orionxer/blog)

## 1.特性
基于[Hexo](https://hexo.io/zh-cn/)博客框架以及[Fluid](https://hexo.fluid-dev.com/docs/)主题，结合VSCode及插件[Foam](https://foambubble.github.io/foam/)实现的个人知识博客。[在线预博客](https://orionxer.github.io/blog/)。支持如下特性：

- 知识图谱
- 双向链接
- 本地Markdown编辑快速预览
- 优雅的渲染效果
- 一键部署

![demo](demo.gif)

## 2.快速开始
> [!IMPORTANT]  
> 使用该仓库默认你已经熟悉`VSCode`并了解`Git`的相关知识以及操作。

### 2.1 VSCode插件安装列表
- Foam
- Hexo Utils 
- Markdown All in One
- Markdown Footnotes
- Markdown Preview Enhanced
- Markdown Preview Github Styling
- Prettier - Code formatter
- Flutter Color

### 2.2 部署命令
需要确保本地hexo已经安装成功
```sh
git clone https://github.com/Orionxer/blog
cd blog
rm -rf node_modules && npm install --force
hexo clean && hexo g && hexo s
```
远程部署命令，执行成功后等待一分钟，再次访问博客地址，可能需要强制刷新页面
```
hexo d
```
### 2.3 进阶技巧

#### 2.3.1 指定目录创建文章
模板`post`或`todo`均在`scaffolds`文件夹下，其余模板同理
```sh
# 普通文章
hexo new post -p todo/simple_post.md
# 以todo模板创建一个crypto的待办列表
hexo new todo "crypto" -p todo/crypto
```

#### 2.3.2 动态监控文件变动
1. 在一个终端中，执行命令`hexo g --watch`
2. 在**另一个终端**执行命令`hexo s`
3. 修改任意文章保存，浏览器刷新页面，修改即时生效

#### 2.3.3 自定义域名配置
```sh
hexo g --config gogo_config.yml
```

## 3.问题处理

### 3.1 推送失败问题
<details>
<summary>如果遇到无法推送的问题, 点击展开解决方法</summary> 

```sh
! [remote rejected] main -> main (refusing to allow an OAuth App to create or update workflow `.github/workflows/ci.yml` without `workflow` scope)
error: failed to push some refs to 'https://github.com/Orionxer/blog'
```
尝试Github Cli触发浏览器鉴权，Windows安装命令
```sh
winget install --id GitHub.cli
```

工程根目录下，执行该命令

```sh
gh auth login
```
根据提示一步步验证，直至该命令提示`Logged in as Orionxer`

```sh
? Where do you use GitHub? GitHub.com
? What is your preferred protocol for Git operations on this host? HTTPS
? Authenticate Git with your GitHub credentials? Yes
? How would you like to authenticate GitHub CLI? Login with a web browser

! First copy your one-time code: FBFE-2BE0
Press Enter to open https://github.com/login/device in your browser... 
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as Orionxer
```
</details>

### 3.2 图片路径不一致问题
```sh
# 删除hexo-asset-image和hexo-img-locator,否则hexo可能渲染失败
npm uninstall hexo-asset-image --save
npm uninstall hexo-img-locator --save
```
VSCode安装[Hexo Utils](https://marketplace.visualstudio.com/items?itemName=fantasy.vscode-hexo-utils)插件，可以解决本地图片路径与Hexo部署路径不一致的问题。快捷键`Ctrl + Alt + v`可以快速粘贴图片至同名文件夹。

### 3.3 VSCode本地预览折叠块
VSCode Markdown本地渲染折叠块：暂时没有找到可行的办法。考虑做一个VSCode插件，参考插件`Markdown Collapsible`

## 4.TODO
- [x] 通过模板创建文章, 比如`hexo new english "25-01-english"`
- [x] 优化scaffolds模板，`todo.md`和`english.md`
- [x] vscode markdown 预览gif（原生Markdown已支持）
- [x] 文章设置可见列表
- [x] 双向链接
- [x] 优化双向链接在待办页面的显示效果
- [x] 测试文章目录分类，合理分配目录
- [ ] 全部文章重新分类
- [x] 重新整理readme
- [x] readme中增加foam的说明以及演示
- [x] 优化Foam知识图谱颜色
- [ ] 试用`hexo-all-minifier`插件

## 5.仓库体积问题

如果文章中包含了较多图片，存在几个问题

- [ ] 图片在没有压缩的情况下，体积较大，占用空间且访问速度慢
- [ ] Github推荐仓库大小不要超过1GB，当前仓库容量占用 ![GitHub repo size](https://img.shields.io/github/repo-size/orionxer/blog)
- [ ] 使用外部图床，则存以下等诸多问题：
  - 易用性：是否易于管理多张图片以及生成图片链接
  - 可用性：是否被墙，或者未来被墙
  - 稳定性：服务商跑路可能性，或者泄露隐私

使用[PicX](https://picx.xpoet.cn)工具实现图片管理，其本质上的存储空间是Github仓库。好处是内置了图片压缩功能(压缩率50%左右)，且能快速优雅地管理图片以及生成图片链接。其限制是依赖于Github网络环境是否稳定，本地图片路径与云端图片路径缺少方便的批量转换的方式。需要编写一个脚本，在部署时将云端图片的路径正确替换本地的图片路径，则能够有效降低对Github Pages的占用（但Github仓库容量大小不变）


