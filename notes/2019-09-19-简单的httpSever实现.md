简单的HTTP Server的实现

## 一、简介

​       本次作业通过网络编程实现了一个简单的HTTP Server（Panther）。Panther使用了线程池以及Epoll技术在后台进行线程和I/O调度，避免线程的频繁创建浪费资源，并且使用Epoll来达到比较好的I/O复用性能（对比select）。最后实现的Panther能够具有下面功能：

 

1）通过命令行配置服务器的参数，端口，最大线程数，核心线程数，网站的根地址等配置参数

2）支持Http Get请求，提供css文件，img图片，js文件，html文档等静态web资源的请求

 

## 二、主要的模块划分

panther > 主程序，线程池的创建、参数的初始化，服务器的监听和停止

 

params > 命令行参数的解析

 

service > 服务器的服务层，提供服务器的启动和资源释放

 

task  > 任务，可以放入线程池消耗的任务，比如处理套接字连接的任务

 

thread_pool > 线程池的实现

 

http_errror > 定义不同的出错类型和出错信息

 

http_method > 实现不同的http方法，GET等

 

http_mine > 定义不同的MIME类型

 

http_request > 解析不同的http请求

 

## 三、核心模块的代码

​       Panther最主要的核心模块有线程池、http_method、（只实现了GET方法）、service、task以及http_request 五大模块

 

## 3.1 线程池的实现

​       为了提高服务器的响应速度我们采用多线程来支持多客户端同时访问，并且避免线程的反复创建，我们使用线程池来进行调度。线程池的实现就是一个简单的消费者和生产者的问题，主线程中产生任务，线程池由一个线程队列表示，每一个线程池对应一个任务队列，线程池里面的线程就消费任务队列里面的任务，如果任务队列为空线程就需要阻塞，等待有新的任务放入任务队列时候再唤醒线程池里面的线程。

 

**线程池的基本定义：**

 

typedef struct worker_t{

​    void* (*process)(void *arg); // 执行任务的方法

​    void *arg; // 任务的参数

​    struct worker_t *next; // 实现任务队列的指针

} worker_t;

 

 

typedef struct pthread_pool_t{

​    pthread_mutex_t mutex; 

​    pthread_cond_t cond;

​    worker_t *worker_queue;

​    int shutdown;

​    pthread_t *tids;

​    int core_thread_num;

​    int max_thread_num; // 最大的线程数量

​    int thread_step_increment; // 每次增加的线程数量

​    int worker_queue_size; 

} pthread_pool_t;

 

 

static pthread_pool_t *pool = NULL;

 

int pool_add_worker(void* (*process)(void *arg), void *arg);

 

void *pthread_run(void *arg);

 

void pool_init(int core_num);

 

int pool_destroy();

 

具体实现参考：ptr_thread_pool.c

 

## 3.2 Service和task层

service和task是对服务器提供的功能和进行的任务进行抽象，服务主要针对服务器的状态管理，task主要针对数据的交互。

 

**服务层：**

// 启动服务器

void start_server(int sock_listen);

 

// 停止服务器

void stop_server();

 

 

**任务层：**

**处理连接套接字，这里采用epoll****来监听套接字接收客户端数据事件，实现I/O****多路复用，接收到数据后，先进性解析，然后在进行响应。**

void* handle_con_task(void* arg);

 

