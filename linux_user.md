创建账户

```bash	
useradd -m username
passwd username
chown -R username /home/username
```

```shell
useradd -m username -d /home/username
```

在一个现有目录下建用户：

```bash
useradd -m username -d /usr1/username
passwd username
chown -R username /usr1/username
vim ~/.profile
```

```bash
# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi
```

```bash
vim ~/.bashrc
```

```bash
# .bashrc
# Source global definitions
if [ -f /etc/bashrc ]; then
    . /etc/bashrc
fi
```

删除用户

```shell
userdel username
```

文件/文件夹权限管理, chmod：
```bash
chmod [-cfvR] [--help] [--version] mode file...
```
mode参数说明：
```bash
[ugoa...][[+-=][rwxX]...][,...]
```
其中：
* u 表示该文件的拥有者，g 表示与该文件的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示这三者皆是。
* + 表示增加权限、- 表示取消权限、= 表示唯一设定权限。
* r 表示可读取，w 表示可写入，x 表示可执行，X 表示只有当该文件是个子目录或者该文件已经被设定过为可执行。
其他参数说明：
-c : 若该文件权限确实已经更改，才显示其更改动作
-f : 若该文件权限无法被更改也不要显示错误讯息
-v : 显示权限变更的详细资料
-R : 对目前目录下的所有文件与子目录进行相同的权限变更(即以递回的方式逐个变更)
--help : 显示辅助说明
--version : 显示版本

实例：
将文件 file1.txt 设为所有人皆可读取 :
```
chmod ugo+r file1.txt
```
将文件 file1.txt 设为所有人皆可读取 :
```
chmod a+r file1.txt
```
将文件 file1.txt 与 file2.txt 设为该文件拥有者，与其所属同一个群体者可写入，但其他以外的人则不可写入 :
```
chmod ug+w,o-w file1.txt file2.txt
```
将 ex1.py 设定为只有该文件拥有者可以执行 :
```
chmod u+x ex1.py
```
将目前目录下的所有文件与子目录皆设为任何人可读取 :
```
chmod -R a+r *
```

此外chmod也可以用数字来表示权限如 :
```
chmod 777 file
```
语法为：
```
chmod abc file
```
其中a,b,c各为一个数字，分别表示User、Group、及Other的权限。

r=4，w=2，x=1
若要rwx属性则4+2+1=7；
若要rw-属性则4+2=6；
若要r-x属性则4+1=5。
```
chmod a=rwx file
```
和
```
chmod 777 file
```
效果相同
```
chmod ug=rwx,o=x file
```
和
```
chmod 771 file
```

统计信息：

```bash
wc -l
cat logfile | grep listening | wc -l
```

首先是压缩：
```bash
tar -czvp -f skype_backup.tar.gz skype_backup
```

分割：
```bash
split -b 4000k skype_backup.tar.gz skype_backup_20090626.tar.gz. –verbose
```

如上两句命令合并为一句：
```bash
tar -czvp -f – skype_backup |split -b 4000k – skype_backup_20090626.tar.gz. –verbose
```
合并文件：
```bash
cat skype_backup_20090626.tar.gz.a* >skype_backup_cat.tar.gz
```
解压：
```bash
tar -zxvf skype_backup_cat.tar.gz
```
