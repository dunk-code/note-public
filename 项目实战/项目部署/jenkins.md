安装部署

1. Jenkins在开发中所属的位置
2. Jenkins+Mavnen+Git持续继承基础使用
3. 安装硬件环境和知识储备

![image-20221010232100659](https://gitee.com/zhang-songyao/blog-images/raw/master/20221010232109.png)

## 安装GitLab

### SSH安装

https://gitlab.cn/install/

配置镜像

```shell
curl -fsSL https://packages.gitlab.cn/repository/raw/scripts/setup.sh | /bin/bash
```

安装

```shell
sudo EXTERNAL_URL="http://159.203.113.130" yum install -y gitlab-jh

```

gitlab常用键

```shell
sudo gitlab-ctl start                    # 启动所有 gitlab 组件；
sudo gitlab-ctl stop                    # 停止所有 gitlab 组件；
sudo gitlab-ctl restart                # 重启所有 gitlab 组件；
sudo gitlab-ctl status                 # 查看服务状态；
sudo gitlab-ctl reconfigure         # 启动服务；
sudo vim /etc/gitlab/gitlab.rb      # 修改默认的配置文件；
gitlab-rake gitlab:check SANITIZE=true --trace    # 检查gitlab；
sudo gitlab-ctl tail                        # 查看日志
```

密码

![image-20221014190037902](https://gitee.com/zhang-songyao/blog-images/raw/master/202210141900169.png)

```
/etc/gitlab/initial_root_password
e+Rrs3Y641mE0uOqqmYtqRK8RbOsd8KmDsI9EZAEugM=
```

卸载

```shell
# 停止gitlab
sudo gitlab-ctl stop
# 卸载gitlab
sudo rpm -e gitlab-ce
# 查看gitlab进程
# 杀掉第一个守护进程
ps -ef|grep gitlab
# 删除gitlab文件
find / -name *gitlab*|xargs rm -rf      
# 删除所有包含gitlab的文件及目录
find / -name gitlab |xargs rm -rf 
# 删除gitlab-ctl uninstall时自动在root下备份的配置文件（ls /root/gitlab* 看看有没有，有也删除）
```



### docker下安装

https://docs.gitlab.cn/jh/install/docker.html

安装所需最小配置：

- 内存至少4G
- 系统内核至少在3.10以上 `uname -r` 命令可以查看系统内核版本

安装docker

更新yum源

```shell
yum update
```

安装依赖

```shell
sudo yum install -y yum-utils
```

设置镜像仓库

> 官方镜像

```shell
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

## 安装jenkins

https://www.jenkins.io/

docker启动

```shell
docker run -d -p 10240:8082 -p 10241:50000 -v /home/jenkins_mount:/var/jenkins_home -v /etc/localtime:/etc/localtime --name jenkins jenkins/jenkins
```

报错

```
touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied	
```



war包下载地址

https://archives.jenkins.io/war-stable/

```shell
java -jar jenkins.war
```

![image-20221017201750893](https://gitee.com/zhang-songyao/blog-images/raw/master/202210172018484.png)

进入页面

![image-20221017202640634](https://gitee.com/zhang-songyao/blog-images/raw/master/202210172026414.png)
