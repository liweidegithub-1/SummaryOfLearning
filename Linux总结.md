# Linux总结

## 一、awk

```shell
1、格式：awk [参数] '$n~/匹配规则/{执行命令}' 需要处理的文件
2、-F 指定间隔符
3、-f 指明awk脚本
4、$n 选取文本间隔后的第n列。$0代表整个文本，$1代表间隔后的第一列数据
5、~ 用来匹配正则表达式
6、-v var=val 在执行处理过程之前，设置一个变量var，并给其设置一个初始值val
7、BEGIN关键字 BEGIN{执行命令} 会强制awk在读取数据之前执行该关键字后指定的脚本命令
8、END关键字 END{执行命令} awk会在读取完数据后执行
9、awk -F " " '{if($1==55) ($1=$1+2); print > "test.txt"}' test.txt
	匹配test.txt文件中第一列数据为55的一行数据，并将其第一列数据加2，将文件的修改操作写入到原文件中，交互界面会打印出文件中所有的数据。
10、awk -F " " '{if($1==57) {$1=$1+2;print}}' test.txt
	只打印文件中修改过的数据A
11、gsub(/s/,t) 在整个文件数据中将s替换为t
	gsub(/s/,t,str) 将str中的所有s替换为t
12、index(s,t) 查询字符串s中t出现的第一个位置，必须用双引号将字符串括起来
13、length(s) 返回s的长度
14、match(s,r) 测试s是否包含匹配r的字符串。不存在返回0，存在则返回出现r的第一个位置
15、split(s,a,fs) 将数据s按照fs分隔符进行分割，分割后的数据存储至数组a中，若要提取数组中的数据可以使用a[n]
	awk 'BEGIN{print split("12/34/56/78",a,"/"); print a[1]}'
	将数据"12/34/56/78"中的数据按照分隔符"/"进行分割，分割后的数据存储在数组a中，提取数组a中的第一个数据
16、sub(s,t,r) 将r中第一次出现的s替换为t
17、substr(str,n1,n2) 从str中的第n1个字 符开始，返回其前面的n2个字符
18、awk脚本头 #!/bin/awk
19、精确匹配
	awk '$1 == "123" {print $1}' file file文件的第一列数据为123的进行打印输出
20、不匹配
	awk '$1 !~ /123/ {print $1}' flie file文件的第一列数据包含123的进行打印输出
21、NF 表示列数
22、NR 表示行数
23、awk中的语法不同于普通shell，与C类似。例如for循环、if判断等
```

## 二、grep

```shell
1、格式：grep [选项] 正则表达式/字符串 [文件] 输入字符串参数时，最好使用双引号括起来
2、选项参数
	-c 只输出匹配行的计数
	-h 查询多文件时不显示文件名
	-l 查询多文件时只输出包含匹配字符的文件名
	-n 显示匹配行及行号
	-s 不显示不存在或无匹配文本的错误信息
	-v 显示不包含匹配文本的所有行
	-i 不区分大小写（只适用于单字符）
	-w 指定匹配的只能是一个完整的单词，不能是其中的一部分
3、精确匹配
	每个匹配模式后加一个<Tab>键或\>
4、模式出现机率
	grep '4\{2,\}' 抽取包含数字4至少重复出现两次的所有行
	grep '4\{2\}' 抽取包含数字4重复出现两次的所有行
	grep '4\{2,6}5' 抽取数字4重复出现2到6次并以5结尾的所有行
5、grep命令加-E参数，这一扩展允许使用扩展模式匹配
	grep -E '219|216' 抽取包含219或216的所有行
```

## 三、sed

