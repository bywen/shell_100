在服务器上，写一个监控脚本，要求如下：

每隔10s去检测一次服务器上的httpd进程数，如果大于等于500的时候，就需要自动重启一下apache服务，并检测启动是否成功？

若没有正常启动还需再一次启动，最大不成功数超过5次则需要立即发邮件通知管理员，并且以后不需要再检测！

如果启动成功后，1分钟后再次检测httpd进程数，若正常则重复之前操作（每隔10s检测一次），若还是大于等于500，那放弃重启并需要发邮件给管理员，然后自动退出该脚本。假设其中发邮件脚本为之前使用的mail.py

核心要点

pgrep -l httpd或者ps -C httpd --no-heading检查进程
for循环5次计数器
参考答案
```
#!/bin/bash
check_service()
{
    n=0
    for i in `seq 1 5`
    do
        /usr/local/apache2/bin/apachectl restart 2>/tmp/apache.err
        if [ $? -ne 0 ]
        then
            n=$[$n+1]
        else
            break
        fi
    done
    if [ $n -eq 5 ]
    then
        ##下面的mail.py参考https://coding.net/u/aminglinux/p/aminglinux-book/git/blob/master/D22Z/mail.py
        python mai.py "123@qq.com" "httpd service down" `cat /tmp/apache.err`
        exit
    fi
}   
while true
do
    t_n=`ps -C httpd --no-heading |wc -l`
    if [ $t_n -ge 500 ]
    then
        /usr/local/apache2/bin/apachectl restart
        if [ $? -ne 0 ]
        then
            check_service
        fi
        sleep 60
        t_n=`ps -C httpd --no-heading |wc -l`
        if [ $t_n -ge 500 ]
        then
            python mai.py "123@qq.com" "httpd service somth wrong" "the httpd process is busy."
            exit
        fi
    fi
    sleep 10
done
```

