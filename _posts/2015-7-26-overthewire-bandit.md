---
layout: post
title: OverTheWire Bandit
tags: ctf
path: /assets/2015-7-26-overthewire-bandit
---
��Ŀ��ַ: [OverTheWire: Bandit](http://overthewire.org/wargames/bandit/).

���Ķ���Щ Linux �µĻ�������, ��Ȼ˵�ǻ�������, ������Щ���û�ù�, ����������Щ��Ҳ�������ǳ. ���л����м����������˼...~~����˼����˼���Ҳ�����~~.

# WeChall �Ʒְ�
OverTheWire ʹ�� Wechall �����Ʒ�, ����μ� [WeChall Scoreborad](http://overthewire.org/about/wechall.html)

��Ϊ��Ŀ����Զ�̵� ssh ������, ��������Ҫ��Զ������֪������˭.
����� `.bashrc` �����������������:

```bash
export WECHALLUSER="���� WeChall ���û���"
export WECHALLTOKEN="���� WeChall �� WarToken"
```

WarToken ������ WeChall ��վ�� [Account -> WarBoxes -> Your current WarToken](http://www.wechall.net/warboxes) ���.

�� `~/.ssh/config` �����:

```
Host *.labs.overthewire.org
SendEnv WECHALLTOKEN
SendEnv WECHALLUSER
```

�� ssh ���ӵ�ʱ��ͻ������ʺ���Ϣ���͸�Զ������.

�ڵ���һ���µĹؿ���, ִ�� `wechall` ��ɻ�øùؿ��ķ���.

#��Ŀ
##bandit17

> The password for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL and which don��t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

Ҫ�� 31000-32000 ֮��Ķ˿����ҳ��п��ŵĲ���ֻ�� SSL ���ӵĶ˿�, ���ö˿ڷ�����һ�ص�����ͻ᷵����һ�ص�����.

�ҳ�������Ӧ��Ķ˿�:

```bash
nc -v -w 2 localhost 31000-32000 2>/tmp.tmpxxx/log
cat /tmp.tmpxxx/log | grep Succ
```

`-v` �����������ӵ���ϸ��Ϣ, `-w 2` ָ�� timeout, ��֪��Ϊʲô `-v` ����Ϣ��ֱ������� `stderr` ��. `tmp/tmp.xxx` ʹ�� `mktemp -d` ��������ʱĿ¼.

�����Ӧ���ֻ������˿�, ����������.

```bash
openssl s_client -connect localhost:31xxx -ssl3 -quiet
```

������ 31790 �˿�, ����һ�� RSA ˽Կ, ����:

```bash
openssl s_client -connect localhost:31790 -ssl3 -quiet > bandit17.private
chmod 0600 bandit17.private
ssh -i bandit17.key bandit17@bandit.labs.overthewire.org
cat /etc/bandit_pass/bandit17
```

> key:xLYVMN9WE5zQ5vHacb0sZEVqbrp7nBTn


##bandit21

> There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).
>
> **NOTE:** To beat this level, you need to login twice: once to run the setuid command, and once to start a network daemon to which the setuid will connect.<br>
> **NOTE 2:** Try connecting to your own network daemon to see if it works as you think

`bandit20` �ļ�Ŀ¼���ṩ��һ������ `suconnect`, �����ָ���Ķ˿ڶ�ȡ `bandit20` ������, �����ȷ�Ļ����ر��ؿ�������.

> echo GbKksEFF4yrVs6il55v6gwY5aVje5f0j | nc -l -p 1234 & ./suconnect 1234

������Ҫ�� `&` ���÷�, ʹ��������ͬʱִ��.

#bandit24
> A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.
>
> **NOTE:** This level requires you to create your own first shell-script. This is a very big step and you should be proud of yourself when you beat this level! <br>
>**NOTE 2:** Keep in mind that your shell script is removed once executed, so you may want to keep a copy around�� 

�����Ҿ����е���˼.

cron ��һ����ʱִ�й���, �������ͨ������ `crontab` �趨, ���ô����� `/etc/cron.d` ��, ÿ���� cron �ᱻ����һ��, ����Ŀ¼����Ƿ�������Ҫִ��.

/etc/cron.d/cronjob_bandit24:

```bash
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
```

����˵ `/usr/bin/cronjob_bandit24.sh` ��ÿ����ִ��һ��, ��������ű���������ʲô:

```bash
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname
echo "Executing and deleting all scripts in /var/spool/$myname:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
        echo "Handling $i"
        timeout -s 9 60 "./$i"
        rm -f "./$i"
    fi
done
```

ÿ�ζ�ִ�� `/var/spool/bandit24` �µ����п�ִ���ļ�, ֮��ɾ��. ��Ȼ���� `bandit24` �����ִ����Щ����.

�������ǿ��Թ���һ���ű�����ִ��.

 ```bash
#!/bin/sh
cp /etc/bandit_pass/bandit24 /tmp/tmp.xxx/psw
chmod 666 /tmp/tmp.xxx/
```

����ű��������ļ����Ƶ���ʱĿ¼���Ҹ�������Ȩ��(�����������˿ɶ�).

�м���˺ܶ��޴��Ĵ���, ����д��Ŀ¼,��� `sh` ��·��ʲô��, ����, �������ض��򵼳� `bandit24` ������, ��Ϊû��Ȩ��(Ϊʲôû��Ȩ���ҾͲ������). 

�ű�д���, `chmod +x`, �ٰ������Ƶ� `/var/spool/bandit24` Ŀ¼��, �ǵñ���, ÿ��һ���Ӹ�Ŀ¼�ͻᱻ���.

�Ƚ�����˼����, �����Ǹ�Ŀ¼�¶�ˢ�¼���, ��ʱ���Կ�������д�Ľű�.


#bandit25
> A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinaties, called brute-forcing.

��һ���ػ������� `30002` �˿ڼ���, �� `bandit24` �������һ����λ������ɵ� pincode ������, �������� pincode ����ȷ�Ļ��᷵�� `bandit25` ������.

������Ȼ�Ǳ�����, ֱ���� nc ���Ӹö˿ڻ���ʾ:

> I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.

���԰��ո�ʽ��, ���� 10000 �� ���� + pincode �����д����ö˿�.

```bash
for i in {0000..9999}; do echo "UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ $i" >> /tmp/pin; done
cat /tmp/pin | nc localhost 30002 > /tmp/log
cat /tmp/log | grep "Corr" -n1
```

��ʵ�𰸾������һ���˿�...

> 5670-Wrong! Please enter the correct pincode. Try again.
> 5671:Correct!
> 5672-The password of user bandit25 is uNG9O58gUE7snukf3bvZ0rxhtnjzSGzG

#bandit26
> Logging in to bandit26 from bandit25 should be fairly easy�� The shell for user bandit26 is not /bin/bash, but something else. Find out what it is, how it works and how to break out of it.

���� 26 �� level ����������˼��һ�����, ��ϧ��û��������...  �ο��Ĵ������: [overthewire-bandit-level-26](http://codebluedev.blogspot.com/2015/07/overthewire-bandit-level-26.html)

�����˵ `bandit26` �� shell ��������ͨ�� `/bin/bash`.

`bandit25` �ļ�Ŀ¼�¸����� `bandit26` ��˽Կ, ��¼��ȥֻ�Ǵ�ӡ���� bandit26 �� ASCII Art ���˳���.

```
  _                     _ _ _   ___   __  
 | |                   | (_) | |__ \ / /  
 | |__   __ _ _ __   __| |_| |_   ) / /_  
 | '_ \ / _` | '_ \ / _` | | __| / / '_ \ 
 | |_) | (_| | | | | (_| | | |_ / /| (_) |
 |_.__/ \__,_|_| |_|\__,_|_|\__|____\___/ 
```

ִ�� `cat /etc/passwd | grep bandit26` �õ�:

> **bandit26**:x:11026:11026:bandit level 26:/home/**bandit26**:/usr/bin/showtext

(�ҵ�����Ϳ�ס��)
���� `showtext` ��һ�� shell �ű�, ��������

```bash
#!/bin/sh

more ~/text.txt
exit 0
```

�� ssh ����ȥ��ִ��Ĭ�� shell, �� `more` ��ӡ���� ~/text.txt ֮����˳���, ��ͼ:
![1]({{ page.path }}/1.png)

һ���Ͼ��˳�, ��������ô����ִ��������Ҫ��������? ֱ���� ssh �� `-t` + ���� �ǲ��е�, �������ᱻ����, ��Ϊ `bash` û��ִ��.

��ȷ����ͨ�� `more`.

`more` ��Ҫ������������������ն�������ʱ���ͣ����, �ȴ��㷭ҳ, �������ǰѵ�ǰ���ն˵�С, �������, �ٴ� shh ��ȥ, `more` ��ͣ������. (���Զ�)
![2]({{ page.path }}/2.png)

�� `more` ���水 v, ϵͳ�����Ĭ�ϵı༭�����༭����ļ�, Ĭ���� `vi`, ���� `vi`, ���൱������һ���ն�.

������ģʽִ�� `:r /etc/bandit_pass/bandit26`, ��������ݾͻᱻ����.
![3]({{ page.path }}/3.png)

Ҫ�� `sh` �Ļ�, ����:

```vimL
:set shell sh=/bin/sh
:sh
```

�����Ϳ���ִ�� `wechall` �÷���.
