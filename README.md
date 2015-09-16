#bash+rsyslog记录操作命令
##From:
###http://evilabandon.tumblr.com/page/19
###http://crazyof.me/blog/archives/1891.html

#1、修改源码
````
vim config-top.h
#打开注释
#define SYSLOG_HISTORY
#define SSH_SOURCE_BASHRC

vim bashhist.c

bash_syslog_history (line)
     const char *line;
{
  char trunc[SYSLOG_MAXLEN];

  if (strlen(line) < SYSLOG_MAXLEN)
    syslog (SYSLOG_FACILITY|SYSLOG_LEVEL, "HISTORY: PID=%d PPID=%d SID=%d User=%s CMD=%s", getpid(), getppid(), getsid(getpid()), current_user.user_name,  line);
  else
    {
      strncpy (trunc, line, SYSLOG_MAXLEN);
      trunc[SYSLOG_MAXLEN - 1] = ' ';
      
      syslog (SYSLOG_FACILITY|SYSLOG_LEVEL, "HISTORY (TRUNCATED): PID=%d PPID=%d SID=%d User=%s  CMD=%s", getpid(), getppid(), getsid(getpid()), current_user.user_name, trunc);
    }
}

vim /etc/profile

test -z "$BASH_EXECUTION_STRING" ||  logger -t -bash -s "HISTORY $SSH_CLIENT  CMD=$BASH_EXECUTION_STRING" >/dev/null 2>&1;

````

#2、配置rsyslog
````
echo 'SYSLOGD_OPTIONS="-m -r 0"' >> /etc/rsyslog.conf
/etc/init.d/rsyslog restart
````

#3、查看效果
````
[root@centos log]# tail messages
Sep 15 05:33:57 centos -bash: HISTORY: PID=2275 PPID=2273 SID=2275 User=root CMD=vim /etc/bashrc
Sep 15 05:35:53 centos -bash: HISTORY: PID=2275 PPID=2273 SID=2275 User=root CMD=vim /etc/rsyslog.conf
Sep 15 05:37:41 centos -bash: HISTORY: PID=2275 PPID=2273 SID=2275 User=root CMD=ls
Sep 15 05:37:43 centos -bash: HISTORY: PID=2275 PPID=2273 SID=2275 User=root CMD=cd /var/lo
Sep 15 05:37:45 centos -bash: HISTORY: PID=2275 PPID=2273 SID=2275 User=root CMD=cd /var/log/
Sep 15 05:37:46 centos -bash: HISTORY: PID=2275 PPID=2273 SID=2275 User=root CMD=ls
Sep 15 05:37:49 centos -bash: HISTORY: PID=2275 PPID=2273 SID=2275 User=root CMD=cat secure
Sep 15 05:37:50 centos -bash: HISTORY: PID=2275 PPID=2273 SID=2275 User=root CMD=cat messages
Sep 15 05:37:57 centos -bash: HISTORY: PID=2275 PPID=2273 SID=2275 User=root CMD=clear
Sep 15 05:38:01 centos -bash: HISTORY: PID=2275 PPID=2273 SID=2275 User=root CMD=tail messages
````
