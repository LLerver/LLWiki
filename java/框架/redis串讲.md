redis串讲

**redis安装 (先编译,后安装)**

​		1)  wget https://download.redis.io/releases/redis-6.0.9.tar.gz      下载tar包

​		2) tar xf redis-6.0.9.tar.gz   进行解压获取到源码

​		3) 进行编译操作,从readME文件中查看对应的命令,

​				比如Building Redis    简单编译 % make

When you update the source code with `git pull` or when code inside the dependencies tree is modified in any other way, make sure to use the following command in order to really clean everything and rebuild from scratch:    % make distclean     简单的讲就是遇到问题时候,你可以通过distclean命令来清除之前的操作

​				Running Redis		运行Redis    ①.% cd src    ②. % ./redis-server  在src目录下 执行redis_server文件即可   但是一般都是后台启动Redis 

​				Installing Redis		安装Redis  % make install    这种命令是默认安装的意思 默认路径是/usr/local/bin 目录下 还有一种方式是指定安装路径的方法来进行安装  % make PREFIX=/some/other/directory install

    In order to install Redis binaries into /usr/local/bin, just use:
    % make install
    You can use `make PREFIX=/some/other/directory install` if you wish to use a
    different destination.

​				安装好之后还可以通过.sh脚本进行开启后台服务线程的方式来运行Redis

    Make install will just install binaries in your system, but will not configure
    init scripts and configuration files in the appropriate place. This is not
    needed if you just want to play a bit with Redis, but if you are installing
    it the proper way for a production system, we have a script that does this
    for Ubuntu and Debian systems:
    % cd utils
    % ./install_server.sh
**编译**

​			进行实际安装,执行make命令即可,如果抛出cc:Command not found相关异常,没有找到cc命令.进一步表示当前虚拟机没有安装C语言的编译器,Redis是C语言写的.所以需要安装对应的CC编译器,运行 yum install gcc -y 命令即可自动安装.

​			安装完成之后,继续执行make命令,如果还报错,则先执行 make distclean命令来进行清除之前的缓存文件即可.再次执行make命令就可以对Redis源码进行编译了.

​			其实make编译命令,主要还是依赖于Makefile文件来执行的,也就是make是通过Makefile里面的内容来进行具体的顺序编译动作.通过查看Makefile文件得知,其实是去Redis中的src目录中读取Makefile文件的.src下的Makefile文件才是真正的安装细则了.而且src目录还有一个Redis-server的可执行程序,在这里不可能直接去使用redis-server可执行程序,需要将其放在操作系统的某个目录下

​			执行完make编译命令之后,接着开始安装redis.这里使用指定路径的方式来进行相关安装,而不是使用redis默认的安装路径. 执行完 make PREFIX=/usr/local/tools/Redis6 install 命令 .在对应的目录下会生成bin目录,bin里面会有多个相关的redis的可执行程序.这个总结make install命令的话,就相当于一个拷贝的过程.

​			接下来做redis的环境变量配置,类似于安装完JDK之后配置环境变量.  执行 vi /etc/profile 命令进行编辑操作.

如果profile文件中没有相关的export PATH的内容.则需要自己添加进去.输入命令echo $PATH.将查询结果输入到profile文件中.保存退出,在执行 . /etc/profile进行重新加载,或者reboot重新启动也行.

![60EE7D76-BB5E-454E-B3FD-F39D010E7602](/Users/maguagua/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/237540824/QQ/Temp.db/60EE7D76-BB5E-454E-B3FD-F39D010E7602.png)

**安装**

​			设置redis进行开机启动,一般这种服务类型的软件,都是设置为开机启动后台运行对应的服务程序. 进入到源码src的utils目录下,有个install_server.sh脚本文件.执行完成之后,就会设置对应的端口,配置文件名称地址,以及redis持久化文件的存放路径等相关信息.执行完成之后,脚本文件会执行一些其他的操作

Copied /tmp/6379.conf => /etc/init.d/redis_6379				// 复制一份配置文件  一般init.d目录下面都是存放服务的脚本文件
Installing service...			// 安装服务
Successfully added to chkconfig!	// 设置开机启动
Successfully added to runlevels 345!	//设置345的运行级别
Starting Redis server...	// 运行了redis的服务程序
Installation successful!	// 安装成功了

​			注意脚本文件install_server.sh是可以重复执行的,执行一次安装一个redis实例.只需要改变对应的端口即可.

Copied /tmp/6380.conf => /etc/init.d/redis_6380		// 又创建了一个6380端口的redis实例
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!



**1.Redis与服务器内核交互**

