# Go项目目录

## Go项目

项目中会有如下三个目录

```text
./bin
./pkg
./src
```

其中，bin存放编译后的可执行文件；pkg存放编译后的包文件；src存放项目源文件。
一般，bin和pkg目录可以不创建，go命令会自动创建，只需要创建src目录即可。

## 设置GOPATH

Go项目的地址需添加到GOPATH中，通过分号```;```连接。import package时，Go会遍历GOPATH中的每个地址下的src目录，判断package是否存在，编译时同样如此。
在开发环境下，还能手工设置系统变量GOPATH，编译环境就不能如此手工添加每个项目的地址。所以需要**项目根目录**下添加install 文件

**windows环境**install.bat

```bat
@echo off
setlocal

if not exist install.bat (
  echo install.bat must be run from its folder
  exit
)

set GOPATH=%cd%
set GOBIN=%GOPATH%\bin

echo set GOPATH=%GOPATH% success
echo set GOBIN=%GOBIN% success

go clean
go install demo

echo install finished
```

## 设置目录结构

```text
demo
│  install.bat
├─bin
│      demo.exe
├─pkg
│  └─windows_amd64
│          utility.a
└─src
    ├─demo
    │      main.go
    └─utility
            utility.go
```

+ utility.go中的**package**名称最好和目录**utility**一致，而文件名可以随意。 如果不一致，生成的.a文件和目录名一致，导致在import时需填写目录名，而引用包时则需要包名。例如：目录为utility，包名为util，则生成的静态包文件是utility.a，引用该包:import "utility"，使用包中成员：util.print()
+ 生成的可执行文件的名称是取自main.go的目录名，所以这里建议为项目名
+ main.go中设置**package main**，文件名建议为**main.go**
