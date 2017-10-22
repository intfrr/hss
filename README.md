# hss

[![Build Status](https://travis-ci.org/six-ddc/hss.svg?branch=master)](https://travis-ci.org/six-ddc/hss)

## 概述

hss是一款可交互式的ssh批量执行命令的客户端，工具基于[libreadline](https://cnswww.cns.cwru.edu/php/chet/readline/rltop.html)实现，使你能像操作bash一样地输入需要执行的命令，同时也支持命令搜索，历史命令纪录等。并且工具支持在输入命令时，按一下`tab`键，即可根据远程服务器的信息，进行文件路径和执行命令补全。另外命令的执行是异步的，无需等待一台机器命令执行完成才执行下一台的ssh操作，可支持同时操作数百台服务器。

hss还支持插件扩展，可通过`Esc`键将运行模式从`remote`切换到`inner`，在这里可处理一些批量操作：动态增加删除机器、设置程序运行时的配置等，更多的有趣的功能可能将在后续版本逐渐添加。

## 预览

```
Usage: hss [-f hostfile] [-o file] [-u username] [command]

Options:
  -f, --file=FILE           file with the list of hosts or - for stdin
  -H, --host                specifies a host option, support the same options as the ssh command
  -i, --identity-file=FILE  specifies a default identity (private key) authentication file
  -u, --user                the default user name to use when connecting to the remote server
  -t, --conn-timeout        ssh connect timeout (default 30 sec)
  -o, --output=FILE         write remote command output to a file
  -v, --verbose             be more verbose (i.e. show ssh command)
  -V, --version             show program version
  -h, --help                display this message
```

* 使用效果图如下：

![](https://github.com/six-ddc/hss/blob/master/demo.gif?raw=true)

## 安装

* 安装依赖

```bash
## on MacOS
brew install readline

## on CentOS
yum install readline-devel

## on Ubuntu / Debian 
apt-get install libreadline6-dev
```

* 编译&安装

```bash
make && make install
```

* 或者直接下载[Release文件](https://github.com/six-ddc/hss/releases)

## 指南

hss的实现原理是对每个host，直接调用本地的`ssh`命令去执行服务器操作，然后再通过进程间通信将执行结果返回给终端。

故此hss支持所有的`ssh`命令的参数选项。以下是hostfile示例文件：

```
192.168.1.1
-p 2222 root@192.168.1.2
-p 2222 -i ~/.ssh/identity_file root@192.168.1.3
-p 2222 -oConnectTimeout=3 root@192.168.1.4
```

连接上述机器的命令如下：

```
# 指定配置文件的方式
hss -f hostfile

# 管道方式，这里必须指定需要执行的命令
cat hostfile | hss -f - 'date'

# 通过传参的方式
hss -H '192.168.1.1' -H '-p 2222 root@192.168.1.2' -H '-p 2222 -i ~/.ssh/identity_file root@192.168.1.3' -H '-p 2222 -oConnectTimeout=3 root@192.168.1.4'
```

hss命令本身也可以携带一些简单的参数，这些参数将作为每一个host的默认值，比如指定了`-t conn-timeout`，那么对于没有配置超时时间的，将用该值作为超时设置。

### inner模式

通过`Esc`可将运行模式从默认的`remote`切换到`inner`，inner模式下支持的命令都是程序内部实现的（可参考[command目录](https://github.com/six-ddc/hss/tree/master/command)），目前支持以下几种：

* help

    列出inner命令列表

* config

    ```
    Usage: config <command>

    Commands:
      get all|<config> : get config
      set <config> [value] : set config

    Config:
      output <filename> : redirect output to a file. stdout is used if filename is '-'
      conn-timeout <filename>  : ssh connect timeout
    ```

    配置管理，可get/set程序运行的一些配置，比如可通过`config set output a.txt`，将后面remote模式下的命令执行结果都重定向输出到a.txt文件中，需要重新输出到终端，则使用`config set output -`复原

* host

    ```
    Usage: host <command>

    Commands:
      list : list all ssh slots
      add <ssh_options> : add a ssh slot
      del <ssh_host> : delete special ssh slot
    ```

    host管理，可动态增加或删除需要连接的远程host