```shell
1、格式：sed [选项] sed命令 输入文件
		sed [选项] -f sed脚本文件 输入文件
		sed脚本文件 [选项] 输入文件
2、选项
		-e 后跟脚本命令
		-f 后跟脚本文件
		-n 屏蔽启动输出
3、使用p显示行
		sed -n '2p' 打印文本第二行
		sed -n '1,3p' 打印文本第一到第三行
		sed -n '/neave/'p 打印匹配到单词neave的行
		sed -n '4,/the/'p 从第四行开始进行匹配查询，直至匹配到the为止
		sed -n '1,$p' 打印整个文件，$意为最后一行
		sed -n -e '/root/'p -e '/root/=' 打印匹配行及行号 =会打印行号
		sed -n -e '/root/=' 只会打印匹配行的行号
4、附加文本
		#!/bin/sed -f
		/000002222299999444477777/ a\
		this is text
		/3333444444777779999000/ a\
		this is text too
	脚本解释：脚本第一行是sed命令解释行；第二行以/000002222299999444477777/开始，这是附加操作起始位置，a\通知sed这是一个附加操作，应该插入一个新行；第三行是附加操作要加入到拷贝的实际文本。（附加文本的每一行都要添加"\"，表示换行，最后一行不需要添加）。如果要保存输出，则需要进行重定向。
5、插入文本
		#!/bin/sed -f
		/77777/ i\
		this is text
	脚本解释：脚本第一行是sed命令解释行；第二行/77777/在其之前插入第三行的数据，i\通知sed这是一个附加操作，应该插入一个新行；第三行是要插入的文本。
6、修改文本
		#!/bin/sed -f
		/777/ c\
		this is text
	脚本解释：脚本第一行是sed命令解释行；第二行/777/是需要修改的行，将其修改为第三行的数据；c\通知sed这是一个附加操作，应该修改一行数据；
7、删除文本
		sed '1d' 删除第一行
		sed '1,3d' 删除一到三行
		sed '$d' 删除最后一行
		sed '/neave/d' 使用正则表达式进行删除操作
8、替换文本
		sed 's/night/NIGHT/' 将文本中每一行的第一个night替换为NIGHT
		sed 's/night/NIGHT/g' 将文本中的所有night替换为NIGHT（g意为匹配所有文本）
		sed 's/splendid/SPLENDID/w sed.out' 将文本中的splendid替换为SPLENDID，并且输出至文件sed.out中（w选项后接一个文件，意为将结果写入至这个文件）
9、插入字符串
		sed -n 's/played/from Hockering/ &/p' 在played之前插入from Hockering
10、将sed结果写入文件命令
		sed '1,2 w filedt' 将输入文本的1、2行写入至filedt中（w通知sed将文本写入至目标文件中）
		sed '/neave/ w dht' 将输入文本中匹配到neave的结果写入至dht中
11、从文本中读取文本
		sed '/company/r sedex.txt' quote.txt 将sedex.txt中的内容插入至quote.txt文件中匹配行/company/后	
12、匹配后退出
		sed '/.a.*/q' 匹配到文本中的规则之后退出
13、sed命令可以结合多个操作指令，每个指令之间通过";"分割
		sed 's/^.//;s/.$//' file 同时移除每一行的第一个字符和最后一个字符
14、删除每一行的前n个字符
		sed -r 's/.{n}//' file
15、删除每一行的后n个字符
		sed -r 's/.{n}$//' file
16、除了每一行的前n个字符外，剩下的都要删除
		sed -r 's/(.{3}).*/\1/' file 
		命令解释：.{3}匹配每一行的头3个字符，并且用()进行分组，.*表示匹配任意多个字符。在替换位通过\1表示保留第一个分组的内容。
17、删除每一行所有字符且保留结尾的n个字符
		sed -r 's/.*(.{n})/\1/' file
18、删除每一行匹配到的第n个字符
		sed 's/u//2' file
		命令解释：sed默认只会处理匹配到的第一个字符，可以指定其处理匹配到的第n个字符。如上命令将会把每一行匹配到的第二个u删除。
19、删除每一行中出现的所有数字
		sed 's/[0-9]//g' file
20、删除除了小写字符之外的其他所有字符
		sed 's/[^a-z]//g' file
21、追加/插入文本
		追加：sed 's/li/&wei/g' file
			命令解释：在file中的每一行的li之后追加wei
			sed 's/$/wei/g' file
			命令解释：在file中的每一行末尾追加wei
		插入：sed 's/li/wei&/g' file
			命令解释：在file中的每一行的li之前插入wei
			sed 's/^/wei/g' file
			命令解释：在file中的每一行的开头插入wei

```

## 四、find命令

