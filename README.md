# GitLab


本教程旨在提供搭建GitLab环境,其中涉及到一些Linux基础命令,以及Git命令,这里不做其他基础讲解,不懂得地方请自行百度！

本教程是我第一次写,写的不好的地方,还望理解,大家一起学习,共同进步！当然这个教程我也是在网上百度的,只是自己添加了一些东西,感谢提供资源的小伙伴们.



GitLab版本管理
      GitLab是利用 Ruby on Rails 一个开源的版本管理系统，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。它拥有与Github类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序(Wall)进行交流。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。
      
      
本机搭建环境介绍：


    Linux服务器：Redhat 7.2 
    GitLab：（版本待会查看）
    VMware Workstation Pro 12.5.5 build-5234757
    
配置网络yum源:

首先,GitLab在其他的yum源下载很缓慢,有些可能链接下不下来,亲测下面的地址是可以的,配置网络yum源请自行百度。

[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key


安装GitLab

yum install gitlab-ce

这里只需要等待

然后打开/etc/gitlab/gitlab.rb,将external_url = 'http://git.example.com'修改为自己的IP地址或者自己的域名。

gitlab-ctl reconfigure
这样如果没有报错就是安装完成了

直接在浏览器访问刚才修改的自己的ip或者域名，就能看到gitlab的页面了。

登录用户名跟密码,登陆后会要求你更改密码的。
Username: root Password: 5iveL!fe

接下来进行配置邮件验证（我自己配置的注册没有发送邮件,但是修改配置到时发了）


配置smtp

vim /etc/gitlab/gitlab.rb


external_url 'http://xxhost.com'
在这里添加以下信息

#Sending application email via SMTP
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"
gitlab_rails['smtp_port'] = 25 
gitlab_rails['smtp_user_name'] = "xxuser@163.com"
gitlab_rails['smtp_password'] = "xxpassword"
gitlab_rails['smtp_domain'] = "163.com"
gitlab_rails['smtp_authentication'] = :login
gitlab_rails['smtp_enable_starttls_auto'] = true

##修改gitlab配置的发信人
gitlab_rails['gitlab_email_from'] = "xxuser@163.com"
user["git_user_email"] = "xxuser@163.com"

注意163,qq什么的记得加白名单,不然可能当成垃圾邮件了!

然后再编译下
gitlab-ctl reconfigure

这里基本的搭建就ok了,其他功能只能自己再去研究了.

测试GitLab

1、使用root用户创建一个新的group，命名为devops

2、在这个组里创建一个新的project，命名为openstack,成功后会在顶部提示你添加ssh密钥，点开后提示你怎么创建.

3、创建公钥 ssh-keygen -C user@XX.com,公钥的目录在（~/.ssh/）,打开给复制到ssh里面就可以了.

4、注册一个新的用户wenbin,这里会要你输入邮箱验证,这里可以看到邮箱会收到之前设置smtp中邮件发送的邮件,激活就行(这里我没有收到邮件,开始以为我配置错了,没有生效,后面改配置收到了邮件!).然后去主机生成一个新的用户，生成ssh的key.

5、用root登陆,然后把wenbin用户添加到devops这个组中,权限为reporter.

6、然后用wenbin登陆,可以看到了openstack的project,然后添加sshkey,这样就可以pull这个项目了.

7、本地使用git克隆项目,测试能否成功push.
git clone git@gitlab.wenbin.com:devops/openstack.git

在clone之后的文件夹,创建测试文件,进行commit,最后进行push.

可以看到提示push失败,因为wenbin只是reporter!是没有权限进行push的.

当通过root用户将权限改为developer后,再次执行发现还是不能成功push文件.原因是gitlab对项目主分支做了保护,只有master的权限,我们之前新建的project的权限是developer权限,这里我们需要修改项目的权限,将其改为master+developer权限即可成功push.

修改步骤:
project->setting->protected branch


git push -u origin master


附上Git教程地址: http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/





























