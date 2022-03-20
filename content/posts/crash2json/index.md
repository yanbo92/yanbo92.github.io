---
title: 使用crash2json库将iOS崩溃日志解析成json文件
date: 2021-12-28
tags:
- iOS
- python
- crash
categories: 
- iOS
image: "cover.png"
url: "crash2json"
---




## 项目背景

- 为了对iOS崩溃日志进行进一步的分析，需要将崩溃日志中的信息拆分成不同的部分，取其中的一部分或者多个部分进行存储、对比，于是我写了一个python库将`.crash`文件转换成`.json`文件。
- 鉴于Apple在iOS15上已经将崩溃文件存储成类似json的格式，本库仅在iOS15以下的版本发挥作用。



## 操作步骤



#### 安装crash2json

```shell
pip install crash2json
```





#### 命令行直接运行

```shell
crash2json yourcrashreport.crash
```





#### 其他参数

```dockerfile
positional arguments:
  crash_file

optional arguments:
  -h, --help            show this help message and exit
  --binary_image_list_only
                        parse binary_image_list to json only
  --crashed_thread_state_only
                        parse crashed_thread_state to json only
  --diagnostic_messages_only
                        parse diagnostic_messages to json only
  --exception_backtrace_only
                        parse exception_backtrace to json only
  --exception_information_only
                        parse exception_information to json only
  --header_only         parse header to json only
  --other_threads_backtrace_only
                        parse other_threads_backtrace to json only
  --thread0_backtrace_only
                        parse thread0_backtrace to json only
  -s, --simple          output a simple json with only header, exceptionInfo, diagnositcMsg, Thread0Backtrace
  -o OUTPUT_NAME, --output_name OUTPUT_NAME
                        the .json file you want to save result to, no need .json suffix
```



## 源码地址

[yanbo92/crash2json](https://github.com/yanbo92/crash2json)





