
---
layout: post
categories: mac 系统管理
---

>利用mac os x的launchd,开机后定时启动shell脚本，并且周期执行shell命令
shell脚本内容：先检测ssh代理是否被使用，如没有使用，则重启本地ssh代理

## shell脚本
```bash
touch $HOME/.ssh.agent.sh
chmod 755 $HOME/.ssh.agent.sh
vi $HOME/.ssh.agent.sh
```

```bash
#!/bin/bash
port=2022;
remote="-i $HOME/remote_login/pem/amazon_free_li.pem root@52.68.78.183";
function restart(){
  acitve=`netstat -an |grep $port|grep ESTABLISHED |wc -l`;
  if [ $acitve -gt 0 ]
  then
    echo "`date` :already ESTABLISHED $acitve ,no need to restart";
  else
    echo "`date` :restart ssh agent";
    ps -ef | grep $port|grep qTfnN | grep -v grep | awk '{print $2}' | xargs kill -9
    ssh -qTfnN -D $port $remote
  fi
}

function dowhile(){
   while true
   do 
    restart ;    
    sleep 600;
   done   
}

restart ;
```

## mac开机启动
### 概念
**background program** is a program that runs in the background, without presenting any significant GUI. This category is subdivided into daemons (system wide background programs) and agents (which work on behalf of a specific user). The next two sections describe these subdivisions in detail.

**A daemon** is a program that runs in the background as part of the overall system (that is, it is not tied to a particular user). A daemon cannot display any GUI; more specifically, it is not allowed to connect to the window server. A web server is the perfect example of a daemon.
>**Note:** If you're coming from a traditional UNIX background, be aware that modifying /etc/rc* is not a supported way of launching a daemon on Mac OS X. Rather, you should launch your daemon via launchd.

**An agent** is a process that runs in the background on behalf of a particular user. Agents are useful because they can do things that daemons can't, like reliably access the user's home directory or connect to the window server. 
The difference between an agent and a daemon is that an **agent** can display GUI if it wants to, while a **daemon** can't. The difference between an agent and a **regular application** is that an agent typically displays no GUI (or a very limited GUI).

**A launchd agent** is like a launchd daemon, except that it runs on behalf of a particular user. It is launched by launchd, typically as part of the process of logging in the user.

**A third party launchd agent** should be installed by adding a property list file to the ~/Library/LaunchAgents directory (to be invoked just for this user) or /Library/LaunchAgents directory (to be invoked for all users).

**A login item** is launched when the user logs in using the GUI. A login item can be any openable item, but it is typically an application or an agent

### 配置Mac开机后定时启动
根据上述概念，需要用户登录后，定时启动，所以对应/Library/LaunchAgents(所有用户)或~/Library/LaunchAgents（单独用户） 
```bash
cd ~/Library/LaunchAgents # Per-user agents provided by the administrator.
vi com.lcz.ssh-agent.plist
```

com.lcz.ssh-agent.plist配置内容如下
`说明：在plist中要写绝对路径，～或者$HOME验证不过`
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">

<plist version="1.0">
    <dict>
        <key>Label</key>    <string>com.lcz.ssh-agent</string>
        <key>ProgramArguments</key>
        <array>
               <string>$HOME/.ssh.agent.sh</string>
        </array>
        <key>StartInterval</key>  <integer>600</integer>
        <key>RunAtLoad</key> <true/>
        <key>KeepAlive</key> <false/>
        <key>StandardErrorPath</key> <string>/tmp/AlTest1.err</string>
        <key>StandardOutPath</key> <string>/tmp/AlTest1.out</string>
    </dict>
</plist>

```


```bash
launchctl load com.lcz.ssh-agent.plist #加载一个配置文件，Jobs立即启动
launchctl list |grep ssh #list all of the jobs loaded into launchd i
launchctl unload com.lcz.ssh-agent.plist
```

## 参考文档
[Daemons and Agents](https://developer.apple.com/library/mac/technotes/tn2083/_index.html#//apple_ref/doc/uid/DTS10003794) :概念科普类文档，仔细读一篇，理论会有很大提升.
[Mac crontab - Mac OS X startup jobs with crontab, er, launchd](http://alvinalexander.com/mac-os-x/mac-osx-startup-crontab-launchd-jobs):使用经验，如Label 和文件名最好要符合命名规范，以免冲突
[Creating Launch Daemons and Agents](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html):官方泛泛文档，质量一般。不如前面那篇概念科普类文档
[launchctl使用手册](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/launchctl.1.html#//apple_ref/doc/man/1/launchctl)
[launchd.plist使用手册](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man5/launchd.plist.5.html#//apple_ref/doc/man/5/launchd.plist)