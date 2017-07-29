gip要解决什么问题？
===

1. go项目的版本依赖，希望所有的开发、测试、生产环境跑的版本都是相同的；
2. 虽然go的vendor机制能够解决#1中的问题，但是会带来另一个问题：开发过程中如果有两个项目，A和B，A依赖B，但是，我们需要经常改B，所以又不想把B放到A的vendor下；那么如果A和B里面都有一个vendor的时候，就有可能会出现一些诡异的编译错误；参考 <https://github.com/akutz/gpd>
3. 在中国不翻墙就能把依赖拉下来

现有方案的一些问题
===

1. [glide](https://github.com/Masterminds/glide)：Glide可以解决#1和#3的问题，但是解决不了#2的问题；特别是当需要同时修改多个代码库的时候就会很痛苦；
2. [vg](https://github.com/GetStream/vg) + [dep](https://github.com/golang/dep): 可以解决#1和#2的问题，但是解决不了#3的问题；因为每次```dep ensure```的时候都需要去获取依赖的meta信息；
3. [gigo](https://github.com/LyricalSecurity/gigo)，解决#1的问题

gip怎么做
===

参考了 [vg](https://github.com/GetStream/vg) 和 [gigo](https://github.com/LyricalSecurity/gigo) 的思路，gip想要实现的是python中 [virtualenv](https://virtualenv.pypa.io/en/stable/) + [pip](https://pypi.python.org/pypi/pip) 的组合。

1. 每次进入一个项目，自动激活这个项目的GOPATH
2. 有一个requirements.txt，记录依赖的go package以及这些package对应的下载地址（现在只支持git）
3. 提供freeze方法（类似 ```pip freeze```），输出现有的依赖库以及版本

使用方法
===

1. 初始化一个项目

	```
	# Just initialize a new GOPATH
	gip init $name
	```

2. 正常开发

	```
	# install the requirements first
	gip install requirements.txt
	
	# use normal gip to install packages
	gip get $pkg
	# or specify the git clone url
	gip get github.com/golang/net#master,golang.org/x/net
	
	# coding, coding, coding
	
	# before push, generate the requirements.txt and commit it
	gip freeze > requirements.txt
	```