​		1) Redis首先会调用一个重装系统内核提供的socket方法(可以使用man 2 socket命令来查看对应的方法),该方法会返回一个连接的文件描述符file descriptor(简称FD)对象回来,其实该对象就是一个数值

​		2) 然后针对socket方法返回的fd6对象,会继续调用内核提供的bind()方法,会将这个fd6对象绑定在对应端口6379上.

​		3) 然后会对文件描述符的监听动作,listen()

​		4) 然后通过accept()进行阻塞等待,等待被监听的文件描述符上面有没有客户端连进去,如果等待到了,则会返回一个客户端对应的文件描述符对象.在没有客户端连接进来的时候,accept会一直阻塞着.

​		5) 此时有个客户端C通过tcp协议,经历了三次握手,然后连接进来了,内核将客户端C的信息,通过对应6379端口号,找到对应的进程,找到对应的fd6对象.此时accept()会返回一个客户端的fd8对象.注意fd8是客户端的文件描述符对象,通过accept()返回得到的.此时fd8代替的是客户端,fd6代替的是监听对象

​		6) 客户端连接进来,肯定是想读取数据的,此时Redis会调用内核方法read();将fd8作为参数传递进去.就可以读取对应的数据了.**注意了,此时客户端只是跟Redis服务器建立了连接,只是想读取数据,但是客户有可能并没有发送读取数据的请求,所以read()此时实际也是处于阻塞状态的!!!**

整体的看起来,服务器启动,然后要进入accept()等待阻塞状态,这个时候阻塞是因为Redis要等待客户端连接它,等客户端真正连接它了,正常讲,客户端连接上了,就会立即发送读取的请求,然后Redis顺理成章的执行read().如果客户端不发送读取的请求,所以Redis还是处于read()阻塞状态的.但是Redis是单线程工作的,这要是在read的时候被阻塞了,也挺麻烦的.怎么办呢?

​		**升级一(BIO时代)**:Redis客户创建一个新的thread线程,通过新建的线程来代替Redis的工作主单线程来进行对应的read()方法.同理,在客户端没有发送读取请求的时候,**这条新线程也是处于阻塞状态的!!!!**但是主线程不会阻塞的,Redis又好了,回到了accept(),继续等待新的客户端连接进来.

​		如果这个时候再来一个新的客户端B,经过TCP三次握手,又建立了连接,Redis又会抛出一条新的线程来,这条新线程来复制客户端B的read()方法,也是阻塞的!

​		升级一方案总结,这就是早期的多线程模型,一条新的线程对应一条连接.哪个客户端想读取数据了,就发送读取请求,对应的线程就会执行read(),然后返回对应的结果给客户端.

​		但是,Redis创建线程成本比较高,需要用户态,内核态的调用.同时,线程栈是独立的,也会占用内存资源,这样无休止的创建新线程,也一直处于阻塞状态的话,资源会非常紧张.所以不能粗暴的通过创建新线程来解决对应的问题.

​		需要注意,其实read方法是内核提供的,跟Redis无关,在执行read()的时候,是内核让对应的线程阻塞的.Redis主线程需要回避一个客户端连接进来而长时间不发消息,但是Redis有需要去应对更多的客户端连接,所以Redis选择了创建新线程来解决.但是根本原因还是因为内核提供的read()是阻塞的,所以能不能从内核上动手呢?!

​		伪代码:

​					fd6 = socket();

​					bind(fd6,6379);

​					lisent();

​					fd8 = accept(fd6);	// fd8为客户端的文件描述符,当没有客户端连接进来的时候,accept()处于阻塞

​					read(fd8);		// 如果客户端没有发送读取的请求,read()也是阻塞状态

​		**升级二(NIO时代)**:

​				伪代码:

​					fd6 = socket();

​					bind(fd6,6379);

​					lisent();

​					while(true){

​						list  = new list();

​						fd9 = accept(fd6);	// fd8为客户端的文件描述符,当没有客户端连接进来的时候,accept()处于阻塞

​						list.add(fd9);

​						for(list){

​							read(fd9);	

​						}

​					}

​			Redis开启一个while死循环,然后accept()进入阻塞,等待客户端的连接,客户端连接进来,开始执行for循环中的read().这个时候可以设置read是非阻塞的,read读fd9的时候,如果客户端此时发送了读取的数据请求,就返回数据,如果没有读到客户端发送的读取请求数据,就会抛一个异常值出去,所以read方法就不会被阻塞住了,因为read()和accept()在死循环中,执行完read又会去继续执行accept(),然后Redis又处于一个阻塞等待客户端连接状态.

