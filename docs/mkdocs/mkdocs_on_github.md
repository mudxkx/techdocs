---
title: 在Github Pages中使用Mkdocs
---

# 在Github Pages中使用Mkdocs

不在本地安装 mkdocs，而是把它放在 CI/CD 流程里（比如 GitHub Actions、Forgejo Actions、GitLab CI）。这样只要 git push，服务器就会自动拉取代码、执行 mkdocs build，并把生成的静态网站部署到目标位置（比如 GitHub Pages、服务器的 /var/www/html，或者一个专门的分支）。

## 设置步骤 
假设项目名称为：techdocs。

1. 仓库结构
 ```bash
 techdocs/
 ├── docs/  # 文档目录，所有md文档都在这个目录及子目录下
 │   ├── index.md  
 │   └── about.md
 ├── mkdocs.yml      # mkdocs 配置文件
 └── .github/
     └── workflows/
         └── deploy.yml  #GitHub Actions 工作流
 ```

2. 创建仓库

在 Github 上创建一个仓库 techdocs。

3. 本地设置
 建立一个目录techdocs，目录下的结构同上面的仓库结构。
 配置 GitHub Actions 工作流
 在仓库里新建文件： .github/workflows/deploy.yml
 ```yaml
 # .github/workflows/deploy.yml
 name: Deploy MkDocs Site
 
 # 触发条件：当推送到 main 分支时
 on:
   push:
     branches:
       - main
 
 jobs:
   deploy:
     # 运行在 GitHub 提供的虚拟机ubuntu上
     runs-on: ubuntu-latest
 
     steps:
       # 1. 检出仓库代码
       - name: Checkout code
         uses: actions/checkout@v4
 
       # 2. 设置 Python 环境
       - name: Set up Python
         uses: actions/setup-python@v5
         with:
           python-version: 3.x
 
       # 3. 安装 MkDocs 和主题
       - name: Install dependencies
         run: |
           pip install mkdocs
           # 下面按照mkdocs主题
           pip install mkdocs-material
           pip install mkdocs-awesome-pages-plugin
           # 如果需要其他插件，也在这里安装，例如：
           # pip install mkdocs-minify-plugin
 
       # 4. 构建网站（生成 site/ 目录）
       - name: Build site
         run: mkdocs build
 
       # 5. 部署到 GitHub Pages
       - name: Deploy to GitHub Pages
         uses: peaceiris/actions-gh-pages@v3
         with:
           github_token: ${{ secrets.GITHUB_TOKEN }}
           publish_dir: ./site
 ```

4. 设置mkdocs
 在 MkDocs 中，mkdocs.yml 是项目的核心配置文件，用于定义网站的结构、主题、插件等所有设置。以下是一个简单的配置：
 ```yaml
 site_name: 我的笔记   # 项目名称
 docs_dir: docs       # 指定文档目录，缺省是docs
 
 plugins:             # 使用的插件
   - search
   - awesome-pages    # 在deploy.yml中，pip install mkdocs-awesome-pages-plugin
 
 theme:               # 使用的主题
   name: material。   # 在deploy.yml中，ip install mkdocs-material
   language: zh       # 主题的语言
 ``` 




5. 写markdown文档
 docs目录下可以建立子目录，例如：linux、github等。
 首先，编写index.md，例如：
 ```markdown
 # 欢迎来到shanren的学习笔记
 这里是我的计算机技术学习笔记，使用 Markdown 编写。
 ``` 

6. 连上仓库
 ```bash
 git init
 git add .
 git commit -m "first commit"
 git branch -M main
 git remote add origin git@github.com:<username>/techdocs.git
 git push -u origin main
 ```

7. 启用 GitHub Pages   
 打开 GitHub 项目 → Settings → Pages
 Source 选择 gh-pages 分支
 保存后，网站会出现在 https://你的用户名.github.io/你的项目/
 这样，你的开发机上 完全不用安装 mkdocs，只要 push，GitHub 就会自动编译 + 部署。
