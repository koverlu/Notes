# Rsync
常用指令：
选项 |说明
-|-
-a, ––archive | 归档模式，表示以递归方式传输文件，并保持所有文件属性，等价于 -rlptgoD (注意不包括 -H)
-r, ––recursive | 对子目录以递归模式处理
-l, ––links	| 保持符号链接文件
––exclude=PATTERN | 指定排除一个不需要传输的文件匹配模式
––exclude-from=FILE | 从 FILE 中读取排除规则
––include=PATTERN | 指定需要传输的文件匹配模式
––include-from=FILE | 从 FILE 中读取包含规则
-L, --copy-links | 将符号链接处理为具体的文件或者文件夹
––copy-unsafe-links | 拷贝指向SRC路径目录树以外的链接文件
––safe-links | 忽略指向SRC路径目录树以外的链接文件（默认）
––existing | 仅仅更新那些已经存在于接收端的文件，而不备份那些新创建的文件
––ignore-existing | 忽略那些已经存在于接收端的文件，仅备份那些新创建的文件
-b, ––backup | 当有变化时，对目标目录中的旧版文件进行备份
––backup-dir=DIR | 与 -b 结合使用，将备份的文件存到 DIR 目录中
––link-dest=DIR | 当文件未改变时基于 DIR 创建硬链接文件
––delete | 删除那些接收端还有而发送端已经不存在的文件
––delete-before | 接收者在传输之前进行删除操作 (默认)
––delete-during | 接收者在传输过程中进行删除操作
––delete-after | 接收者在传输之后进行删除操作
––delete-excluded | 在接收方同时删除被排除的文件
-m | 不同步空目录
-v, ––verbose | 详细输出模式
-q, ––quiet | 精简输出模式
-n, ––dry-run | 显示哪些文件将被传输
––list-only | 仅仅列出文件而不进行复制

```bash
# 不同步config/kkk目录，同步所有头文件（如有链接，将复制源文件），在接收端删除发送端没有的文件。
rsync -avL --delete --exclude "config/kkk/" --include "*/" --include "*.h" --include "*.hpp" --exclude "*" ./ xxx@xxUBT:/home/xxx/workspace/xxx
# 同步所有代码文件，忽略空目录
rsync -avm --include "*/" --include "*.c" --include "*.cpp" --include "*h" --include "*.hpp" --exclude "*" ../w1/gc/src/verif code/
```