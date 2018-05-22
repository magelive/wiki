# shell 快捷键

Alt + b				后移一个单词
Alt + f				前移一个单词
Ctrl + b			后移一个字符
Ctrl + f			前移一个字符
Ctrl + a			把光标移到行首
Ctrl + e			把光标移到行尾
ctrl + ←			光标移动到前一个单词开头
ctrl + →			光标移动到后一个单词结尾
Ctrl + x Ctrl + x	在 EOL 和当前位置移动光标

Ctrl + h			删除前一字符
Ctrl + d			删除当前字符
Ctrl + k			删除当前字符到行末
Ctrl + u			删除行首到当前字符
Ctrl + w			删除单词到当前字符
Alt + d				从当前位置向后删除单词
Alt + ←				从当前位置向前删除单词
Esc + t				互换相邻两个单词
Alt + t				互换相邻两个单词
Ctrl + t			互换相邻两个字符
ctrl + ?			撤消前一次输入
Alt + r				撤消前一次动作
Alt + l				小写当前单词
Alt + u				大写当前单词
Alt + c				首字母大写当前单词
^oldstr^newstr		替换前一次命令中字符串
Ctrl + s			锁住终端
Ctrl + q			解锁终端
Ctrl + l			清除终端
Ctrl + d			退出终端
Ctrl + c			中止命令
Ctrl + z			挂起命令
ctrl + o			重复执行命令
Ctrl + r			向后查询历史，增量地
Ctrl + s			向前查询历史，增量地
Alt + p				向后查询历史，非增量地
Alt + n				向前查询历史，非增量地
Ctrl + p / ↑		显示上一条命令
Ctrl + n / ↓		显示下一条命令
Alt + <				移动到历史的首行
Alt + >				移动到历史的末行
Alt + .				插入最后一个参数
Alt + _				插入最后一个参数
Esc + .				插入最后一个参数
Esc + _				插入最后一个参数
Ctrl + y			粘贴刚才所删除的字符
Ctrl + Alt + y		插入上条命令的第一个参数
history				查看历史命令，按顺序全部显示出来，有对应的编号。 
!num				执行history历史命令列表中第num条命令。 
!!					执行上一条命令。 
!?string?			执行含有string字符串的最新命令。 
ls !$				执行命令ls，并以上一条命令的最后一个字符串为其参数。

Ctrl + i			同 Tab
Ctrl + j			同 Enter
Ctrl + v CHAR		输入特殊字符
Ctrl + x @			显示所有的可用的主机名自动完成
Ctrl + x Ctrl + e	使用 vim 写入 script 一次执行
2T					命令行补全
(string)2T			命令行补全
$2T					列出系统变量
=2T					列出当前目录
/2T					显示整个目录结构，包括隐藏文件
./2T				只显示子目录，包括隐藏目录



*2T					只显示子目录，不包括隐藏目录
@2T					“/etc/hosts” 文件的条目
~2T					/etc/passwd” 文件中系统所有的当前用户
