# windows subsystem linux 编译JDK

## 源码下载页面
https://jdk.java.net/java-se-ri/11
a single [zip file](https://download.java.net/openjdk/jdk11/ri/openjdk-11+28_src.zip)(sha256) 178.1 MB。

我们也可以去[github](https://github.com/openjdk/jdk)下载,不同的版本使用不同`tag`,同时也请注意阅读[building](https://github.com/openjdk/jdk/blob/master/doc/building.md)文档


## 软件环境准备
（bootstrap JDK、gcc等编译器、make工具、autoconf工具等

## 编译配置
```
sh ./configure \
 --disable-warnings-as-errors \
 --with-target-bits=64 \
 --with-boot-jdk=/opt/jdk-11.0.16 \
 --with-debug-level=slowdebug \
 --with-native-debug-symbols=internal \
 --enable-ccache
```

```sh
make all
```

## VSCODE 使用自己编译的jdk跑java代码`setting.json`
```json
{
    "java.project.sourcePaths": ["src"], // java源目录
    "java.project.outputPath": "bin",   //java编译后的目录
    "java.project.referencedLibraries": [
        "lib/**/*.jar"
    ],
    "java.configuration.runtimes": [
        {
            "name": "JavaSE-11",
            "path": "/home/lws/jvm/openjdk11/build/linux-x86_64-normal-server-slowdebug/images/jdk", //自己编译出来的jdk
            "sources" : "/home/lws/jvm/openjdk11/src/java.base/share/classes", // 要关联的java源码
            "default":  true
        }
    ]
}
```

## VSCODE 配置调试hotspot C/C++源码`launch.json`
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "cppdbg",
            "name": "gdb hotspot",
            "request": "launch",
            "program": "/home/lws/jvm/openjdk11/build/linux-x86_64-normal-server-slowdebug/jdk/bin/java", //gdb要调试的程序
            "args": ["HelloWorld"], //要运行调试的java的.class文件
            "stopAtEntry": true,  // 停留在main方法
            "cwd": "${workspaceFolder}",
            "environment": [
                {"name": "JAVA_HOME", "value": "/home/lws/jvm/openjdk11/build/linux-x86_64-normal-server-slowdebug/jdk"},
                {"name": "CLASSPATH","value": "/home/lws/Desktop/code/jvm"}
            ],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        },
        // remote debug
        {
            "type": "cppdbg",
            "name": "(gdb) Attach",
            "request": "attach",
            "program": "/home/lws/jvm/openjdk11/build/linux-x86_64-normal-server-slowdebug/jdk/bin/java",
            "processId": "${process id to attach to}", // jps 查看出来的进程id
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```

[参考视频](https://www.bilibili.com/video/BV16e4y1S7mC/)


