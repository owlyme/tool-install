## Linux-GitLab安装及汉化
Gitlab相关操作及说明：
  ```
  /etc/gitlab/gitlab.rb          #gitlab配置文件
  /opt/gitlab                    #gitlab的程序安装目录
  /var/opt/gitlab                #gitlab目录数据目录
  /var/opt/gitlab/git-data       #存放仓库数据
  gitlab-ctl reconfigure         #重新加载配置
  gitlab-ctl status              #查看当前gitlab所有服务运行状态
  gitlab-ctl stop                #停止gitlab服务
  gitlab-ctl stop nginx          #单独停止某个服务
  gitlab-ctl tail                #查看所有服务的日志

  Gitlab的服务构成：
  nginx：                 静态web服务器
  gitlab-workhorse        轻量级反向代理服务器
  logrotate              日志文件管理工具
  postgresql             数据库
  redis                  缓存数据库
  sidekiq                用于在后台执行队列任务（异步执行）
  ```

## 安装环境：
1. CentOS 6或者7    （此处使用7）
2. 2G内存（实验）生产（至少4G），不然会很卡
3. 安装包：gitlab-ce-10.2.2-ce
4. 禁用防火墙，关闭selinux

## 安装gitlab
1. 安装依赖
```
[root@gitlab ~]# yum install -y curl policycoreutils-python openssh-server        
[root@gitlab ~]# wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-10.2.2-ce.0.el7.x86_64.rpm        #下载软件包
[root@gitlab ~]# rpm -ivh gitlab-ce-10.2.2-ce.0.el7.x86_64.rpm    #安装gitlab
```
2. 根据安装完成提示界面进行访问URL更改及重新加载配置文件 更改次选项为自己的域名或者IP external_url 'http://gitlab.example.com'
```
[root@gitlab ~]# vim /etc/gitlab/gitlab.rb      #编辑配置文件  
external_url 'http://192.168.1.21'        #改为自己的IP地址
[root@gitlab ~]# gitlab-ctl reconfigure    #重新加载配置文件
```
3. 重装完成访问http://192.168.1.21，会首先叫更改密码（root用户），改完后登录。

4. 汉化
  - 下载汉化补丁
[root@gitlab ~]# git clone https://gitlab.com/xhang/gitlab.git
[root@gitlab ~]# cd gitlab    
  - 查看全部分支版本
[root@gitlab ~]# git branch -a
  - 对比版本、生成补丁包
[root@gitlab ~]# git diff remotes/origin/10-2-stable remotes/origin/10-2-stable-zh > /tmp/10.2.2-zh.diff
  - 停止服务器
[root@gitlab ~]# gitlab-ctl stop
  - 打补丁
[root@gitlab ~]# patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < /tmp/10.2.2-zh.diff
  - 启动和重新配置
[root@gitlab ~]# gitlab-ctl start
[root@gitlab ~]# gitlab-ctl reconfigure
  - 说明：这里如果使用的同样是gitlab10.2.2，下载汉化较慢的话，可以直接在这里下载10.2.2-zh.diff

5. 汉化完成页面如下

![ok.png](https://github.com/owlyme/tool-install/blob/main/gitlab/ok.png)

## 疑难问题
1. 问题一
    - 在CentOS 7搭建GitLab服务器的过程中，一开始是报Whoops, GitLab is taking too much time to respond 502 错误错误，改了/etc/gitlab/gitlab.rb文件的如下配置，本来默认配置是8080，可能已经被其他应用占用了，我就改成和EXTERNAL_URL配置一样的端口号，改成了8090，结果变成报如下错误了；
    CentOS 7搭建GitLab服务器踩坑——解决nginx 400 Bad Request Request Header Or Cookie Too Large问题

      ```
      EXTERNAL_URL="http://11.86.9.67:8090"
      unicorn['port'] = 8090
      gitlab_workhorse['auth_backend'] = "http://localhost:8090" 
      ```


    - 解决方法：其实解决方法很简单，*unicorn和gitlab_workhorse的端口号不要和EXTERNAL_URL设置一样的就行了*，开一个新的端口，记得在防火墙放开，比如我开了8091端口
      1. 首先vim  /etc/gitlab/gitlab.rb打开配置文件
      2. 修改配置
        ```
        EXTERNAL_URL="http://11.86.9.67:8090"
        unicorn['port'] = 8091
        gitlab_workhorse['auth_backend'] = "http://localhost:8091" 
        ```
      3. 输入如下命令让配置生效
      ```
        sudo gitlab-ctl reconfigure
        ```
      4. 最后重启服务, 因为重启服务后刷新可能不能马上成功，差不多要等个一分钟左右再重新刷新页面就成功了
      ```
      sudo gitlab-ctl restart
      ```
