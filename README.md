# 个人博客说明

![Build Workflow](https://github.com/orionxer/blog/actions/workflows/build.yml/badge.svg)
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

```
npm install -g hexo-cli
npm install --save hexo-theme-fluid
npm install hexo-deployer-git --save
hexo s
```

## Pages空间限制