​			如果这个有客户端又进来了,则会获取到fd10,添加到集合list当中去,继续for循环,然后调用read(fd9);read(fd10);如果有对应客户端的读取请求进来,则顺利读取到数据返回,如果没有读取请求进来,抛异常,不会阻塞read();又回到accept()方法.

​			这个阶段主要解决了无限创建新线程的问题,但是这种设计,如果客户端特别特别多,有10W个客户端,那么对应的list集合中的fd对象就有10W个,然后for循环里面会调用read方法10W次,这里需要明白,每次调用内核read方法都会涉及到用户态和内核态之间的切换过程,这个过程中可能只会有一个fd有对应的数据,这个时候Redis会读取到对应的数据,但是这一个for循环走下来,其他9W多个read调用,都是无效的,**严重造成了资源的浪费**!!!read方法的复杂度趋向于O(n);但是一个读取请求进来,难道read方法的复杂度不应该是O(1)么???怎么回避这种多次无效的read调用?

​			**升级三(多路复用器)**:

​				伪代码:

​					fd6 = socket();

​					bind(fd6,6379);

​					lisent();

​					while(true){

​						list  = new list();

​						fd9 = accept(fd6);	// fd8为客户端的文件描述符,当没有客户端连接进来的时候,accept()处于阻塞

​						list.add(fd9);

​						newList = select(list);

​						for(newList){

​							read(fd9);	

​						}

​					}

内核再次升级,提供了一个select();select方法的大概含义是,调用select方法一次,就能让程序一次性监控多个文件描述符!然后等待一个或多个文件描述符到达了文件可读的状态.

ok,再来梳理一遍,假如此时list集合中存储了10W个fd对象,然后调用一次内核select(list);(时间复杂度是O(1))然后内核当中会对这个10w个对象的list集合进行遍历,是内核去遍历这个大的集合.找到其实能够进行读取操作的fd对象,装进newList集合中,返回给Redis(返回结果集的动作时间复杂度也是O(1)),此时Redis拿到可执行read的结果集之后,再次进行遍历调用read方法,因为内核已经监听了这个结果集中的fd对象,确保在对这些fd对象调用read方法时候,不会使read方法进入到阻塞状态.所以Redis中的for循环可以顺利完成.

​		注意,在时代二中,是Redis通过for循环反复的调用read方法10W次,反复的内核态/用户态之间切换,但是时代三种,Redis只调用了一次select方法,只进行了一次内核态/用户态切换,然后内核去完成了这10W次的循环遍历监控操作.节省了很多很多的资源,这里就可以明白什么是多路复用了,Redis只调用了一次select,就管理了很多个连接通路的事件!select返回了一个可执行read且不会阻塞read的结果集.复用了select().**但是时代三还是有缺陷,就是这10W的list集合问题.**因为内核只是接收了这个大list集合,内核也不知道哪个fd的数据到了,只能主动的去遍历这些对象,挨个挨个的处理.内核这样会浪费很多CPU时间片,那要怎么办呢?

​		假如假如假如,现在客户端能够自己在发送读取请求的时候,同时去告知内核,自己的fd数据到了,可以让内核去返回Redis进行read操作,这种思路行不行?

​		**升级四(epoll)**:这部分内容还需要继续揣摩

​				大致思路如下:

​					fd6 = socket();

​					bind(fd6,6379);

​					lisent();

​					fd5 = epoll_create();

​					epoll_ctl(fd5,fd6);

​					while(true){
​						result =epoll_wait();	//epoll_wait还会返回一些别的信息,比如fd6描述的事件

​						fd9 = accept(result.fd6);		// fd9是客户端对应的文件描述符

​						epoll_ctl(fd5,fd9);

​						result =epoll_wait();

​						read(fd);

​					}

​				Redis在调用监听方法之后,会去调用epoll_create()方法,这个时候内核中会创建一个epoll,同时会开辟一块空间fd5出来,同时将fd5返回给Redis,

​				然后Redis调用epoll_ctl();将epoll对象fd5和socket对象的fd6进行绑定操作,也就是在内核fd5的空间内,会存储fd6的文件描述符.然后内核中的fd5空间(红黑树)中的fd6对象会做什么呢?fd6此时是一个监听,监听accept()事件

​				然后Redis此时就会在死循环里面持续调用epoll_wait(fd5),wait在等什么呢?等客户端C1的连接进来,(这里有个计算机系统的中断概念,后续需要补充进来.),CPU自身其实是在忙别的事情了,这个时候有个客户端进来和操作系统建立了连接,通过计算机中断操作,触发了fd5空间中的fd6对应的accept事件,然后此时内核又会开辟一块新的空间list(list集合)!!!这个新的空间主要用来存储fd6文件描述符,这个时候epoll_wait就有返回值了,Redis根据wait的返回值,开始调用accept(),得到客户端的文件描述符.注意,此时客户端也仅仅是刚跟Redis建立连接,客户端并没有发送读取的请求事件,