```shell
1、格式：find pathname -opentions [-exec -print -ok]
		解释：-print 将匹配结果打印到交互界面
			 -exec 对匹配的文件执行该参数所给出的shell命令。命令格式：'command {} \;'
			 -ok 和-exec作用相同，只不过是以一种更加安全的模式来执行该参数所给出的shell命令
2、find命令选项
		-name 按照文件名查找文件
		-perm 按照文件权限来查找文件
		-prune 使用这一选项可以使find命令不在当前指定的目录中查找，如果同时使用了-depth选项，那么-prune选项将被find命令忽略
		-user 按照文件属主来查找文件
		-group 按照文件所属的组来查找文件
		-mtime -n +n 按照文件的更改时间来查找文件，-n表示文件更改时间距现在n天以内，+n表示文件更改时间距现在n天以前。
		-nogroup 查找无有效属组的文件，即该文件所属的组在/etc/groups中不存在
		-nouser 查找无有效属主的文件，即该文件的属主在/etc/passwd中不存在
		-newer file1 ! file2 查找更改时间比文件file1新但是比file2旧的文件
		-type 查找某一类型的文件。
				b 块设备文件
				d 目录
				c 字符设备文件
				p 管道文件
				l 符号链接文件
				f 普通文件
		-size n[c] 查找文件长度为n块的文件，带有c时表示文件长度以字节计
		-depth 查找文件时，首先查找当前目录中的文件，然后在其子目录中查找
		-fstype 查找位于某一类型文件系统中的文件，这些文件系统类型通常可以在配置/etc/fstab中找到，该配置文件中包含了本系统中有关文件系统的信息
		-mount 在查找文件时不跨越文件系统mount点
		-follow 如果find命令遇到符号链接文件，就跟踪至链接所指向的文件
3、使用name选项
		find ~ -name "*.txt" -print
			命令解释：在根目录$HOME中查找文件名符合*.txt的文件，~代表$HOME目录
		find . -name "*.txt" -print
			命令解释：在当前目录及子目录中查找所有的*.txt文件
4、使用perm选项
		find . -perm 755 -print
			命令解释：在当前目录中查找文件权限为755的文件
5、忽略某个目录
		find . -path "需要忽略的目录" -prune -o -name "*.txt" -print
			命令解释：在当前目录及其子目录下除了需要忽略的目录中查找后缀为txt的文件并打印到交互界面。-o意为or，以上指令顺序不能错
		find . \(-path "需要忽略的目录1" -o -path "需要忽略的目录2"\) -a -prune -o -name "*.txt" -print
			命令解释：在当前目录及其子目录下除了两个需要忽略的目录中查找后缀为txt的文件并打印到交互界面。-a意为and
		 find . \(-path "目录1" -o -path "目录2"\) -a -name "*.txt" -print
		 	命令解释：在目录1和目录2及其子目录中查找后缀为txt的文件
6、使用user和nouser选项
		find . -user liwei -print
			命令解释：在当前目录中查找属主为liwei的文件并打印到交互界面
		find . -nouser -print
			命令解释：在当前目录中查找属主账户已经被删除的文件
7、使用group和nogroup选项
		find . -group liwei -print
			命令解释：在当前目录中查找liwei用户组的文件
		find . -nogroup -print
			命令解释：在当前目录中查找没有有效所属用户组的所有文件
8、按照更改时间查找文件
		find . -mtine -5 -print
			命令解释：在当前目录下查找更改时间在5日以内的文件
		find . -mtime +3 -print
			命令解释：在当前目录下查找更改时间在3日以前的文件
9、查找比某个文件新或旧的文件
		find . -newer file1 ! -newer file2
10、使用type选项
		find . -type d -print
			命令解释：查找当前目录下所有的目录
		find . ! -type d -print
			命令解释：查找当前目录下除了目录以外的所有类型的文件
		find . -type l -print
			命令解释：查找当前目录下所有的符号链接文件
11、使用size选项
		find . -size +100c -print
			命令解释：在当前目录下查找文件长度大于100字节的文件
		find . -size 100c -print
			命令解释：在当前目录下查找文件长度恰好为100字节的文件
		find . -size -10 -print
			命令解释：在当前目录下查找长度超过10块的文件（一块等于512字节）
12、使用depth选项
		find . -name "匹配文件" -depth -print
			命令解释：先在当前目录下进行查找，再进入子目录进行查找
13、使用mount选项
		find . -name "匹配文件" -mount -print
			命令解释：在当前的文件系统中查找文件，不进入其他文件系统
```

## 五、流程控制

```shell
1、if语句
	if [ 条件判断式 ]
	then
		程序
	elif [ 条件判断式 ]
	then
		程序
	else
		程序
	fi
2、case语句
	case $变量名 in
	"值1")
		如果变量的值等于值1，则执行程序1
	;;
	"值2")
		程序2
	;;
	*)
		如果变量的值与以上值都不匹配，则执行此程序
	;;
	esac
3、for循环	(遍历文本时一个一个的读取打印)
	3.1、方式一：
		for ((初始值;循环条件;变量变化))
		do
			程序
		done
	3.2、方式二：
		for 变量 in 值1 值2 值3...
		do
			程序
		done
4、while循环	(遍历文本的时候一行一行的打印，所见即所得)
	while [ 条件判断式 ]
	do
		程序
	done
5、until循环
	until 条件
	do
			程序1
			程序2
			...
	done
```

## 六、后台执行命令

```shell
1、crontab
	1.1、格式：crontab [-u user] -e -l -r
	1.2、参数
		-u 用户名。如果使用自己的名字登录，就不用使用-u选项，因为在执行crontab命令时，该命令能够知道当前的用户
		-e 编辑crontab文件
		-l 列出crontab文件中的内容
		-r 删除crontab文件
	1.3、crontab文件域
		第一列：分钟1~59
		第二列：小时1~23(0表示子夜)
		第三列：日1~31
		第四列：月1~12
		第五列：星期0~6(0表示星期天)
		第六列：要运行的命令
	1.4、提交方式
		crontab 提交的文件名
2、at
	at命令允许用户向cron守护进程提交作业，使其在稍后的时间运行。
	1、格式：at [-f 脚本] [-m -l -r] [time] [date]
	2、参数
		-f 指明需要提交的脚本
		-l 列出当前所有等待运行的作业
		-r 清除作业
		-m 作业完成后给用户发邮件
		time 时间格式非常灵活，可以使H、HH.HHMM、HH:MM或H:M
		date 日期格式可以使月份数或日期数，而且at命令还能识别today、tomorrow这样的词。
	3、提交命令或脚本
		3.1、提交若干行的命令，可以在at命令后面跟上日期时间并回车，然后这就进入了at命令提示符，只需逐条输入相应的命令，最后键入ctrl+d退出。
		3.2、提交一个shell脚本，使用-f选项
	4、清除一个作业
		atrm job no 或 at -r job no
		job no 为作业标识，先用at -l列出所有作业，第一列为作业标识
3、&
	可以使用&命令将作业放到后台执行，但是退出时该进程会被终止。而nohup会使后台进程能够在退出后继续运行。
	格式：命令 &
		命令 > 输出文件 2>&1 & 将标准输出以及错误输出都重定向到输出文件中
4、nohup
	格式：nohup 命令 &
```



