---
title: 设置基于Hexo的Github Pages博客
date: 2024-12-18 20:00:27 +0800
tags: [hexo]
---

**步骤：**

1. 上传代码到Github仓库。 `git push -u origin main`
2. 确保 `.gitignore` 内包含 `public/` 行。
3. 记录你的 nodejs 主要版本。 `node --version` (eg. `v20.y.z`)
4. 在Github仓库：`Settings > Pages > Source` 设置 `source` 为 `Github Actions` 并保存。
5. 创建 `.github/workflows/pages.yml` 文件，内容见附件1，注意 `node-version` 要和步骤3的一致。
6. 如果你使用了 `CANEM` ，务必在目录 `source/` 和根目录 `/` 均创建 `CNAME` 。（没有使用则无视）
7. 确保所有内容都已推送，然后访问你的博客地址。

**附件1：**

```yml
name: Pages

on:
  push:
    branches:
      - main # default branch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 20
        uses: actions/setup-node@v4
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: "20"
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**参考：**

https://hexo.io/docs/github-pages
