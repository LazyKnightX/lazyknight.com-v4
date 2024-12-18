---
title: Windows11安装StableDiffusion的问题记录
date: 2023-09-11 00:11:26 +0800
tags: [stable-diffusion]
---

## 运行 `webui-user.bat` 安装 python package 超时

如：

`Installing gfpgan`
`Installing clip`
...

1. `win+r` 输入 `%appdata%` 找到或创建目录 `pip`
2. 创建 `pip.ini`
3. 输入以下内容：

```ini
[global]
timeout = 6000
index-url = https://mirrors.aliyun.com/pypi/simple
trusted-host = mirrors.aliyun.com
```

4. 如果需要使用代理，根据情况输入类似下方的内容：

```ini
proxy = http://127.0.0.1:12340
```

5. 重新运行 webui-user.bat

**参考资料**

* [Stable-Diffusion-Webui 使用笔记(1) -- windows下安装问题汇总 | 知乎专栏](https://zhuanlan.zhihu.com/p/631381743)
* [win10 pip设置代理  | 知乎专栏](https://zhuanlan.zhihu.com/p/110945788)

## 连接 huggingface 超时

### 方案1：加载本地模型，通过修改源代码绕过链接来解决

路径： `Repo\stable-diffusion-webui\venv\Lib\site-packages\transformers`

![](./img/stable-diffusion/0x0000.png)

![](./img/stable-diffusion/0x0001.png)

**参考资料**

* [这可能是全网最好解决中国hugggingface.co无法访问问题 | 知乎专栏](https://zhuanlan.zhihu.com/p/627688602)

## No module 'xformers'. Proceeding without it.

按如下步骤执行：

1. 打开 webui 目录 (如 Repo/stable-diffusion-webui)
2. 运行 `.\venv\scripts\activate.bat`
3. `cd repositories`
4. `git clone https://github.com/facebookresearch/xformers.git`
5. `cd xformers`
6. `git submodule update --init --recursive`
7. `pip install -r requirements.txt`
8. `pip install -e .`

最后一个步骤会很消耗时间，耐心等待。

可能会在运行 `git submodule update --init --recursive` 出现 `Filename too long` 问题，可以将仓库移动到磁盘根目录，减少路径长度。

**参考资料**

* [AUTOMATIC1111](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/5303#discussioncomment-4303079)