## 七、文件的分割与合并

```shell
1、split用法
	1.1、格式
		split 参数 输入文件 输出文件
	1.2、参数
		-a 生成长度为n的后缀（默认值为2）
		-b 按大小切割文件
		-C 按字节切割文件
		-d 使用数字后缀替代字母
		-e 不生成带有'-n'的空输出文件
		-l 按行切割文件
		-n 按生成文件个数切割
		-u 打印日志
	1.3、按文件大小切分，并指定后缀
		split -b 1k input_file -d -a 1 date_
		结果：
			date_0
			date_1
	1.4、按字节切分
		split -C 200 input_file -d -a 1 date_
	1.5、按行数切割，并重命名文件
		split -l 50 input_file -d -a 1 date_
	1.6、按输出文件个数切割
		split -n 5 input_file -d -a 2 date_
		结果：
			date_00
			date_01
			date_02
	1.7、批量为文件添加后缀
		ls | grep date_ | xargs -n1 -i{} mv {} {}.txt
2、sort用法
	2.1、格式
		sort [-bcdfimMnr][-o 输出文件][-t 间隔符][输入文件][-k 按指定的列进行排序]
	2.2、参数
		-b 忽略每行前面开始出的空格字符
		-c 检查文件是否已经按照顺序排序
		-d 排序时，处理英文字母、数字及空格字符，忽略其他字符
		-f 排序时，将小写字母视为大写字母
		-i 排序时，除了040至176之间的ASCLL字符外，忽略其他字符
		-m 将几个排序好的文件进行合并
		-M 将前面3个字母依照月份的缩写进行排序
		-n 依照数值大小进行排序
		-u 意味着是唯一的(unique)，输出的结果是去重了的
		-o 输出文件，将排序后的结果写入指定的文件
		-r 以相反的顺序进行排序
		-t 分隔符，指定排序时多用的栏位分隔符
		+起始栏位-结束栏位，以指定的栏位来排序，范围由起始栏到结束栏位的前一栏位
		-k 起始列，终止列，按指定的列进行排序
3、uniq用法
		用于从一个文本文件中取出或禁止重复行。
		1、格式：uniq [参数] 输入文件 输出文件
		2、参数
			-u 只显示不重复行
			-d 只显示有重复数据行，每种重复行只显示其中一行
			-c 打印每一重复行出现次数
			-f 前n个域被忽略。一些系统不识别-f选项，这时替代使用-n
4、join用法
		用来将来自两个分类文本文件的行连接在一起
		1、格式：join [参数] 输入文件1 输入文件2
		2、参数
			-an n为数字，用于连接时从文件n中显示不匹配行。类似于左外连接或右外连接
			-o n.m n为文件号，m为域号。1.3表示只显示文件1第三域。每个n.m之间必须使用','进行分隔
			-j n m n为文件号，m为域号。使用其他域做连接域
			-t 域分隔符。用来设置非空格或tab键的域分隔符。例如，指定冒号做域分隔符'-t:'
5、cut用法
		用来从标准输入或文本文件中剪切列或域。剪切文本可以将之粘贴到一个文本文件。
		1、格式：cut [参数] 文件1 文件2
		2、参数
			-c list 指定剪切字符数
			-f filed 指定剪切域数
			-d 指定域空格和tab键不同的域分隔符
			-c 指定剪切范围
				如：-c1,5-7 剪切第一个字符，然后是五到七个字符
					-c1-50 剪切前五十个字符
					-f1,5 剪切第一域，第五域
					-f1,10-12 剪切第一域，第五域
6、paste用法
		按行将不同文本信息放在一起
		1、格式：paste [参数] -文件1 文件2
		2、参数
			-d 指定不同于空格或tab键的域分隔符。如-d@
			-s 将每个文件合并成行而不是按行粘贴
			- 指定每一行输出几列，有几个"-"就输出几列。
				如：ls | paste -d":" - - 每一行输出两列，使用":"进行分割
7、tr用法
		用于转换或删除文件中的字符。
		tr指令从标准输入设备读取数据，经过字符串转译后，将结果输出到标准输出设备
		1、格式：tr [参数] set1 [set2]
		2、参数
			-c,--complement:反选设定字符。也就是符合set1的部分不做处理，不符合的剩余部分才进行转换
			-d,--delete:删除指令字符
			-s,--squeeze-repeats:缩减连续重复的字符成指定的单个字符
			-t,--truncate-set1:消减set1指定范围，使之与set2设定长度相等
		eg：
			cat testfile | tr a-z A-Z
			cat testfile | tr [:lower] [:upper]
			解释：将文件testfile中的所有小写字母全部转换为大写字母
			echo "aaaasdhaaaja" | tr -s [a]  结果---> asdhaja
			解释：压缩重复字符a，使重复出现的字符a只保留一个
			echo "asdjaskd219382938kdjkdj" | tr -d [a-z]    结果--->219382938
			解释：删除字符串中的英文字符
			echo "hdjad38492jkdj" | tr -t [hdja] [abcd]    结果--->abcdb38492ckbc
			解释：将set1中字符用set2对应位置的字符进行替换
			echo "asdjdioj234343" | tr -c [a-z] "*"    结果--->asdjdioj*******
			解释：用set2替换set1中没有包含的字符
```

