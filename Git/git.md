
## 本地项目向github上传

### 禁用换行符转换
```
git config --global core.autocrlf false
git config --global core.filemode false
git config --global core.safecrlf true
```

### 操作
https://blog.csdn.net/weixin_42312323/article/details/107407674

```
首先在github上创建仓库。
git init //初始化本地文件为仓库
git remote add <简称> <url> //设置远程仓库别名，通常是origin
git pull
git branch --set-upstream-to=<简称>/<分支名> //用当前本地分支追踪远程分支。上面的简称，分支名在pull后的无追踪分支提示信息上可以看到，选择想要的分支即可
git pull --allow-unrelated-histories //如果远程仓库有内容，就要允许不想关的历史合并
// 如果冲突，那就解决冲突后重新提交再pull
git push -u <简称> <分支名> //推送即可
```

### 问题：要输入用户名和密码
`git remote -v` 观察远程分支的url，如果是https打头那就要输入用户名
`ssh -T git@github.com` 测试能否访问
`ssh -Tvvv git@github.com` debug访问，大量信息。

首先，为了避免每次推送和拉取都要输入用户名，我们选择ssh的仓库地址
`git@github.com:Ninemerlin/MyWebServer_KIGOLF.git`

然后，在github的账户设置（而非仓库设置）中，添加本机的公钥（id_rsa.pub)。
![[Pasted image 20240403001345.png]]
这个公钥无需必须使用命令`ssh-keygen -t rsa -C "你的邮箱"`生成：
邮箱不是必要的，不必和github的相同。rsa也不是必要的，甚至有记录称rsa 1因过时而不能认证。git中的邮箱信息，用户名信息也不是必须对应，它只影响提交信息——认证只需要用上传的公钥，匹配本地的配对私钥即可。

理论上，使用ssh的仓库地址，上传公钥，即可无认证push和pull。

但是，计算机这些东西，不出问题才怪了。
如果你遇到22端口被拒绝`connect to address 127.0.0.1 port 22: Connection refused`，那么可能并不是公钥上传错了或者哪里疏忽了。

检查~/.ssh/config，添加
```
Host github.com
	HostName ssh.github.com
	Port 443
	User git
	IdentityFile ~/.ssh/id_rsa
# PubkeyAcceptedKeyTypes +ssh-rsa
```

上面这部分，大概率是没有的。先不要直接复制，看一下各行：
Host和HostName：Host为HostName的缩写，HostName为真实ssh链接到的域名。Host的存在允许ssh Host来替代ssh HostName
```
$ ssh git@github.com # 等同于 ssh ssh.github.com
```
Port：ssh的访问端口
User：访问的用户名
IdentityFile：指定私钥
PubKeyAcceptedKeyTypes：可用的协议

### 坑
你会发现一些文章告诉你补充config文件为如下样式
```
Host github.com
	HostName github.com
	Port 22
```
- [ ] 该部分可能存在不准确的描述
首先，你很可能不知道，除了22端口，ssh还能走443端口，甚至992，2020，8080。443走https流量，而https一般不会被拦截（22会，严加看管）。

问题在于，ssh连接的github.com通过22端口会被拒绝，而这正是我们默认什么也没配置的情况：大概率就是不知道在哪被拦了。
`ssh: connect to host github.com port 22: Connection refused`

直到有文章访问了`ssh.github.com`，你试了试，才知道访问这个会有一些变化，才知道能走443端口：
```
The authenticity of host 'ssh.github.com (20.205.243.160)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6Tu....
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:13: [ssh.github.com]:443
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ssh.github.com' (ED25519) to the list of known hosts.
Hi Ninemerlin! You've successfully authenticated, but GitHub does not provide shell access.
```
突然间就能访问了。
ssh.github.com大概是github专门设置的服务器。这个HostName允许443访问，同时，还能提醒你不要用22端口，试试443.

那在github.com上用443端口呢？结果是设置为443端口，会因为不开放而被本地ssh代理关闭。
```
kex_exchange_identification: Connection closed by remote host
Connection closed by 127.0.0.1 port 443
```

于是，为config文件添加如下部分即可
```
Host github.com
	HostName ssh.github.com
	Port 443
```

### 经验：IdentityFile
该语句来源于某篇文章，作者表示不能访问是配对的私钥手动生成到了别的地方，而默认就是`~/.ssh/id_rsa`
所以通过`IdentityFile ~/.ssh/id_rsa`来限定使用的私钥

### 经验：PubkeyAcceptedKeyTypes +ssh-rsa
作者表示，sha-1因为过时而在新版本git中被拒绝使用。导致以往的项目因此无法认证。
使用该语句重新强制启用。

### 额外：代理机制及其缓存
- [ ] 有关ssh-agent，没有什么详细的信息。以下内容为推导。
作者表示，ssh-agent是ssh的一个本地代理，有自己的缓存。他连不上是因为配置改了，但是缓存没更新。
手动更新：
	前置命令：`eval 'ssh-agent -s`和`ssh-agent bash`
	主要命令：`ssh-add "c:/Users/<用户名>/.ssh/id_rsa`