void* handle_con_task(void *arg){

 

​    int confd = *((int*) arg);

​    free(arg);

​    int i=0;

​    int epfd;

​    int readyn = 0;

​    

​    struct epoll_event config_ev,events[MAX_FD_SIZE];

​    epfd = epoll_create(1000);

​    

​    config_ev.events = EPOLLIN | EPOLLET;

​    config_ev.data.fd = confd;

​    epoll_ctl(epfd, EPOLL_CTL_ADD, confd, &config_ev);

 

​    int retval = 1;

 

​    while(retval){

​    

​        readyn = epoll_wait(epfd, events, MAX_FD_SIZE, 500);

​        for(i=0; i < readyn; i++){

​        

​            if(events[i].data.fd == confd){

​             // 客户端数据到来,解析报文并且响应

​                struct worker_ctl *wctl = (struct worker_ctl*) 

​                    malloc(sizeof(struct worker_ctl));

​                wctl->opts.work = wctl;

​                wctl->conn.work = wctl;

​                

​                wctl->conn.con_req.conn = &wctl->conn;

​                wctl->conn.con_res.conn = &wctl->conn;

​                wctl->conn.cs = -1;

​                wctl->conn.con_req.req.ptr = wctl->conn.dreq;

​                wctl->conn.con_req.head = wctl->conn.dreq;

​                wctl->conn.con_req.uri = wctl->conn.dreq;

​                wctl->conn.con_res.fd = -1;

​                wctl->conn.con_res.res.ptr = wctl->conn.dres;

 

​                wctl->conn.cs = confd;

​                

​                struct vec *req = &wctl->conn.con_req.req;

​                memset(wctl->conn.dreq, 0, sizeof(wctl->conn.dreq));

​                req->len = read(confd, wctl->conn.dreq,

​                        sizeof(wctl->conn.dreq));

​                req->ptr = wctl->conn.dreq;

 

​                if(req->len > 0){

​                

​                  wctl->conn.con_req.err = Request_Parse(wctl);

​                  Request_Handle(wctl);

​                }

​                else{

​                    close(confd);

​                    retval = 0;

​                    free(wctl);

​                    wctl = NULL;

​                }

​            

​            }

​        }

​    

​    }

 

​    return NULL;

}

 

## 3.3 http_request和http_method

http_request就是接收http请求，并且解析http，获得请求的uri和请求的资源。http_method是进行方法响应的实现，响应GET、POST、等方法。通过HTTP协议来构造正确的响应头，和资源对应的响应体。

 

 

**http_request:**

// 解析请求

int  Request_Parse(struct worker_ctl *wctl);

// 处理请求，判断请求方法，调用不同处理请求函数

int Request_Handle(struct worker_ctl* wctl);

 

**http_method:**

 

// 实现了一个处理http Get请求的函数

static int Method_DoGet(struct worker_ctl *wctl)

 

static int Method_DoGet(struct worker_ctl *wctl)