## 八、shell函数

```shell
1、格式：
	函数名(){
		命令1
		命令2
		……
	}
2、向函数中传递参数
	当向函数中传入参数后，在调用这个函数时，必须在其函数名之后再次传入参数
		如sum $1 $2 ...
3、从调用函数中返回
	使用return关键字，只能返回0~255之间的数，如果返回值大于255，将返回他与255相除的余数
4、删除shell函数
	使用unset关键字，unset 函数名
```

## 九、条件测试

```shell
1、test测试文件状态
	1.1、格式：test condition或[ condition ]
	1.2、参数
		-d 目录
		-f 普通文件
		-L 符号连接
		-r 可读
		-s 文件长度大于0，非空
		-w 可写
		-u 文件是否有suid位
		-x 可执行
		-e 文件是否存在
		-a 逻辑与，操作符两边均为真，结果为真，否则为假
		-o 逻辑或，操作符两边至少一边为真，否则为假
		！ 逻辑否，条件为假，结果为真
2、test字符串测试
	2.1、格式：
		test "string"
		test string_operator "string"
		test "string" string_operator "string"
		[string_operator string]
		[string string_operator]
	2.2、这里string_operator可为：
		= 两个字符串相等
		!= 两个字符串不等
		-z 空串
		-n 非空串
3、数值测试
	3.1、格式：
		"number" numeric_operator "number"
		["number" numeric_operator "number"]
	3.2、numeric_operator可为：
		-eq 数值相等
		-ne 数值不相等
		-gt 大于
		-lt 小于
		-le 小于等于
		-ge 大于等于
4、expr
	expr命令一般用于整数值，但也可以用于字符串
	1、格式：expr argument operator argument
		expr 10 + 10
		expr 30 / 3
		expr 30 \* 3 (使用乘号时，必须用反斜线屏蔽其特定含义)
	2、增量计数
		loop=0
		loop=`expr $loop + 1`
		命令解释：首先初始化loop为0，然后将使用expr将loop放入循环中
	3、数值测试
		value=12
		expr $value + 10 > /dev/null 2>1&
		echo $?
		命令解释：对一个变量进行数值运算，若是$?返回0或是expr返回1，则表明这个变量为数值。特别注意，expr与系统返回状态不同，返回1为正确。
	4、模式匹配
		使用冒号计算字符串中字符数
		value=accounts.doc
		expr $value : '.*'  结果为12
		使用字符串匹配操作，抽取字符串
		expr $value : '\(.*\).doc' 结果为accounts
```

## 十、命令执行顺序

```shell
1、使用&&
	使用形式
		命令1 && 命令2	命令1执行成功才会执行命令2
2、使用||
	使用形式
		命令1 || 命令2	命令1执行失败才会执行命令2
3、使用()和{}将命令结合在一起
	3.1、在当前shell中执行一组命令
		(命令1;命令2;……)
	3.2、在子shell中执行一组命令
		{命令1;命令2;……} 只有在{}中所有命令的输出作为一个整体被重定向时，其中的命令才会被放到子shell中执行，否则在当前shell中执行
```

## 十一、环境和shell变量

