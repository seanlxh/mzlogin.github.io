---
layout: post
title: 初步研究nginx代码
categories: nginx
description: nginx代码的调试与复现
keywords: nginx, 源码
---

---
## 初步研究nginx代码

---

### 使用vscode调试nginx代码
我是使用MAC机器编译的，在nginx官网下载好源码之后，使用VS Code打开源码
<div align="center"><img width="350px" height="300px" src="https://seanlxh.github.io/images/posts/nginx/vscode.jpg"/></div>
打开控制台，在nginx工程根目录下
1. 修改/auto/cc/conf文件，将ngx_complie_opt 改为“-c -g”
2. 执行 sudo ./auto/configure --prefix=（nginx编译目标目录）注意地址后面不能加‘/’否则后面路径会有两个//从而找不到conf文件
3. 执行sudo make
4. 进入编译好的目标目录，执行./sbin/nginx 此时打开浏览器访问127.0.0.1 同时ps -ef | grep nginx可以看到worker与master即可。
5. 在vscode中新建一个configuration，配置好相应的地址即可开始编译。
 
```C++
for (int i = 1; i < 3; ++i) {
        result = fork();
        if(result == 0){
            startWorker(i,serverSock);
            printf("start worker %d\n",i);
            break;
        }
    }

void startWorker(int workerId , int serverSock)
{
    int kqueuefd = kqueue();
    struct kevent change_list[1];
    struct kevent event_list[1];

    EV_SET(&change_list[0] , serverSock, EVFILT_READ, EV_ADD|EV_ENABLE,0,0,0);

    while(true){
        pthread_mutex_lock(&mm->mutex);
        printf("Worker %d get the lock\n",workerId);

        int nevents = kevent(kqueuefd,change_list,1,event_list,1,NULL);

        pthread_mutex_unlock(&mm->mutex);

        for(int i = 0 ; i < nevents ; i ++){
            struct kevent event = event_list[i];
            if(event.ident ==serverSock){
                handleNewConnection(kqueuefd,serverSock);
            }
            else if(event.filter == EVFILT_READ){
                char * msg = handleReadFromClient(workerId,event);
                handleWriteToClient(workerId,event,msg);
            }
        }
    }
}

void handleNewConnection(int kqueuefd , int serverSock){
    struct sockaddr_in clnt_addr;
    socklen_t clnt_addr_size = sizeof(clnt_addr);
    int clientSock = accept(serverSock, (struct sockaddr*)&clnt_addr,&clnt_addr_size);

    struct kevent change_list[1];

    EV_SET(&change_list[0],clientSock,EVFILT_READ,EV_ADD | EV_ENABLE , 0 , 0 , 0);
    kevent(kqueuefd,change_list,1,NULL,0,NULL);
}


char* handleReadFromClient(int workerId,struct kevent event)
{
    //event.ident 为 连接的 socket 描述符
    int bufferSize = 60;
    char *buffer = new char[bufferSize];
    memset(buffer, 0,  bufferSize);

    if(read(event.ident, buffer, bufferSize - 1)>0){
        printf("Worker %d receive : %s\n",workerId,buffer);
    }
    return buffer;
}


void handleWriteToClient(int workerId,struct kevent event,char* msg)
{
    //event.ident 为 连接的 socket 描述符
    string s = "I am Worker ";
    s.append(to_string(workerId));
    s.append(" & I reveive :");
    s.append(msg);
    char str[60];
    strcpy(str,s.c_str());
    write(event.ident, str, strlen(str));
}
```
在连续两次启动接口时，bind普遍遭遇的问题是试图绑定一个已经在使用的端口。该陷阱是也许没有活动的套接字存在，但仍然禁止绑定端口（bind 返回EADDRINUSE），它由 TCP 套接字状态 TIME_WAIT 引起。该状态在套接字关闭后约保留 2 到 4 分钟。在 TIME_WAIT 状态退出之后，套接字被删除，该地址才能被重新绑定而不出问题。等待 TIME——WAIT 结束可能是令人恼火的一件事，特别是如果您正在开发一个套接字服务器，就需要停止服务器来做一些改动，然后重启。幸运的是，有方法可以避开 TIME_WAIT 状态。可以给套接字应用 SO_REUSEADDR 套接字选项，以便端口可以马上重用。
```
int on = 1;
int ret = setsockopt( serverSock, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on) );
```
---

