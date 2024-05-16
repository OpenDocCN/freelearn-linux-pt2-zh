评估

# 第二十三章：知识检查 1

1.  `cal 2023`

1.  `免费-h`

1.  `ls /home/elliot`

1.  `passwd`

1.  echo "Mr.Robot is an awesome TV show!"

## 真或假

1.  假

1.  假

1.  真

1.  假

1.  真

# 知识检查 2

1.  `ls -l /var/log`

1.  `cat /etc/hostname`

1.  `touch file1 file2 file3`

1.  `ls -a /home/elliot`

1.  `mkdir /home/elliot/fsociety`

## 真或假

1.  假

1.  假

1.  真

1.  假

1.  真

# 知识检查 3

1.  `头-n 2 facts.txt`

1.  `tail -n 1 facts.txt`

1.  `tac facts.txt`

1.  `vi facts.txt`

1.  `:q`

# 知识检查 4

1.  `touch hacker1 hacker2 hacker3`

1.  `mkdir Linux Windows Mac`

1.  `touch Linux/cool`

1.  `touch Windows/boring`

1.  `touch Mac/expensive`

1.  `cp hacker1 hacker2 /tmp`

1.  `cp -r Windows Mac/tmp`

1.  `mv hacker3 /tmp`

1.  `mv Linux /tmp`

1.  `rm Mac/expensive`

1.  `rmdir Mac`

1.  `rm -r Windows`

1.  `rm hacker2`

1.  `mv hacker1 hacker01`

## 真或假

1.  假

1.  假

1.  真

1.  假

1.  真

# 知识检查 5

1.  `类型 echo`

1.  `which uptime`

1.  `whatis mkdir`

1.  `man mv`

1.  `apropos calendar`

1.  `帮助历史`

## 真或假

1.  假

1.  假

1.  真

1.  真

# 知识检查 6

1.  `ls -id /var/log`

1.  `stat /boot`

1.  `mkdir coins`

1.  `ln -s coins currency`

1.  `touch coins/silver coins/gold`

1.  `touch currency/bronze`

1.  `ls coins currency`

1.  `ln 饮料 饮料`

1.  `rm 饮料`

1.  `cat drinks`

## 真或假

1.  假

1.  真

1.  真

1.  假

1.  真

1.  假

1.  真

# 知识检查 7

1.  `su root`

1.  `passwd root`

1.  `su - elliot`

1.  `su`

## 真或假

1.  真

1.  真

1.  假

# 知识检查 8

1.  `useradd -u 333 abraham`

1.  `groupadd admins`

1.  `usermod -aG admins abraham`

1.  `chgrp admins /home/abraham`

1.  `chmod g=r /home/abraham`

## 真或假

1.  真

1.  假

1.  假

# 知识检查 9

1.  `头-n 5 facts.txt *|* 尾-n 1`

1.  `free > system.txt`

1.  `lscpu >> system.txt`

1.  `rmdir /var 2> error.txt`

# 知识检查 10

1.  `du -b /etc/hostname`

1.  `cut -d: -f1 /etc/group`

1.  `wc -l /etc/services`

1.  `grep bash /etc/passwd`

1.  `uptime *|* tr [:lower:] [:upper:]`

# 知识检查 11

1.  `locate boot.log`

1.  `find / -size +50M`

1.  `find / -size +70M -size -100M`

1.  `find / -user smurf`

1.  `find / -group developers`

# 知识检查 12

1.  `apt-get install tmux`

1.  `apt-cache depends vim`

1.  `apt-get install cowsay`

1.  `apt-get purge cowsay`

1.  `apt-get update 然后运行 apt-get upgrade`

# 知识检查 13

1.  `pgrep terminal`

1.  `ps -fp pid_of_terminal`

1.  `杀-9 pid_of_terminal`

1.  `firefox &`

1.  `renice -n -20 pid_of_firefox`

# 知识检查 14

1.  `smurf ALL=(ALL) /sbin/fdisk`

1.  `%developers ALL=(ALL) /usr/bin/apt-get`

1.  `sudo-lU smurf`

# 知识检查 15

1.  `hostnamectl set-hostname darkarmy`

1.  `netstat -rn 或 ip 路由`

1.  `traceroute www.ubuntu.com`

1.  `cat /etc/resolv.conf`

1.  `nslookup www.distrowatch.com`

1.  `ifconfig eth0 down`

1.  `ifconfig eth0 up`

# 知识检查 16

1.  ```
    #!/bin/bash
     cal
    ```

1.  ```
    #!/bin/bash
     cal $1
    ```

1.  ```
    #!/bin/bash
     for i in {2000..2020}; do
            cal $i
     done
    ```

# 知识检查 17

1.  */10 *   *   *   *   echo "10 minutes have passed!" >> /root/minutes.txt

1.  `0 1  25  12  *   echo "Merry Christmas!" >> /root/holidays.txt`

# 知识检查 18

1.  `tar -cvf /root/var.tar.gz /var`

1.  `tar -jvf /root/tmp.tar.bz2 /tmp`

1.  `tar -Jvf /root/etc.tar.xz /etc`

# 知识检查 19

1.  `别名 ins=”apt-get install”`

1.  `别名 packages=”dpkg -l”`

1.  添加这行

`别名 clean="rm -r /tmp/*"`

到`.bashrc`文件的末尾。

# 知识检查 20

1.  转到虚拟机设置>存储>创建新磁盘。

1.  `fdisk /dev/sdc`

1.  `pvcreate /dev/sdc1 /dev/sdc2 /dev/sdc3`

1.  `vgcreate bigvg /dev/sdc1 /dev/sdc2 /dev/sdc3`

1.  `lvcreate -n **biglv** -L 500M **bigvg**`

1.  `mkfs -t **ext4** /dev/mapper/bigvg-biglv`

1.  `mount **/dev/mapper/bigvg-biglv** /mnt/wikileaks`