```shell
1、本地变量
	1.1、本地变量在用户现在的shell生命期的脚本中使用，这个值只在用户当前shell生命期有意义，如果在shell中启动另一个进程或退出，此值将无效。
	1.2、定义格式
		variable-name=value 或 ${variable-name=value}
	1.3、显示变量
		echo $变量名
	1.4、清除变量
		unset 变量名
	1.5、显示所有本地shell变量
		set
	1.6、结合变量值
		echo ${变量1}${变量2}……
	1.7、测试变量是否已经设置
		var=value	设置实际值到var
		var+value	如果设置了var，则重设其值
		var:?value	如果未设置var，显示未定义用户错误信息
		var?value	如果未设置var，显示系统错误信息
		var:=value	如果未设置var,设置其值
		var:-value	同上，但是如果取值并不设置到var，可以被替换
	1.8、使用变量来保存系统命令参数
		使用变量保存文件拷贝的文件名信息
			SOURCE="/etc/passwd"
			DEST="/tmp/passed.bak"
			cp ${SOURCE} ${DEST}
	1.9、设置只读变量
		readonly 变量名
2、环境变量
	2.1、设置环境变量
		variable=value;export variable
	2.2、显示环境变量
		echo $变量名
		env查看所有的环境变量
	2.3、清除环境变量
		unset 变量名
3、局部变量
	local name
	在函数中使用局部变量，保证了变量只局限在该函数中
	#!/bin/bash
	fun(){
		local name=10
	}
	name=5
	echo name
	最终的输出结果为5
4、数组变量
	4.1、创建数组，不同元素之间以空间隔
	arr=(1 2 3)
	4.2、获取数组元素
		获取第n-1个元素：echo ${a[n]}
		获取全部元素：echo ${a[*]}或echo ${a[@]}
```

## 十二、RPM

```shell
1、查询命令
	1.1、基本语法
		rpm -qa 查询所安装的所有rpm软件包
	1.2、经验技巧
		由于软件包较多，一般都会采取过滤rpm -qa | grep rpm软件包
2、卸载命令
	2.1、基本语法
		rpm -e rpm软件包
		rpm -e --nodeps 软件包
	2.2、选项说明
		-e	卸载软件包
		--nodeps	卸载软件时，不检查依赖。这样的话，那些使用该软件包的软件在此之后就不能正常工作了
3、安装命令
	3.1、基本语法
		rpm -ivh rpm软件包
	3.2、选项说明
		-i	install，安装
		-v	--verbose，显示详细信息
		-h	--hash，进度条
		--nodeps	安装前不检查依赖
```

## 十三、YUM

```shell
1、基本语法
	yum [选项] [参数]
2、选项说明
	-y	对所有提问都回答"yes"
3、参数说明
	install	安装rpm软件包
	update	更新rpm软件包
	check-update	检查是否有可用的更新rpm软件包
	remove	删除指定的rpm软件包
	list	显示软件包信息
	clean	清理yum过期的缓存
	deplist	显示yum软件包的所有依赖关系
```

## 十四、tar

```shell
1、基本语法
	tar [选项] XXX.tar.gz 将要打包的内容
2、选项说明
	-c	产生.tar打包文件
	-v	显示详细信息
	-f	指定压缩后的文件名
	-z	打包同时压缩
	-x	解包.tar文件
	-C	解压到指定目录
3、压缩
	tar -zcvf 压缩后文件名.tar.gz 要打包的内容
4、解压
	tar -zxvf 要解压的内容 -C 想要解压到的目录
```

## 十五、shell的输入与输出

```shell
1、echo
	1.1、一般形式
		echo [参数] string
	1.2、参数
		-n	使用-n选项来禁止echo命令输出后的换行
		echo -n "What's your name:"
		-e	使用-e选项使转义字符生效
		echo -e "\007your home directory is $HOME"
2、read
	可以使用read语句从键盘或文件的某一行文本读入信息，并将其赋给一个变量。如果只指定了一个变量，那么read将会把所有的输入赋给该变量，直至遇到第一个文件结束符或回车
	2.1、一般形式
		read [参数] variable1 variable2 ……
		注意：(1)shell将空格作为变量之间的分隔符
			 (2)如果输入的文本域过长，shell将所有的超长部分赋予给最后一个变量
	2.2、参数
		-a	后跟一个变量，这个变量会被认为时数组，然后给其赋值，默认空格为分隔符。读取时使用echo ${arr[n]}
		-d	后跟一个标志符，其实只有其后的第一个字符有用，作为结束的标志
		-p	后面跟提示信息，即在输入前打印提示信息
		-e	在输入的时候可以使用命令补全功能
		-n	后跟一个数字，定义输入文本的长度。输入字符长度达到之后自动退出
		-r	屏蔽\，如果没有该选项，则\作为一个转义字符，有的话\就是个正常的字符
		-s	安静模式，在输入字符时不在屏幕上显示，例如login时输入密码
		-t	后面跟秒数，定义输入字符的等待时间
3、cat
	用来显示文件内容，创建文件，显示控制字符
	3.1、一般形式
		cat [参数] 文件1 文件2 ……
	3.2、参数
		-v	显示控制字符
		-n	显示文件行号
		-e	在文件中的行尾及两端之间的间隙之间显示$
		-T	将tab字符显示为^|
		-s	当遇到有连续两行以上的空白行，就替换为一行空白行
	3.3、一次显示多个文件，多个文件之间使用空格进行分隔。如cat file1 file2 ……
4、管道
	通过管道把一个命令的输出传递给另一个命令作为输入
	一般形式
		命令1 | 命令2
5、tee
	把输出的一个副本输送到标准输出，另一个副本拷贝到相应的文件中。
	5.1、一般形式：
		tee [参数] files
	5.2、参数
		-a	--append 附加到既有文件的后面，而非覆盖它
		-i	--ignore-interrupts 忽略中断信号
```

