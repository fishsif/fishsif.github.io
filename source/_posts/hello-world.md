---
title: windows下搭建Hexo + Github Pages
---

前几天工作不饱和，于是研究了很早之前就想搭建的hexo + github pages，发现它既有着高逼格，操作还非常简单灵活，对于我这种服务端小白来说是非常完美了。刚刚把本地的页面部署成功，有点小激动，第一篇文章就记录一下**windows**下的搭建过程（特别是遇到的一些坑）吧~

## 本地环境：安装node.js和git
因为hexo是一个基于node.js、用git进行内容发布的博客框架，所以首先需要在本地环境安装node.js和git。大部分开发人员的电脑里应该都已经安装了这两个工具，在此就不做赘述，[hexo官方文档](https://hexo.io/zh-cn/docs/index.html)也进行了详细的说明。


## github部分
首先，[GitHub Pages](https://pages.github.com/)是什么？
> GitHub Pages 是一个静态网站托管服务，被设计来管理你的来自一个GitHub 库的个人的、组织的、或者项目的页面。

我们可以在GitHub Pages上搭建个人网站，免费+稳定，用它来搭一个博客再合适不过了。

在这一步，我们需要注册一个github账号，完成注册后，建立仓库，添加SSH key将本地git项目与远程的github建立联系。

### 建立仓库
在github上新建一个与用户名对应的仓库，比如我的用户名为fishsif，仓库命名就是【fishsif.github.io】。  
<img src="http://pbuw2vh6j.bkt.clouddn.com/%E6%96%B0%E5%BB%BA%E4%BB%93%E5%BA%93.png" width="550" />

### 添加SSH key
添加SSH key的目的就是在我们以后使用时，不用每次手动输入密码，把本地的公钥拷贝到github里，以后就可以通过公钥进行登陆了。

首先设置github注册的用户名和邮箱
``` bash
$ git config --global user.name "用户名" 
$ git config --global user.email "邮箱@youremail.com" 
```
检查电脑上现有的SSH key
``` bash
$ cd ~/.ssh
```
如果不存在直接进行下面的步骤，如果存在，可以备份再删除.ssh文件夹里的所有文件。

生成新的SSH key
``` bash
$ ssh-keygen -t rsa -C "邮箱@youremail.com"
```
我是一路回车~  
SSH key设置成功后，将id_rsa.pub文件的内容输出
``` bash
$ cd ~/.ssh
$ cat id_rsa.pub
```
把cat出来的内容拷贝到github上，先打开Settings，再点进SSH and GPG keys，Title随便起：  
<img src="http://pbuw2vh6j.bkt.clouddn.com/setting" width="250" />  

<img src="http://pbuw2vh6j.bkt.clouddn.com/sshkey" width="250" />

然后测试一下，输入下面的命令，看看设置是否成功，git@github.com不要修改
``` bash
$ ssh -T git@github.com
```
如果出现 Are you sure you want to continue connecting (yes/no)? 输入yes就好。
``` bash
$ Hi fishsif! You've successfully authenticated, but GitHub does not provide shell access.
```
这样，就成功连接github了~

**说下这里遇到的坑，`ssh -T git@github.com`之前一直不成功，原因是网络问题，在公司内网或者是有防火墙之类的限制，回家再试一下，就好了。**

## Hexo
现在就可以用npm安装Hexo了~
``` bash
$ npm install -g hexo-cli
```
安装完Hexo后，一般可以新建一个网站文件夹，开始一系列配置和写作。我们换电脑时，可以选择拷贝这个文件夹（当然太不方便）或者把这个文件夹放到github上托管。但是，如果每一个GitHub Pages都需要创建一个额外的仓库来存放Hexo网站文件，还是挺麻烦的。

我参考了利用分支的方法，每个想建立GitHub Pages的仓库，起码有两个分支，一个用来存放Hexo网站的文件，一个用来发布网站。非常方便，不需要维护两个仓库，具体可以参考博文 https://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/#more  

下面是我按照该方法的具体操作：  
1. 创建仓库，fishsif.github.io；   
2. 使用git clone拷贝仓库至本地（此时只有master分支）；  
3. 新建本地分支hexo并git push推送至远端，在github上设置hexo为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件）；  
4. 在本地fishsif.github.io文件夹下，切换到hexo分支，**先将.git文件备份（重要），再清空文件夹（因为hexo init要求当前文件夹为空）**，通过Git bash依次执行hexo init、npm install 和 npm install hexo-deployer-git --save，然后将.git文件粘贴回来；  
5. 修改_config.yml中的deploy参数，branch参数应为master；  
<img src="http://pbuw2vh6j.bkt.clouddn.com/deploy%E9%85%8D%E7%BD%AE" width="400" />  
6. 依次执行git add .、git commit -m “…”、git push origin hexo提交网站相关的文件；
执行hexo g、hexo d生成网站并部署到GitHub上。

**hexo d部署过程中遇到的坑：**  
1. 和前面`ssh -T git@github.com`不成功的原因一样，关闭防火墙或者换个网络就可以了。  
2. 报错`fatal: HttpRequestException encountered.`这个只需要将_config.yml中的deploy配置中的`repo`路径由https格`https://github.com/fishsif/fishsif.github.io.git `改为ssh格式`git@github.com:fishsif/fishsif.github.io.git`即可。  
3. 如果出现报错`permission denied (publickey)`，或者`Please make sure you have the correct access rights
and the repository exists`一般是公钥设置、git配置没有成功，可以参照前面的步骤检查一遍。

好啦，现在可以在本地运行`hexo s`启动服务预览页面了，`hexo d`之后，浏览器打开 fishsif.github.io 就可以看到推送的页面咯！！其实还可以配置专属域名，但是我嫌麻烦就木有研究了。具体的hexo操作命令参考[官方文档](https://hexo.io/zh-cn/docs/index.html)，这里就不列举了~

十分不擅长写作的我竟然又写完了一篇文章。11点有世界杯总决赛，我决定不看了，good night~


