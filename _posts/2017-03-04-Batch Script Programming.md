---
layout:     post
title:      Batch Script Programming
subtitle:   兼谈Android SDK Lint Windows启动脚本
date:       2017-03-04
author:     Cliu
header-img: img/post-daily-bg.jpg
catalog: true
tags:
    - DOS
    - Script
    - Batch File
---

```
@goto start
-------------------------------------------------------------
goto label不要冒号，call :label要冒号
call可以调用命令、批处理文件或指定执行点，后一个功能与goto相同
goto是跳转，call是调用，调用在执行完标签部分后会继续执行其后部分
另start不仅可以调用命令、批处理文件，还可以调用应用程序，如
start ping 127.0.0.1 -t
start test.bat
start evernote
start会新开启一个窗口
call和start调用程序时都可添加命令行参数，在程序中用%1~%9引用这些参数即可
批处理中有一个隐式label，那就是eof，表示批处理文件末尾，在call外部调用表示exit，在call内部调用表示break

cmk /k echo. /k选项会在批处理运行完后保留窗口
-------------------------------------------------------------
DOS常用命令:
查看目录内容命令 DIR
创建目录命令 MD
显示文本文件内容命令 TYPE  如type nul > new_file ::新建空文件；echo= ::输出空行
fsutil file createnew new_file 1000可以创建空字符(ASCII的NUL)填充的指定大小的文件
更改文件名命令 REN
设置环境变量 SET 如set /p var=提示语句 ::接收用户输入；set var= ::删除环境变量
set /a  2+2 (arithmetic)用于算术运算，会显示结果
清除屏幕命令 CLS
删除文件 DEL
删除目录 RD
清除当前行 按Esc键
List of DOS commands:
https://en.wikipedia.org/wiki/List_of_DOS_commands
-------------------------------------------------------------
未定义的变量n，引用时%n%会扩展为空，如n+=1-->!n!=1
rem this is a comment. will display, rem represents remark
:: this is a comment. will not display
-------------------------------------------------------------
:start

@echo off
echo %1 say %2.
::批处理中的变量包括系统变量和自定义变量，系统变量如path、windir
::另外%0~%9也是系统变量，用于引用指定批处理的命令行参数，%0是脚本名称
::自定义变量由set定义，可用%var%的形式引用变量
::for循环中引用延迟变量要用!var!，循环变量在命令行中用%var，在批处理中用%%var
pause

ping www.qq.com>BatchTest.txt
ping www.baidu.com>>BatchTest.txt
:: |和>、>>分别是管道和重定向操作符，管道将前一命令的输出当做后一命令的输入
pause

rem IF [NOT] ERRORLEVEL number do command  ::结果判断，可转化为第二种写法
rem IF [NOT] "string1"=="string2" do command  ::输入判断，注意string常量应以""括起来
rem IF [NOT] EXIST filename do command and ::存在判断
if exist C:\Progra~1\Tencent\AD\*.gif del C:\Progra~1\Tencent\AD\*.gif
:: 文件短名规则参见下文
:: if语句的command中如果有多个变量，应该用goto逐条命令引导，否则变量将不起作用，如
:: @echo off
:: set /p n=请输入要执行的操作
:: if "%n%"=="new" (
:: set /p a=请输入文件名：
:: set /p b=请输入文件内容：
:: set /p c=请输入复制后的文件名：
:: goto a
:: :a
:: echo %b% > %a%.txt
:: goto b
:: :b
:: copy %a%.txt C:\Users\CHANG\Desktop\%c%.txt
:: )
:: set a= & set b= & set c=
@pause

@echo off
pause > nul
:: 在DOS中NUL为空设备，>nul屏蔽命令的输出内容；ASCII中的字符'\0'为NUL，用于结束一个ASCII字符串

find用于在文件中搜索字符串，默认区分大小写
find "1336" C:\Users\dp\Desktop\a.txt  ::这里文件使用了绝对路径
find "1336" < a.txt > b.txt
type a.txt | find "7626" && echo "Congratulations! You have infected GLACIER!"
:: 批处理命令按行执行，find命令输出包含关键字的行，关键字必须用引号
:: find和more、sort一起称为筛选器命令，筛选器可从命令行，<或|接收输入。
:: 使用参数/c则仅显示包含指定字符串的行数，/I指定搜索不区分大小写，/v反转搜索结果，即输出所有不匹配的行
:: %errorlevel%是上一条命令的返回值，用find的话，则：
:: %errorlevel%为0的时候，表示find找到字符串
:: %errorlevel%为1的是偶，表示find找不到字符串
:: echo %errorlevel%类似于linux shell中的echo $?，也可以用来查看exe程序中main函数的返回值
:: 假设1.txt为目标文件：
@echo off
find "run" 1.txt >nul
if "%errorlevel%"=="0" (
  echo Running c:\run.exe
) else (
  echo No Run
)
pause

:: &和&&、||一起称为组合命令，组合命令连接的命令被当作一条命令
:: 被&连接的命令将不顾是否出错地按顺序执行
:: 而&&遇到错误就不继续执行，||遇到错误才继续执行

for /? ::查看变量扩展相关知识
set /? ::查看变量替换/子字符串提取相关知识


windows短文件名是dos+fat16时代的产物，又称8.3命名法
fat16为文件/夹名预留了8个字符，为扩展名预留了3个字符，文件名和扩展名以点(.)分割
这里要注意的是通过%%~xi得到的文件扩展名包括点.
自win95引入fat32之后，已支持长文件名，可用dir /x查看短名
只有文件名超过8字符 or 扩展名超过3字符or 文件名有空格 的文件才有短名

短文件名的创建规则是：
文件名部分A：超过8字符时取其前6个字符后面加上~1
扩展名部分B：超过3字符时取前3个字符
A.B即为短名，如果存在多个文件A.B形式相同，则后面依次用~2，~3，~4以区分
含空格的文件名不论其字符多少都会有短名，短名会删除文件名中的空格，如

12345678.txt不存在短名
1 2.txt的短名可能为125ACA~1.TXT，此时(字节数少于3时？)文件名被随机补齐为6个字符
123.html的短名为123~1.htm
123456789.htm的短名为123456~1.htm
同一文件夹下123456789.txt的短名为123456~1.txt
1234567890.htm的短名则为123456~2.htm
----------------------------------------------------------------------------
C盘根目录下进入Program Files文件夹可用三种方式：
cd "Program Files"
cd Progra~1
cd pro*
同时可用cd progra~2进入Program Files （x86）文件夹

cd用于更改工作目录，不能改变工作盘符
cd C rem C不是一个有效路径，提示找不到路径
cd C:\ rem C:\是C盘根目录，可以
cd D: rem无错，但无效，更改盘符应直接用盘符D:作为命令，然后才可以用cd改变工作目录
在C盘工作目录中cd D:\Android无效，但是如果紧接着使用D:更改盘符，则会直接进入D:\Android目录
-----------------------------------------------------------------------------

批处理中逗号，等号=分号；都是分隔符，相当于空格
如dir,C:\      dir;C:\       dir=C:\
另echo,       echo=       echo;和echo.等一样都输出空行，而且效率更高
这涉及到批处理的预处理机制，如cmd能识别unix的路径分隔符/也是因为在其预处理阶段就将/替换为\了
批处理技术内幕http://demon.tw/reverse/cmd-internal-echo.html
批处理之预处理http://bbs.bathome.net/viewthread.php?tid=3349
-----------------------------------------------------------------------------
@echo off
title 具有一定格式的批处理文件
cls
:start
echo.
echo. ---------1、输入数字1并回车，注销系统
echo. ---------2、输入数字2并回车，重启系统
echo. ---------3、输入数字3并回车，关闭系统
echo. ---------4、输入字母q并回车，退出程序
echo. ---------5、输入其他字符，显示帮助
set /p var=请选择你要执行的操作：
if "%var%" =="1" (
logoff
)
if "%var%" =="2" (
cls
shutdown -r /t 000 > nul
echo 系统将立即关机············
)
if "%var%" =="3" (
shutdown -p
)
if "%var%" =="q" (
exit
) else (
cls
echo 你输入的是%var%字符··········
goto start
)
set var=
:: 以上应注意左括号(和右括号)（作用：grouping）之前的空格，注意是"%var%"而不是"var"也不是%var%
:: 因为1=="1"永远不成立，改写成%var%=1，即把1周围的""也去掉也可
--------------------------------------------------------------------------
显示和设置时间/日期
date [mm-dd-yy]
月日年的有效分隔符还包括.及/
time [h:[m:[s]]][a|p]
不带任何参数是，两者都是显示并提示输入，可用/t参数控制仅显示

改变cmd窗口的颜色
color [fg]
前景色f和背景色g由十六进制数字指定，如
color FC 前景色设为红色，背景色设为白色

改变cmd命令提示符
prompt [text]
除了普通文本，text也可以包含一些表示特殊含义的字符组合（以$开头），如
$t 当前时间
$d 当前日期
$p 当前驱动器和路径
$v Windows版本号
$g 大于号>
$l 小于号<
$s 空格
$_ 回车换行

call命令对变量扩展的影响
set n=123
::case1. 常规写法，完美执行输出123
::for %%i in (1) do echo %n%
::case2. 无法正确执行，将显示字符串%n%
for %%i in (1) do echo %%n%%
::case3. call命令使echo命令正确扩展%n%
for %%i in (1) do call echo %%n%%
pause

在运行case2的时候，我们看到的echo是
for %i in (1) do echo %n%
这是cmd读取命令并执行变量扩展之后的结果，可以看到%%n%%被扩展成了%n%
由于for把echo扩展变量的权力剥夺了，所以echo %n%显示的是%n%而不是真实值123

综上，如果想引用一个不被for扩展而在do里扩展的变量，也就是说想达到延迟环境变量的效果，应该使用call。

@echo off
title 阶乘--递归算法
setlocal enabledelayedexpansion
set /p n=请输入一个非负整数 ：
set result=1
if !n!==0 (
echo 结果等于 1 & pause>nul & goto eof
) else (
call :loop !n!
)
echo. & echo 结果等于!result!
pause>nul

:loop
if not %1==1 (
set /a result=!result!*%1
set /a x=%1
set /a x-=1
call :loop !x!
)
```