## 十六、定义变量

```shell
shell定义变量及类型
declare 和 typeset命令是等价的
	参数
		-r 将变量设置为只读变量
		-i 将变量设置为整数
		-a 将变量设置为数组
		-x 将变量设置为环境变量
```

## 十七、xargs

```shell
管道符与xargs的区别：管道符是将上一层的标准输出作为下一层的标准输入，而xargs是直接将上一层的标准输出作为下一层的参数
1、格式
	命令1 | xargs [参数] 命令2
2、参数
	-a file	从文件中读入作为标准输入
	-e/E flag	flag必须是一个以空格分隔的标志，当xargs分析到含有flag这个标志的时候就停止
	-p	当每次执行一个参数的时候询问一次用户是否执行后面的命令，y执行，n不执行
	-n num	后面加次数，表示结果在输出的时候一行输出n个
	-t	表示先打印命令，然后再执行
	-i/-I	将xargs的每项名称，一般是一行一行赋值给{}，可以用{}代替
	-r 当xargs的输入为空的时候就停止xargs，不用再去执行了
	-s num	命令行的最大字符数，指的是xargs后面那个命令的最大命令行数字符数
	-L num	从标准输入一次读取num行送给命令
	-d 分隔符	默认的分隔符是回车，参数的分隔符是空格，这里修改的是xargs的分隔符
	-x exit的意思，主要是配合-s使用
	--max-procs	xargs默认只用一个进程执行命令。如果命令要执行多次，必须等上一次执行完才能执行下一次。这个参数可以指定同时用多少个进程并行执行命令。--max-procs 2表示同时最多使用两个进程，--max-procs 0表示不限制进程数。
	-0	表示用null当做分隔符
使用案例：
1、将多行文件转单行文件输出
	cat test.txt | xargs
	a b c d e f g h i j k l m n o p q r s t u v w x y z
2、指定文件输出列数
	cat test.txt | xargs -n3
	a b c
    d e f
    g h i
    j k l
3、使用分隔符（-d指定分隔符X，-n指定将文件内容输出为2列）
	echo "nameXnameXnameXname" | xargs -dX -n2
	name name
	name name
4、将读取内容赋值给{}，传给后面命令调用(test中的内容为一列a、b、c、d)
	cat test | xargs -I {} echo "begin{}end"
	beginaend
	beginbend
	begincend
	begindend
```

## 十八、系统参数

```
$?:返回上一条命令的执行结果，若是正确则返回0，不正确则返回其他数
$0:返回文件名
$#:返回参数个数
$*:将输入参数视为一个整体，而不是多个个体
$@:将输入参数作为同一个字符串中的多个独立的单词
$1:输入的第一个参数
$n:输入的第n个参数
```

## 十九、标准输入、输出、错误

```
命令 1> 文件:标准输出，将命令正确的输出结果输入至文件中
命令 2> 文件:标准错误，将命令错误的输出结果输入至文件中
命令 &> 文件:命令生成的所有结果都输入至文件中，相较于标准输出，bash shell会自动赋予错误消息更高的优先级
echo "输出信息" >&2:有意将输出信息转化为标准错误信息
exec 0< 文件:从文件中读取数据
echo 3>&-:要关闭文件描述符，将他重定向到特殊符号&-
命令 > /dev/null:将命令的输出结果输出至null中，类似于垃圾桶，输出至这个文件中的数据都会被丢掉

```

## 二十、Ctrl+z、Ctrl+c

```
Ctrl+z：中止程序
Ctrl+c：终止程序
jobs -l:查看中止的程序
jobs -n:只列出上次shell发出通知后改变了状态的作业
jobs -p:只列出作业的PID
jobs -r:只列出运行中的作业
jobs -s:只列出已停止的作业
fg 作业号：前台继续执行作业
bg 作业号：后台继续执行作业
```

## 二十一、库函数

```
1、创建库文件
	和普通文件一样，在文件中编写函数，方便被别的文件引用
	$ cat myfuncs 
	# my script functions 
	function addem { 
 		echo $[ $1 + $2 ] 
	} 
	function multem { 
	 	echo $[ $1 * $2 ] 
	} 
	function divem { 
	 if [ $2 -ne 0 ] 
	 then 
	 	echo $[ $1 / $2 ] 
	 else 
	 	echo -1 
	 fi 
	} 
2、在别的脚本文件中引用库文件
	使用库函数的关键在于source命令，source命令会在当前shell上下文中执行命令，source命令有个快捷的别名，称作点操作符。假定库文件和shell脚本位于同一目录，只需在shell脚本中添加
	. ./库文件
	例子：
	#!/bin/bash 
	# using functions defined in a library file 
	. ./myfuncs 
	value1=10 
	value2=5 
	result1=$(addem $value1 $value2) 
	result2=$(multem $value1 $value2) 
	result3=$(divem $value1 $value2) 
	echo "The result of adding them is: $result1" 
	echo "The result of multiplying them is: $result2" 
	echo "The result of dividing them is: $result3" 
	$
```