{

​    DBGPRINT("==>Method_DoGet\n");

​       struct conn_response *res = &wctl->conn.con_res;

​       struct conn_request *req = &wctl->conn.con_req;

​       char path[URI_MAX];

​       memset(path, 0, URI_MAX);

 

​       size_t            n;

​       unsigned long     r1, r2;

​       char const *fmt = "%a, %d %b %Y %H:%M:%S GMT";

 

​       size_t status = 200;           /*状态值,已确定*/

​       char const *msg = "OK";   /*状态信息,已确定*/

​       char  date[64] = "";         /*时间*/

​       char  lm[64] = "";                    /*请求文件最后修改信息*/

​       char  etag[64] = "";          /*etag信息*/

​       big_int_t cl;                       /*内容长度*/

​       char  range[64] = "";        /*范围*/      

​       struct mine_type *mine = NULL;

 

​       time_t t = time(NULL);      

​       (void) strftime(date, 

​                            sizeof(date), 

​                            fmt, 

​                            localtime(&t));

 

​       (void) strftime(lm, 

​                            sizeof(lm), 

​                            fmt, 

​                            localtime(&res->fsate.st_mtime));

 

​       /*ETAG*/

​       (void) snprintf(etag, 

​                            sizeof(etag), 

​                            "%lx.%lx",

​                            (unsigned long) res->fsate.st_mtime,

​                            (unsigned long) res->fsate.st_size);

​       

​       mine = Mine_Type(req->uri, strlen(req->uri), wctl);

 

​       cl = (big_int_t) res->fsate.st_size;

 

​       memset(range, 0, sizeof(range));

​       n = -1;

​       if (req->ch.range.v_vec.len > 0 )/*取出请求范围*/

​       {

​              printf("request range:%d\n",req->ch.range.v_vec.len);

​              n = sscanf(req->ch.range.v_vec.ptr,"bytes=%lu-%lu",&r1, &r2);

​       }

 

​       printf("n:%d\n",(int)n);

​       if(n   > 0) 

​       {

​              status = 200;

​              lseek(res->fd, r1, SEEK_SET);

​              cl = n == 2 ? r2 - r1 + 1: cl - r1;

​              (void) snprintf(range, 

​                                   sizeof(range),

​                                   "Content-Range: bytes %lu-%lu/%lu\r\n",

​                                   r1, 

​                                   r1 + cl - 1, 

​                                   (unsigned long) res->fsate.st_size);

​              msg = "OK";

​       }

 

​       memset(res->res.ptr, 0, sizeof(wctl->conn.dres));

​       // 构造响应报文

​       snprintf(

​              res->res.ptr,

​              sizeof(wctl->conn.dres),

​              "HTTP/1.1 %d %s\r\n" 

​              "Date: %s\r\n"

​              "Last-Modified: %s\r\n"

​              "Etag: \"%s\"\r\n"

​              "Content-Type: %.*s\r\n"

​              "Content-Length: %lu\r\n"

​              //"Connection:close\r\n"

​              "Accept-Ranges: bytes\r\n"            

​              "%s\r\n",

​              (int)status,

​              msg,

​              date,

​              lm,

​              etag,

​              (int)strlen(mine->mime_type),

​              mine->mime_type,

​              cl,

​              range);

​       res->cl = cl;

​       res->status = status;

​       printf("content length:%d, status:%d\n",res->cl, res->status);

​       DBGPRINT("<==Method_DoGet\n");

​       return 0;

}

 

## 3.4 Panther主程序

 

Panther主模块主要负责信号量的处理，监听套接字的产生，以及参数的解析，线程池的管理。

 

**创建监听套接字：**

 

int do_listen(){

 

​    int sock_listen;

​    sock_listen = socket(AF_INET, SOCK_STREAM, 0);

 

​    if(sock_listen == -1){

​        perror("创建套接字失败");

​    }

 

​    struct sockaddr_in server_sockaddr;

 

​    memset(&server_sockaddr, 0, sizeof(server_sockaddr));

​    server_sockaddr.sin_family = AF_INET;

​    server_sockaddr.sin_port = htons(param_conf.port);

​    server_sockaddr.sin_addr.s_addr = htonl(INADDR_ANY);

 

​    int res = bind(sock_listen, (struct sockaddr *) &server_sockaddr, sizeof(server_sockaddr));

​    

​    if(res == -1){

​        perror("绑定套接字失败");

​    }

 

​    res = listen(sock_listen, 100);

​    if(res == -1){

​        perror("监听失败");

​    }

 

​    printf("启动panther服务器成功!\n");

​    printf("在端口:%d上监听", ntohs(server_sockaddr.sin_port));

​        

​    return sock_listen;

}

 

**主程序main****函数：**

 

int main(int argc, char* argv[]){

​    

​    signal(SIGINT, sig_int);

 

​    init_param(argc, argv);

 

​    int sock_listen = do_listen();

 

​    start_server(sock_listen);

 

​    return 0;

}

## 四、测试结果

 采用makefile来对Panther进行编译：

 ![](F:\Notebook\Assets\pather-make.png)

 

 

启动Panther     

 ![](F:\Notebook\Assets\lauch_pather.png)

 

 Panther 响应图片文件

 ![](F:\Notebook\Assets\pather-pic.png)

 

  Panther 响应css

 ![](F:\Notebook\Assets\pather-css.png)

Pather 响应js文件

![](F:\Notebook\Assets\pather-js.png)

 Pather响应Html页面（综合css，js，img和文本内容）

 ![](F:\Notebook\Assets\pather-htm.png)

Panther后台信息

​     ![](F:\Notebook\Assets\pather-back.png)

 