title: 记一次crontab执行scp异常解决
author: Peng Fang
tags:
  - 异常解决
categories:
  - 运维
date: 2018-10-22 15:57:00
---


### Python执行shell命令时的奇怪问题
通过crontab命令，周期性执行Python脚本。简单来说，用Python脚本实现的是将一些指定目录下的文件scp到远程主机的某个目录下。奇怪的是，该scp命令在命令行中执行没问题，通过上述的方式执行就不能成功，日志记录的是“scp传输失败”

### 问题具体一些
我们通过crontab命令，可以在固定的间隔时间执行指定的系统指令或shell script脚本。时间间隔的单位可以是分钟、小时、日、月、周及以上的任意组合。   
任务定义格式：   
.---------------- minute (0 - 59)   
|  .------------- hour (0 - 23)   
|  |  .---------- day of month (1 - 31)   
|  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...   
|  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat   
|  |  |  |  |   
`*  *  *  *  * user-name  command to be executed `  

例如：每五分钟执行一次验证脚本
`*/5 * * * * python /data/mgxy/scripts/verify.py MGSM ZL BLXY`

crontab有个用户域，也就是说当前用户可以设置自己的crontab命令，设置成功可以通过`crontab -l`命令查看当前用户设置的任务，不能看到其他用户设置的任务。通过`crontab -e`命令来编辑当前用户的crontab，也就是在执行命令后的编辑器中输入上面的验证脚本，然后保存后会将编辑的crontab文件提交给cron进程执行。

"verify.py"中执行scp命令是采用免密方式，免密登录是通过sshpass来实现的，用-p参数指定明文密码：
``` shell
sshpass -p loginPassord scp -oUserKnownHostsFile=/dev/null -oStrictHostKeyChecking=no /sourcePath/sourceFile username@targetIp:/targetPath/
```

每次ssh访问过的计算机公钥都会记录在`~/.ssh/known_hosts`，方便下次访问该计算机时核对。记得首次ssh登录一个主机的时候，命令行都会提示：“RSA key fingerprint is *****. Are you sure you want to continue connecting (yes/no)?”，如果回复yes,ssh客户端就会继续登录，将主机key存在文件`~/.ssh/known_hosts`中，如果回复no，连接就会中断。

scp 可以通过`-o`来指定ssh选项。`StrictHostKeyChecking=no`，该选项会禁用掉上面的交互提示，自动将主机key添加到文件`~/.ssh/known_hosts`中。如果我们确认远程主机密码更改时合法的，我们可以跳过主机密钥的校验，通过设置`UserKnownHostsFile=/dev/null`将密钥发送到一个null的known_hosts文件中。

### 一些尝试
- 将实际执行的命令，放到命令行中执行
  结果：能够正常执行。
  猜想：
  1. 会不会命令行用户和执行脚本的用户不同导致？实际这两个都同一个用户。
  2. 会不会是Python脚本的问题？python脚本在另一台机器运行正常。
- 不用crontab命令，将Python脚本在命令行执行
  结果：脚本正常执行。说明是crontab执行任务的问题。
  涛哥指导：crontab的用户比较特殊，尽可能使用绝对路径
- crontab命令中将python改为绝对路径
  结果：`*/5 * * * * /usr/bin/python xxx`问题依然存在
  设想：会不会是python脚本中执行的scp命令也要换成绝对路径
- 将python中执行的命令换成绝对路径
  结果：成功！能够正常拷贝文件。
### 解决办法
想想修改python中的命令为绝对路径，这不是通用办法。有没有通用点的办法，不用修改python脚本呢。然后我谷歌了一下，发现`/etc/contab`中可以设置环境变量，如：
``` shell
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

很多命令就在以上的$PATH中啊。想想该怎么设置环境变量呢，在哪里设置呢？他们以前会不会也有这样的问题呢？是怎么解决的呢？所以我去线上环境`crontab -l`看了一下，发现类似的设置：
``` shell
*/10 * * * * . /etc/profile;/bin/sh xxx.sh
```

然后百度了一下，有这么一段描述(来自[博客](https://www.cnblogs.com/intval/p/5763929.html))：
>当手动执行脚本OK，但是crontab死活不执行时。这时必须大胆怀疑是环境变量惹的祸，并可以尝试在crontab中直接引入环境变量解决问题。如：
>`0 * * * * . /etc/profile;/bin/sh /var/www/java/audit_no_count/bin/restart_audit.sh`

最后，我在crontab命令按照以上配置（crontab命令设置绝对路径是好习惯）：
``` shell
*/5 * * * * . /etc/profile;/usr/bin/python  xxx.py xxx
```
并在/etc/profile中设置环境变量：
`PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:$PATH`   
bingo, 成功！

#### 参考资料
https://www.cnblogs.com/chenlaichao/p/7727554.html   
https://www.shellhacks.com/disable-ssh-host-key-checking/   
https://stackoverflow.com/questions/2388087/how-to-get-cron-to-call-in-the-correct-paths   
https://www.cnblogs.com/intval/p/5763929.html   