## 二十二、文件传输

```shell
1、rsync
本地同步
	初次同步时, rsync会全量拷贝从源文件或目录到目标位置. 第二次往后同步时, rsync 仅仅会拷贝变化的数据块或字节到目标位置这将使得文件传输非常迅速
	rsync 可以使用ssh协议加密传输
	rsync 在发送时会压缩数据块, 接收后再解压缩数据块. 所以和其他文件传输协议比起来, rsync在跨主机传输文件时会占用较小的带宽
	源和目标既可以在本地也可以在远端. 如果是远端的话,需要指明登录用户名, 远端服务器名, 和远端文件或目录. 同时源可以是多个, 目标位置只能是一个。
	参数：
		-r 表示递归，即包含子目录
		eg：rsync -r source destination
		解释：source目录表示源目录，destination表示目标目录，执行命令之后，目标目录下就会出现destination/source这个子目录。
		eg：rsync -r source1 source2 destination
		解释：执行命令后，source1、source2都会被同步到destination目录。
		-a -a参数可以替代-r，除了可以递归同步以外，还可以同步元信息（比如修改时间、权限等）。由于 rsync 默认使用文件大小和修改时间决定文件是否需要更新，所以-a比-r更有用
		eg：rsync -a source destination
		解释：目标目录destination如果不存在，rsync会自动创建。执行上面的命令后，源目录source被完整得复制到目标目录destination下面，形成destination/source的目录结构。
		eg：rsync -a source/ destination
		解释：如果只想同步源目录source里面的内容到目标目录destination，则需要在源目录后面加上'/'。执行上面命令后，source目录里面的内容，就都复制到了destination目录里面，并不会在destination下面创建一个source子目录。
		-n 模拟命令执行的结果，并不真的执行命令
		-v 将同步结果输出到终端，这样就可以看到哪些内容会被同步
		--delete 默认情况下，rsync只能确保源目录的所有内容（明确排除的文件除外）都复制到目标目录。它并不会使两个目录保持相同，并且不会删除文件。如果要使目标目录成为原目录的镜像副本，则必须使用--delete参数，这将删除只存在于目标目录而不存在于源目录的文件
		--exclude 同步时排除某些文件或目录，这时可以使用--exclude参数指定排除模式。如果需要排除多个模式，可以使用多个--exclude参数（注：rsync会同步以“点”开头的隐藏文件）
		eg：rsync -av --exclude 'dir1/*' source/ destination
		解释：执行命令，只同步source目录下除去dir1的所有目录文件
		eg：rsync -av --exclude 'file1.txt' --exclude 'file2.txt' source/ destination
		解释：多个排除模式，可以使用多个--exclude参数
		eg：rsync -av --exclude={'file1.txt','file2.txt'} source/ destination
		解释：多排除模式也可以使用bash的大括号的扩展功能，只用一个--exclude参数
		eg：rsync -av --exclude-from='exclude-file.txt' source/ destination
		解释：如果排除模式很多，可以将他们写入一个文件，每个模式一行，然后用--exclude-from参数指定这个文件。
		--include 指定必须同步的文件模式
		eg：rsync -av --include "*.txt" --exclude "*" source/ destination
		解释：同步时排除所有文件，但是会同步txt文件

远程同步
	rsync [参数] source/ username@host:/destination
```

## 二十三、字符串截取

```shell
一、${str:开始索引:截取长度}
1.1、解释
	① str为需要截取的字符串
	② 索引从0开始
	③ 截取长度可不写，不写则为从开始索引至结尾
1.2、示例
	str="Hello World!"
	
	#从第7个字符开始，截取5个字符
	echo ${str:6:5} #输出World
	
	#从第7个字符开始，截取到末尾
	echo ${str:6} #输World!
	
	#从倒数第6个字符开始截取，至末尾
	echo ${str: -6} #输出World!（-和:之间要有一个空格）
二、从开头或结尾删除
2.1、删除左边（前缀），保留右边
2.1.1、解释
	${str#pattern} #删除最短匹配
	${str##pattern} #删除最长匹配
2.1.2、示例
	str="path/to/file.txt"
	echo ${str#*/} #输出to/file.txt（删除匹配到的第一个pattern）
	echo ${str##*/} #输出file.txt（删除最后一个匹配到的pattern）
2.2、删除右边（后缀），保留左边
2.2.1、解释
	${str%*/} #删除最短匹配
	${str%%*/} #删除最长匹配
2.2.2、示例
	str="path/to/file.txt"
	echo ${str%/*} #输出path/to（删除匹配到的第一个pattern）
	echo ${str%%/*} #输出path（删除最后一个匹配到的pattern）
```







































