# TCP端口扫描
### 一、TCP connect()扫描原理是：
扫描主机通过TCP/IP协议的三次握手与目标主机的指定端口建立一次完整的连接。连接由系统调用connect开始。如果端口开放，则连接将建立成功；否则，若返回-1则表示端口关闭。
### 二、TCP connect()扫描要求是：
（1）输入目的IP地址以及端口范围         
（2）设置获取的用户输入IP地址为远程IP地址；         
（3）从开始端口到结束端口依次扫描，每扫描一个端口创建一个新的套接字；         
（4）设置远程地址信息中的端口号为需要扫描的当前端口号；         （5）连接到当前端口号的目的地址；         
（6）若连接成功（ connect（）函数返回0 ）则输出该端口为开启状态，否则输出该端口为关闭状态；         
（7）关闭当前套接字。
### 三、实验代码：
```
import time, sys 
import queue 
import optparse
import socket
import threading as th    # 导入threading包
lock = threading.Lock()
openNum = 0
threads = []
#测试当前主机和端口是否开放，直接使用socket连接
def portScanner(host,port):
    global openNum
    try:
        s = socket(AF_INET,SOCK_STREAM)
        s.connect((host,port))
        lock.acquire()
        openNum+=1
        print('[+] %d open' % port)
        lock.release()
        s.close()
    except:
        pass

def main():
    setdefaulttimeout(1)
    IP=raw_input('Input IP :')
    PORT=raw_input('Input PORT:')
    list = PORT.split(",")
    for i in range(len(list)):
        if list[i].isdigit():
            t = threading.Thread(target=portScanner, args=(IP, int(list[i])))
            threads.append(t)
            t.start()
        else:
            newlist = list[i].split("-")
            startPort = int(newlist[0])
            endPort = int(newlist[1])
            for p in range(startPort, endPort):
                t = threading.Thread(target=portScanner, args=(IP, p))
                threads.append(t)
                t.start()

    for t in threads:
        t.join()

    print('[*] The scan is complete!')
    print('[*] A total of %d open port ' % (openNum))



if __name__ == '__main__':
    main()

```
### 四、实验结果
对百度：119.75.217.26
```
[+]21 open
[+]80 open
[+]443 open
[*]The scan is complete1!
[*]A total of 3 open port
```
### 五、分析总结
扫描百度的ip，扫描到百度开放的443端口，443是基于 TLS/SSL 的网页浏览端口，能提供加密和通过安全端口传输的另一种 HTTP。


