---
layout: post
title: 初步研究nginx代码
categories: nginx
description: nginx代码的调试与复现
keywords: nginx, 源码
---

---
## 初步研究nginx代码

---

### 使用vscode调试nginx代码
我是使用MAC机器编译的，在nginx官网下载好源码之后，使用VS Code打开源码
<div align="center"><img width="350px" height="300px" src="https://seanlxh.github.io/images/posts/nginx/vscode.jpg"/></div>
打开控制台，在nginx工程根目录下
1. 修改/auto/cc/conf文件，将ngx_complie_opt 改为“-c -g”
2. 执行 sudo ./auto/configure --prefix=（nginx编译目标目录）注意地址后面不能加‘/’否则后面路径会有两个//从而找不到conf文件
3. 执行sudo make
4. 进入编译好的目标目录，执行./sbin/nginx 此时打开浏览器访问127.0.0.1 同时ps -ef | grep nginx可以看到worker与master即可。
5. 在vscode中新建一个configuration，配置好相应的地址即可开始编译。
 


---

