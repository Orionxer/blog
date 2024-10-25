# 个人博客说明

![Hexo Deployment Status](https://github.com/orionxer/blog/workflows/pages%20build%20and%20deployment/badge.svg)
![Website](https://img.shields.io/website?url=https%3A%2F%2Forionxer.github.io/blog)
![GitHub repo size](https://img.shields.io/github/repo-size/orionxer/blog)


## 推送失败问题
如果遇到无法推送的问题
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

## 本地部署命令

```sh

npm install -g hexo-cli
npm install --save hexo-theme-fluid
npm install hexo-deployer-git --save
hexo s
# 自定义域名配置
hexo g --config gogo_config.yml
```

## 图片路径不一致问题

本地图片路径与Hexo部署路径不一致的问题
- [ ] 未验证
```sh
npm install hexo-asset-image --save
npm install hexo-img-locator --save
```
确保`_config.yml`打开此项
```sh
post_asset_folder: true
```



## Pages空间限制

使用外部图床节省Pages空间
