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
在开发环境下，还能手工设置系统变量GOPATH，编译环境就不能手工添加每个项目的地址。所以需要**项目根目录**下添加build脚本文件

**windows环境**build.bat

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

**mac os环境**build.sh

```shell
#!/bin/zsh

curpath=${PWD}
oldgopath=${GOPATH}
oldgobin=${GOBIN}

export GOPATH=${oldgopath}:${curpath}
export GOBIN=${curpath}/bin

#echo $GOPATH
#echo $GOBIN

go clean
go install demo
cp -rf src/config bin

export GOPATH=${oldgopath}
export GOBIN=${oldgobin}

echo install finished
```

## 设置目录结构

```text
demo
│  build.bat
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

## Go命令

+ go build : 加上可以编译的go源文件编译生成一个可执行文件，放在为当前路径
+ go install : go build + 把编译后的可执行文件放到GOPATH/bin目录下
+ go get : 从指定源上面下载或者更新指定的代码和依赖，并对他们进行编译和安装。git clone + go install

## VSCode中Go Debug配置

VSCode Debug时会自动生成launch.json文件，需修改其中的一些参数如下

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch",
      "type": "go",
      "request": "launch",
      "mode": "debug",
      "remotePath": "",
      "port": 2345,
      "host": "127.0.0.1",
      //指定debug时的启动入口路径
      "program": "${workspaceRoot}/src/demo",
      "env": {},
      "args":[],
      "showLog": true,
      "cwd": "${workspaceRoot}/src"
    }
  ]
}
```

代码中如果涉及到读取相对路径文件等功能，如

```go
func readConfig() string {
	configPath := "config/logconfig.xml"
	file, err := ioutil.ReadFile(configPath)
	if err != nil {
		fmt.Println("read config file error", err)
		panic(err.Error())
	}
	return string(file)
}
```

就需在配置中添加选项```"cwd": "${workspaceRoot}/src"```指定源代码的当前工作路径(cwd是current working directory的缩写)，否则默认取启动入口文件所在的路径做为工作路径，进而导致报错

```text
read config file error open config/logconfig.xml: no such file or directory
```

配置文件中更多选项信息详见[官方文档](https://code.visualstudio.com/docs/editor/debugging#_launchjson-attributes)