​				所以Redis会接着调用epoll_ctl(),在空间fd5中存储客户端的文件描述符fd9.那么fd9在内核空间fd5中会等待一个read事件.Redis又会调用epoll_wait();假如此时客户端C1刚准备发送读取请求事件时候,恰好又有一个新的客户端C2来了.那么在内核空间fd5中会触发fd6的accept事件,同时也会触发fd9的read事件,那么fd6和fd9又会全部进入到隔壁的list空间中,通过epoll_wait方法的返回值给Redis,然后Redis根据返回信息开始进行accept或者read操作.



​			升级四相比升级三,升级三每次循环会传递10W个fd对象.而且内核会主动遍历10W个fd对象.

​			升级四:当一个客户端连接进来之后,通过epoll_ctl方法,只传递了这一次,后续Redis和内核之间都是通过epoll_wait来死循环调用的.怎么确保只需要传递一次的?就是因为内核中开辟一块空间来永久存储这个客户端对应的文件描述符fd对象的.调用epoll_wait的时候并不需要再传递fd对象了.

​			怎么解决内核10W次遍历的呢?通过内核的中断事件机制event来保证的.这个event事件是异步的了,内核只需要管理好红黑树和list这两块空间的fd对象迁移即可.

​			**这个强调一下,上述四种升级,包括最原始的多线程模式,都是Redis程序自身在调用read(),只要调用read(),那么Redis就是同步的!!!!**,读取数据这事还是需要程序自己去调用.客户端的事件到达了,通过内核的中断机制来触发事件,内核转手将事件返给了程序进程本身,读取这个操作还是需要程序Redis自己去完成,内核并不会帮忙去读取相关数据.epoll_ctl的作用主要是操作控制红黑树,添加或删除对应的fd文件.

​			epoll是所有模型当中,让CPU内核浪费最少,速度最快的一种模型.回顾起来,Redis还是一个单线程就能完成这些事情喔~但是Redis的单线程也有弊端,那就是无法发挥服务器多核心的优势!!!



​		Nginx也是使用epoll,不过是多进程的.Nginx的worker是根据核心数一比一或者一比二的比例,1核心,1Nginx或2个Nginx.那Nginx为什么能使用多线程呢,

​		首先要明确,使用多线程会遇到一致性的问题,Redis需要存储数据的啊.如果多线程的话,就需要加锁,多线程一旦跟加锁沾上边,就转变成了串行化,无论是悲观锁还是乐观锁.Redis单线程,也是串行化,但是不需要加锁啊.如果Redis变成多线程,然后加了锁,变成串行化.加锁解锁是需要资源开销的啊!!!何必呢!

​		而Nginx是静态的web server ,它不是一个容器,它自身是不存储数据的.它没有对数据写的行为,它更多的是对数据读的行为.而且内核提供了一个零拷贝的sendfile();

​		原本客户端进来   客户端->内核->Nginx 然后Nginx调用read()到内核,内核去找到对应的文件加载到内核的空间中,然后内核将文件返回给Nginx,然后Nginx调用write(),将文件写到内核中去,然后内核通过socket返回给客户端.

​		而零拷贝这是  客户端->内核->Nginx 然后Nginx直接调用sendfile()到内核,内核找到对应文件加载到内核空间,然后直接通过socket直接返回给客户端了.所以Nginx提供了一个配置项叫sendfile



​		而且Redis在企业级系统中,不建议Redis存储很多很多热数据,进来多弄几个Redis实例,如果Redis做AOF或者RDB的速度会很快.



**2.redis的IO thread模型**

​		在之前讲述了redis是利用Linux中的epoll来完成单线程工作的,但是redis的大体区分可以划分为两个阶段,一个是状态,一个是IO

​		所谓的状态阶段就是客户端连接到redis中,然后通过内核,epoll以及内核的中断机制等方式,来让redis了解到客户端的状态.

​		而IO阶段则是,客户端与redis建立了连接,客户端开始进行实质性的操作redis了,比如增删改查等操作.假如一个修改请求进来,先通过内核和epoll_wait让redis知道了它接下来的事,然后redis进行read指令,然后从内存中读取对应的数据,加载到redis中,然后redis开始计算操作,完事将结果重新写回到内存中.

​		这里可以总结出有三个步骤,第一,read数据,第二,计算数据,第三,write数据.redis的工作线程是单线程,整个操作redis的过程也是串行化的.

