视频链接：https://www.linkedin.com/learning/web-servers-and-apis-using-c-plus-plus/building-crow?autoplay=true
# 0. 介绍
## 1. 目标：
如何使用MongoDB作为数据库，eroku作为云服务器创建网站。
## 2. 所需技能
* c++中级知识
* Docker
* 熟悉使用命令行bash
# 1. 安装工具
## 1. 安装Docker
https://docs.docker.com/desktop/install/windows-install/
安装后遇到问题：需更新WSL，按提示安装最新版本运行即可，https://learn.microsoft.com/en-us/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package
## 2. 安装c++编辑器
视频用Atom，我用Vscode
## 3. 创建Docker文件
见`cppweb\cppbox\Dockerfile`
用 `#` 注释
错误：注释只能注释一整行，在命令行后加#,注释不起作用 do
## 4. 运行Docker文件
### 运行命令
```
cd cppbox
docker build -t cppbox .
```
-t: 标记选项
图像标记为cppbox
### 调试过程
#### error1：apt-get -qq update
* 错误：
```
ERROR [2/6] RUN apt-get -qq update
#0 7.914 W: The repository 'http://security.debian.org/debian-security stretch/updates Release' does not have a Release file.
#0 7.914 W: The repository 'http://deb.debian.org/debian stretch Release' does not have a Release file.
#0 7.914 W: The repository 'http://deb.debian.org/debian stretch-updates Release' does not have a Release file.   
#0 7.914 E: Failed to fetch http://security.debian.org/debian-security/dists/stretch/updates/main/binary-amd64/Packages  404  Not Found
#0 7.914 E: Failed to fetch http://deb.debian.org/debian/dists/stretch/main/binary-amd64/Packages  404  Not Found 
#0 7.914 E: Failed to fetch http://deb.debian.org/debian/dists/stretch-updates/main/binary-amd64/Packages  404  Not Found
#0 7.914 E: Some index files failed to download. They have been ignored, or old ones used instead.
```
* 解决办法：
把gcc版本从`7.3.0`改新一点:`12.2.0`
#### error2:安装libboost-all-dev
* 错误
```
[5/6] RUN apt-get -qq install libboost-all-dev=1.62.0.1:
#0 1.455 E: Version '1.62.0.1' for 'libboost-all-dev' was not found
```
* 解决办法：
删除版本号，即ls -l /var/lib/dpkg/info | grep -i`RUN apt-get -qq install libboost-all-dev
`
* 新的错误
```
Errors were encountered while processing:
#0 193.9 gfortran
#0 193.9 libcoarrays-dev:amd64
#0 193.9 libcoarrays-openmpi-dev:amd64
#0 193.9 E: Sub-process /usr/bin/dpkg returned an error code (1)
```
* 解决办法
`RUN apt-get install -y libboost-all-dev --no-install-recommends`
#### error3: Unable to locate package built-essential
* 错误
```
#0 1.145 E: Unable to locate package built-essential
```
* 解决办法：
built拼错,改正。
### 查看libboost库是否安装成功
在容器里运行bash
`docker run -ti cppbox:latest bash`
查看所有libboost库
`find /usr -name libboost*.*`
## 5. 添加Volume
* 为了编辑Docker容器上的文件，所以添加卷。
* 卷是主机和1个或多个docker容器共享的目录。
* 添加卷的命令
` docker run -v <host>:<container> -ti <image> bash`
`-v`:创建卷的选项
`<host>`:主机目录的绝对路径
`<container>`:docker容器的绝对路径
`image`:要运行的镜像的名字
本project:
```
docker run -v C:/Users/Auly/Desktop/cppweb:/usr/src/cppweb -ti cppbox:latest bash
```
-ti：把docker置于终端交互模式; cppbox:之前创建的cppbox镜像的本地版本；latest bash:置于容器的最新的bash shell中
* 切换目录到：
```
cd /usr/src/cppweb
```
* 创建MY-FILE.txt
`touch MY-FILE.txt`
并查看
`ls`
。
## 6. 创建Crow
### 下载Crow
Crow:c++ web微框架。
下载crow_all.h，用最新版本：https://github.com/CrowCpp/Crow
### 创建hello_crow
* 在cppweb里创建hello_crow
`mkdir hello_crow`
* 切换目录到hello_crow
```
cd hello_crow
code .
```
* 创建main.cpp
* 创建CMakeLists.txt
### 链接
* 建build目录
`mkdir build`
* 切换目录
`cd build`
* 在容器里使用cmake
```
cmake ..
make
```
## 7. 在容器中运行服务器
* 查服务器端口

在容器bulid目录里，
```
./hello_crow
```  
![picture 2](../images/b47b528f8c30f34ea14dd831bd7d36843644f0d5549344f2d2ce0a8ca4d3409a.png)  

可知，服务器正是用端口18080
*
在浏览器输`localhost:18080`，无法访问网站。
Why?
默认情况下，每个容器都是隔离的，它的所有端口都未打开。为了访问服务器，需打开一个端口并告诉服务器使用哪个端口。
* 打开一个端口
退出容器
```
exit
```
```
docker run -v <host>:<container> -p <host port>:<container port> -e PORT=8081 <image> <app to run>
```
-p: 打开一个端口，冒号左边是主机的端口号，冒号右边是容器的端口号。
-e: 创建一个环境变量
本project:
```
docker run -v C:/Users/Auly/Desktop/cppweb:/usr/src/cppweb -p 8080:8080 -e PORT=8080 cppbox:latest /usr/src/cppweb/hello_crow/build/hello_crow
```
\* error:
```bash
docker: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "C:/Program Files/Git/usr/src/cppweb/hello_crow/build/hello_crow": stat C:/Program Files/Git/usr/src/cppweb/hello_crow/build/hello_crow: no such file or directory: unknown.
```
指定的可执行文件在容器内无法找到。
\* 解决办法
在容器的绝对路径上加单引号，并在路径前加反斜杠\，防止它变成windows路径。
```
docker run -v C:/Users/Auly/Desktop/cppweb:'\/usr/src/cppweb' -p 8080:8080 -e PORT=8080 cppbox:latest '\/usr/src/cppweb/hello_crow/build/hello_crow'
```
![image.png 1](../images/6e1d1ec8ea6a5c86a9b18a32cf582b400bc365cc7a3b38b6b056aba4bc9b1674.png)  
得到端口号=8080。
* 显示网页

回到浏览器，输：
```
localhost:8080
```
显示网页
![image.png 1](../images/e0eb63c98d89ab96f705b780f7736adfb82b3d9171751854f30bef222e7bc9dd.png)  
乌鸦网络服务器正在运行🎉
## 8. 练习:更改示例网页
改为"hello,\<your name>"
![image.png 1](../images/b7679601c19e3b2948a6b5c2bfe023b4391334ef9509e36041b6173f1d5e6920.png)
## 9. 解决方案
1. 修改main.cpp
2. 用make重新编译和链接服务器
用5中的命令到bash，切换到hello_crow，用6链接命令。
3. 在容器中运行hello_crow,重启服务器
用7中打开端口命令