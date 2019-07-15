# VSCode配置Git
## 安装Git
官方网站：https://git-scm.com/

国内镜像：https://github.com/waylau/git-for-win

## 配置git
任意目录下，右键鼠标，`Git bash here`
```bash
git config --global user.name "your name" 
git config --global user.email "your email address"
ssh-keygen -t rsa -C "your email address"
cat ~/.ssh/id_rsa.pub
```
github->Settings->SSH and GPG Keys->New SSH ley

## 克隆仓库
```bash
git clone https://github.com/koverlu/Notes
```

## 配置VSCode
右键Notes文件夹，`Open With VSCode`。
File->Preferences->Settings，搜索gitpath，打开后添加：
```json
{
    "git.path": "C:/Program Files/Git/bin/git.exe"
}
```
