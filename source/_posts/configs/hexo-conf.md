---
title: hexo & console 配置记录
date: 2022-11-24 23:03:47
tags:
---
# 博客配置
如何配置博客？

# 从console开始

实习时，发现自带的console非常的漂亮。于是想美化一下自己的console。\
效果：
![](../statics/console_normal.PNG)
<p align="center">普通文件夹</p>

![](../statics/console_git.PNG)
<p align="center">git</p>

## 需要安装
- p10k  
    提供可视化的目录/状态栏  
    **vscode**适配：
    https://github.com/romkatv/powerlevel10k/issues/671  
    (`ctrl+,` 进入`设置` , 搜索 `terminal.integrated.fontFamily` 键入 `MesloLGS NF`)  

- zsh-autosuggestions  
    https://www.youtube.com/watch?v=Gj5BuFwGK6o  
    命令前有灰字提示

- zsh-syntax-highlighting  
    可行的命令被染色

# 部署hexo
 
1. 配置好服务器的反向代理 nginx  
    nginx是个反向代理，可以把不同的http request分发到不同的服务上，再加以返回。

2. 配置好服务器和客户端的git（你在服务器上给别人看部署好的网站，在客户端写博客）
```
 --------                --------
| CLIENT | --- GIT ---> | SERVER | ← visit here
 --------                -------
 ↑
edit blog here 
```
  

## 我的配置
### client
client:  
```
deploy:
  type: git
  repo: ssh://git@server:28483/home/git/blog.git
  branch: master
```

server:  
参照 

https://zhuanlan.zhihu.com/p/359394085  
https://blog.csdn.net/Y_Ace/article/details/121764537 

1. 新建user `useradd git` 设置密码`passwd git`

`cat /etc/passwd` 可以看见 `git:x:1000:1000::/home/git:/bin/bash` 时则设置完毕了

2. 给 git 添加 sudo 权限  
 chmod 740 /etc/sudoers  
 vim /etc/sudoers  
```
git ALL=(ALL) ALL
```
chmod 400 /etc/sudoers

2. 创建服务端仓库  
cd /home/git & git init --bare hexo.git  
vim /home/git/hexo.git/hooks/post-receive  
```
git --work-tree=/home/www/website --git-dir /home/git/hexo.git checkout -f
```

3. 创建网站目录  并设置权限  
mkdir /home/www/website  
chmod -R 777 /home/www  
chmod -R 777 /home/www/website  

4. 修改nginx
```
 server {
                server_name proc.moe;
                root /home/www/website;
                include /etc/nginx/default.d/*.conf;

                error_page 404 /404.html;
                location = /404.html{}
}
```

5. 部署
hexo clean & hexo g & hexo d

## 问题
### 无法更新
- 尝试查看 nginx 是否有 reload

ss -tulpn
kill -9 $(all_pids)
systemctl start nginx

- 是否真的有更新

别当傻瓜