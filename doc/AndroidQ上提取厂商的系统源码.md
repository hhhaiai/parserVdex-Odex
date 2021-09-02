# Android Q 上提取厂商的系统源码


>   原文地址[[czlee's blog](https://blog.czlee.cn/)](https://blog.czlee.cn/archives/androidq%E4%B8%8A%E6%8F%90%E5%8F%96%E5%8E%82%E5%95%86%E7%9A%84%E7%B3%BB%E7%BB%9F%E6%BA%90%E7%A0%81)

在软件开发过程中，我们有些时候需要了解厂商对framework层的改动，以进行适配工作，可能需要拿到系统framework层的源码，下面以华为最新的10.1系统为例提取系统源码：

1. 首先从android studio的device file explorer导出/system/framework目录到你电脑的某个目录，我这里为：/home/czlee/文档/hw10.1/。

![image-20210902165255733](https://raw.githubusercontent.com/hhhaiai/Picture/main/img/202109021653569.png)

`framework.jar`是个压缩包，里面包含了部分系统源码的`dex`文件，查看方式如下：

2.   安装`jeb`反编译神器
    + [jeb 3.7 下载地址](https://blog.czlee.cn/upload/2020/06/jeb-demo-3.7.0.201909272058-cracked-3362f6666e0a472b8493c3f1eb2701c1.rar)
     下载之后，解压该压缩包，将得到三个文件：`jeb***.zip`，`jeb.jar`，`Keygen.class`
        该软件应该是收费的，破解教程如下：

        -   将解压之后的`zip`压缩包再次解压到`jeb3.7`目录下，将解压的`jeb.jar`替换掉`jeb3.7`目录下的`bin/app/jeb.jar`文件。

        -   运行`jeb3.7`的脚本文件，`linux`下是`jeb_linux.sh`，`windows`下是`jeb_wincon.bat`，依次类推。

        -   如果未激活，会提示激活，点击弹出界面的`Manual Generation`按钮，会弹出注册码页面。

        -   打开控制台，进入`Keygen.class`所在目录，在命令行运行`java Keygen`，会输出注册码，比如：

        ```shell
            2844195934392344985
            0	
            key:8997933301527419414Z9168951233
        ```

         将`key`的值复制到`jeb`注册码页面里即可激活。

    + 将`framework.jar`直接拖入`jeb`，效果如下：

![image-20210902165555849](https://raw.githubusercontent.com/hhhaiai/Picture/main/img/202109021655879.png)

上面的反编译的代码是`smali`语法，不太利于我们阅读，我们可以选中代码标签页，使用菜单栏的`行为`->`解析`将代码解析成`java`代码。



![image-20210902165616029](https://raw.githubusercontent.com/hhhaiai/Picture/main/img/202109021656050.png)

1.  到此我们可以查看`framework.jar`的代码了。

2.  `framework.jar`并不包含`xxService`的代码，比如：`ActivityManagerService`，这块的代码可以在`oat/arm64/`下找到，比如：`services.vdex`和`services.odex`，当然还有厂商自定义服务的其它文件，如：`hwServices.odex`，`hwServices.vdex`等文件，这里以`services`这个前缀的文件为例，反编译其代码：

    -   `vdex`应该是`android o`引入的，我们可以使用`vdexExtractor`进行`dex`提取。
        [vdexExtractor@github 下载](https://github.com/anestisb/vdexExtractor)
        该库更新的较慢，不确定是否已经支持`Android Q`，我基于该库做了`Android Q`的适配：
        [vdexExtractor@fixQ 下载](https://blog.czlee.cn/upload/2020/06/vdexExtractor-a77ef7c991f84ed7b36812050fe05b2b.zip)
        该库需要在`Linux`下使用，可以使用`Windows`的`Linux 子系统`，以`/home/czlee/文档/hw10.1/`为例，使用方法如下：

    ```shell
    	cd vdexExtractor/tools/deodex
    	./run.sh -i /home/czlee/文档/hw10.1/oat/arm64 -o /home/czlee/文档/hw10.1/oat/arm64
    ```

    执行结果会输出到`/home/czlee/文档/hw10.1/oat/arm64/vdexExtractor_deodexed`目录，我们可以在该目录查看生成的`dex`文件，`dex`文件可以使用拖入`jadx`或上面的`jeb`查看。[jadx@github 下载](https://github.com/skylot/jadx/releases/)

    -   ```
        odex
        ```

         

        文件的提取需要使用

        ```
        baksmali
        ```

        和

        ```
        /apex/com.android.runtime
        ```

        -   [baksmali 下载地址](https://bitbucket.org/JesusFreke/smali/downloads/)
            将`baksmali-2.4.0.jar`和`smali-2.4.0.jar`下载之后，放到`smali`目录
        -   `/apex/com.android.runtime`是 `Android Q`引入的运行库的安装包，我们可以从`/system/apex/com.android.runtime.release.apex`拿到。

![image-20210902165640995](https://raw.githubusercontent.com/hhhaiai/Picture/main/img/202109021656023.png)

将上面的文件导出后，由于`apex`其实就是一种压缩包，直接解压即可，解压之后的目录结构如下：

![image-20210902165706225](https://raw.githubusercontent.com/hhhaiai/Picture/main/img/202109021657258.png)

这里有个`apex_payload.img`，它是 ext2 文件，使用 ubuntu 可以直接双击挂载。

![image-20210902165725154](https://raw.githubusercontent.com/hhhaiai/Picture/main/img/202109021657184.png)

-   我这里解压之后放置到`/home/czlee/文档/smali/com.android.runtime`下，创建软连接，模拟`android`系统的挂载路径。

```shell
	# 模拟 /apex/com.android.runtime 挂载路径
	sudo mkdir /apex/
	sudo ln -s /home/czlee/文档/smali/com.android.runtime /apex/

	#  模拟 /system/framework 挂载路径
	sudo mkdir /system/
	sudo ln -s /home/czlee/文档/hw10.1 /system/framework 
```

这里以反编译`services.odex`为例：

```shell
	cd smali
	# 建立软连接
	ln -s /home/czlee/文档/hw10.1 framework
	# 运行反编译程序，反编译成 smali
	java -jar baksmali-2.4.0.jar x framework/oat/arm64/services.odex -o services
	# 运行打包程序，打包成 dex 文件
	java -jar smali-2.4.0.jar a services/ -o services.dex
```

到此`services.odex`提取完成，直接拖入 `jadx` 查看即可。
