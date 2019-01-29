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
``
