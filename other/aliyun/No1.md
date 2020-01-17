# 如何杀掉kdevtmpfsi挖矿进程


1、top   找到挖矿程序进程号

2、systemctl status 进程号         根据上面的进程号找父程序

3、关掉Active变色字体下面的进程(4-5)

4、rm -rf /tmp/kdevtmpfsi  删除掉这个东西

5、kill -9 进程号  ps -ef|grep kinsing查询进程号

6、kill -9 top里面的主挖矿进程号

————————————————

原文链接：https://blog.csdn.net/qq503758762/article/details/103714342



https://blog.csdn.net/u014589116/article/details/103705690



https://zhuanlan.zhihu.com/p/100068076



https://blog.csdn.net/yueaini10000/article/details/103680907