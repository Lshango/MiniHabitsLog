# Nov-2020 [MiniHabits](./miniHabit.md)

## 28/11/2020

+ Coding
  > [Linux进程间通信-->Fork](https://blog.csdn.net/qq_33154343/article/details/105157044)

  ``` C++
  #include<stdio.h>
  #include<unistd.h>
  int main(int argc, char* argv[]){
    for (int i = 0; i < 4; ++i){
      printf("i = %d\n", i);
    }
      pid_t pid = -1;
      pid = fork();

      if (pid > 0) {
        printf("this is a parent process, pid = %d\n", getpid());
      } else if (pid == 0) {
        printf("this is a child process, pid = %d  ppid = %d\n", getpid(), getppid());
      }

      for (int i = 0; i < 4; ++i) {
        printf("----i = %d\n", i);
      }
      return 0;
  }
  ```

+ Breaking
  > [gdb](https://blog.csdn.net/qq_33154343/article/details/104904798)
  >> layout：用于分割窗口，可以一边查看代码，一边测试：
  >>
  >> layout src：显示源代码窗口
  >>
  >> layout asm：显示反汇编窗口
  >>
  >> layout regs：显示源代码/反汇编和CPU寄存器窗口
  >>
  >> layout split：显示源代码和反汇编窗口
  >>
  >> Ctrl + L：刷新窗口
  >>
  > cgdb 更强大的工具

* * *

## 29/11/2020

+ Coding
  > [Linux进程间通信-->wait/waitpid/execl](https://blog.csdn.net/qq_33154343/article/details/105164215)

  ``` C++
  #include<unistd.h>
  #include<sys/wait.h>
  #include<iostream>
  using namespace std;

  int main(int args, char* argv[]){
      pid_t pid{-1};
      pid = fork();

      if(pid > 0){
          cout << "This is the parent process " << getpid() << ", wait ......" << endl;
          int status{0};
          pid_t chld_pid = wait(&status);
          if(WIFEXITED(status)){
              cout << "The child process exit with " << WEXITSTATUS(status) << endl;
          }else if(WIFSIGNALED(status)){
              cout << "The child process stopped with signal " << WTERMSIG(status) << endl;
          }
      }else if(pid == 0) {
          cout << "----------------------------------------------------------\n" << 
          "This is the child process " << getpid() << ", parent process " << getppid() << endl;
          execl("/bin/ls", "wanghao", nullptr);
          cout << "Child process finish" << endl;
      }

      for (int i = 0; i < 4; ++i){
          cout << "--------" << i << endl;
      }
      return 0;
  }
  ```

+ Breaking
  > [make makefile cmake qmake都是什么，有什么区别](https://www.zhihu.com/question/27455963/answer/89770919)
  >
  > [在Linux中，编写入门的makefile文件](https://blog.csdn.net/qq_33154343/article/details/104758512)

* * *

## 30/11/2020

+ Coding
  > [Linux进程间通信-->pipe/dup2/execl](https://blog.csdn.net/qq_33154343/article/details/105164215)

  ``` C++
  #include<unistd.h>
  #include<sys/wait.h>
  #include<iostream>
  using namespace std;

  int main(int args, char* argv[]){
    int fd[2]{0};
    if(pipe(fd) == -1){
        perror("[pipe create the fd]\n");
        return -1;
    }

    pid_t pid = fork();
    if(pid > 0){
        cout << "This is the parent process" << endl;
        dup2(fd[1], STDOUT_FILENO);
        close(fd[0]);
        execl("/bin/ps", "ps", "aux", nullptr);
    }else{
        cout << "This is the child process" << endl;
        dup2(fd[0], STDIN_FILENO);
        close(fd[1]);
        execl("/bin/grep", "grep", "bash", nullptr);
    }
    return 0;
  }
  ```

+ Breaking
  > pipe默认是阻塞的，[阻塞/非阻塞/全双工/半双工区别](https://blog.csdn.net/chaofanwei/article/details/13274815)
  >
  > [同步异步的概念](https://www.zhihu.com/question/19732473)

  > [git stash](https://www.cnblogs.com/tocy/p/git-stash-reference.html)
  very useful, means save current status and try to process other emergency things.

* * *

## 01/12/2020

+ Coding
  > [Linux进程间通信-->pipe/dup2/wait/execl](https://blog.csdn.net/qq_33154343/article/details/105164215)

  ``` C++
  #include<unistd.h>
  #include<sys/wait.h>
  #include<iostream>
  #include<fcntl.h>
  using namespace std;
  //const int N{10};
  // int main(int args, char* argv[]){
  //     pid_t pid[N];
  //     int i, child_status;
  //     for (i = 0; i < N; i++) {
  //     if ((pid[i] = fork()) == 0) {
  //     /* Child */
  //         sleep(i % 3);
  //         printf("Done %d\n", getpid());
  //         exit(i);
  //         }
  //     }
  //     for (i = 0; i < N; i++) {
  //         pid_t wpid = waitpid(-1, &child_status, 0);
  //         if (WIFEXITED(child_status))
  //             printf("Saw %d done with %d\n", wpid, WEXITSTATUS(child_status));
  //         else
  //             printf("Child %d terminated abnormally\n", wpid);
  //     }
  // }

  int main(int args, char* argv[]){
      int fd[2]{0};
      if(pipe(fd) == -1){
          perror("[pipe create the fd]\n");
          return -1;
      }
      int child{0};
      for (; child < 2; ++child){
          pid_t pid = fork();
          if(pid == 0){
              break;
          }
          if(pid < 0){
              perror("fork error");
          }
      }

      if (child == 0) {//this is the first child process;
          cout << "This is the first child process ---> " << getpid() << ", parent ---> " << getppid() << endl;
          cout << "This pipe size ---> " << fpathconf(fd[1], _PC_PIPE_BUF) << endl;
          dup2(fd[1], STDOUT_FILENO);
          close(fd[0]);
          execl("/bin/ps", "ps", "aux", nullptr);
      }

      if (child == 1) {//this the second child process;
          cout << "This is the second child process ---> " << getpid() << ", parent ---> " << getppid() << endl;
          dup2(fd[0], STDIN_FILENO);
          cout << "This pipe size ---> " << fpathconf(fd[0], _PC_PIPE_BUF) << endl;
          // int flags = fcntl(fd[0], F_GETFL);  //获取原来的 flags， F_GET FL （get flags 的缩写）
          // flags |= O_NONBLOCK;                //添加非阻塞属性
          // fcntl(fd[0], F_SETFL, flags);      //设置新的属性
          close(fd[1]);
          execl("/bin/grep", "grep", "bash", "--color=auto", nullptr);
      }

      if (child == 2){//this is the parent process;
          cout << "This is the parent process ---> " << getpid() << endl;
          close(fd[0]);
          close(fd[1]);
          int wpid{0}, status{0};
          while((wpid = waitpid(-1, nullptr, WNOHANG)) != -1){//all child process
          // while((wpid = wait(&status)) != -1){//all child process
              if (wpid == 0 || wpid == 1) {
                  // cout << "--------------------------" << endl;
                  continue;
              }
              cout << "child died pid ---> " << wpid <<endl;
          }
      }
      return 0;
  }
  ```

+ Breaking
  > [Linux useful command](https://linuxtools-rst.readthedocs.io/zh_CN/latest/base/02_file_manage.html)
  1. 查看当前目录下文件个数:
  2. 给每项文件前面增加一个id编号:
  3. 搜寻文件或目录:
  4. 递归当前目录及子目录删除所有.o文件:
  5. 管道和重定向
     + 批处理命令连接执行，使用 |;
     + 串联: 使用分号 ;
     + 前面成功，则执行后面一条，否则，不执行:&&
     + 前面失败，则后一条执行: ||
  
  ```sh
  find ./ | wc -l;
  $ ls | cat -n;
  $ find ./ -name "core*" | xargs file;
  $ find ./ -name "*.o" -exec rm {} \;  
  $ ls /proc && echo  suss! || echo failed;
  $ if ls /proc; then echo suss; else echo fail; fi
  $ ls . > list 2> &1
  $ ls . &> list
  ```

* * *

## 02/12/2020

+ Coding
  > [Linux进程间通信-->fifo/mkfifo/](https://xmuli.blog.csdn.net/article/details/105266919)
  >> Read process

  ```C++
  #include<stdio.h>
  #include<unistd.h>
  #include<iostream>
  #include<sys/types.h>
  #include<sys/stat.h>
  #include<sys/fcntl.h>
  #include<string.h>
  using namespace std;

  int main(int argc, char* argv[]){
      if(argc < 2){
          cout << "Should input the pipe name as ./fifoRead myfifo." << endl;
          _exit(1);
      }

      if(access(argv[1], F_OK) == -1){
          if(mkfifo(argv[1], 0777) == -1){
              perror("create fifo file");
              _exit(1);
          }else{
              cout << "mkfifo create success ---> " << argv[1] << endl;
          }
      }

      int fd{0};
      if((fd = open(argv[1], O_RDONLY)) == -1){
          perror("Open file");
          _exit(1);
      }else{
          char buff[512];
          int num{0};
          while(true){
              int len = read(fd, buff, 512);
              buff[len] = '\0';
              if(len > 0){
                  num ++;
              }else{
                  sleep(1);
              }
              printf("%s, size %d, %d\n", buff, len, num);
          }
      }

      close(fd);
      return 0;
  }
  ```

  >>Write Process
  
  ```C++
  #include<stdio.h>
  #include<unistd.h>
  #include<iostream>
  #include<sys/types.h>
  #include<sys/stat.h>
  #include<sys/fcntl.h>
  #include<string.h>
  using namespace std;

  int main(int argc, char* argv[]){
      if(argc < 2){
          cout << "Should input the pipe name as ./fifoWrite myfifo." << endl;
          _exit(1);
      }

      if(access(argv[1], F_OK) == -1){
          if(mkfifo(argv[1], 0777) == -1){
              perror("create fifo file");
              _exit(1);
          }else{
              cout << "mkfifo create success ---> " << argv[1] << endl;
          }
      }

      int fd{0};
      if((fd = open(argv[1], O_WRONLY)) == -1){
          perror("Open file");
          _exit(1);
      }else{
          const char* msg = "2020-12-01 wanghao~";
          int i = 10;
          while(i--){
              cout << "write msg = (" << msg << "), " << i << endl;
              write(fd, msg, strlen(msg)+1);
              sleep(1);
          }
      }

      close(fd);
      return 0;
  }
  ```

+ Breaking
  > [linux可重入、异步信号安全和线程安全])(https://www.cnblogs.com/wuchanming/p/4020184.html)

* * *

## 03/12/2020

+ Coding
  > [Linux进程间通信-->signal](http://beej.us/guide/bgipc/html/single/bgipc.html#audience)

  ```C++
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <signal.h>

    void sigint_handler(int sig)
    {
        write(STDOUT_FILENO, "Ahhh! SIGINT!\n", 14);
    }

    int main(void)
    {
        // void sigint_handler(int sig); /* prototype */
        char s[200];
        struct sigaction sa;

        sa.sa_handler = sigint_handler;
        sa.sa_flags = SA_RESTART;//0; // or SA_RESTART
        sigemptyset(&sa.sa_mask);

        if (sigaction(SIGINT, &sa, NULL) == -1) {
            perror("sigaction");
            exit(1);
        }

        printf("Enter a string:\n");

        if (fgets(s, sizeof(s), stdin) == NULL) {
            perror("fgets");
        } else {
            printf("You entered: %s\n", s);
        }
        return 0;
    }
  ```

+ Breaking
  > [learning git](https://learngitbranching.js.org/?locale=zh_CN)

* * *

## 04/12/2020

+ Coding
  > [Linux进程间通信-->sig_atomic_t](http://beej.us/guide/bgipc/html/single/bgipc.html#audience)
  
  ```C++
    #include<unistd.h>
    #include<signal.h>
    #include<iostream>
    using namespace std;

    // int got_usr1{0};
    volatile sig_atomic_t got_usr1;
    void sigusr1_handler(int sig){
        got_usr1 = 10;
    }

    int main(int argc, char* args[]){
        got_usr1 = 0;
        struct sigaction sa;
        sa.sa_handler = sigusr1_handler;
        sa.sa_flags = 0;
        sigemptyset(&sa.sa_mask);

        if(sigaction(SIGUSR1, &sa, nullptr) == -1){
            perror("sigaction create");
            exit(1);
        }

        while (!got_usr1) {
            cout << "PID " << getpid() << ": working hard, get a console by #kill -USR1 pid# to stop!" << endl;
            sleep(1);
        }
        cout << "Done in by SIGUSR1 " << got_usr1 << endl;

        return 0;
    }
  ```

+ Breaking
  > [learning git](https://learngitbranching.js.org/?locale=zh_CN)
  >> git branch/commit/checkout/cherry-pick/merge/rebase/fetch/pull/push

* * *

## 05/12/2020

+ Coding
  > [Linux进程间通信-->File Locking](http://beej.us/guide/bgipc/html/single/bgipc.html#audience)

  ```C++
    #include<unistd.h>
    #include<fcntl.h>
    #include<iostream>
    using namespace std;

    int main(int argc, char* argv[]){
                        /* l_type   l_whence  l_start  l_len  l_pid   */
        struct flock fl = {F_WRLCK, SEEK_SET,   0,      0,     0 };
        fl.l_pid = getpid();
        if (argc > 1) {
            fl.l_type = F_RDLCK;
        }
        int fd;
        if ((fd = open("lockdemo.cpp", O_RDWR)) == -1) {
            perror("open, lockdemo.cpp");
            exit(1);
        }

        cout << "Press <RETURN> to try to get lock: " << endl;
        getchar();
        cout << "Trying to get lock..." << endl;

        if (fcntl(fd, F_SETLKW, &fl) == -1) {
            perror("fcntl");
            exit(1);
        }

        cout << "got lock" << endl;
        cout << "Press <RETURN> to release lock: " << endl;
        getchar();

        fl.l_type = F_UNLCK;  /* set to unlock same region */

        if (fcntl(fd, F_SETLK, &fl) == -1) {
            perror("fcntl");
            exit(1);
        }

        cout << "Unlocked" << endl;
        close(fd);
        return 0;
    }
  ```

+ Breaking
  > [learning git](https://learngitbranching.js.org/?locale=zh_CN)
  >> git branch/commit/checkout/cherry-pick/merge/rebase/fetch/pull/push
  >
  > C++ stream
  >> ![c++ stream](./Image/iostream.gif)

* * *

## 07/12/2020

+ Coding
  > [Linux进程间通信-->mmap](https://xmuli.blog.csdn.net/article/details/105322927)
  
  ```C++
    #include <stdio.h>
    #include <unistd.h> 
    #include <sys/wait.h>
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <fcntl.h>
    #include <stdlib.h>
    #include <string.h>
    #include <sys/mman.h>

    int main(int argc, char *argv[])
    {
        int fd = open("mmap_text.txt", O_RDWR);

        if (fd == -1) {
            perror("[open file] ");
            _exit(1);
        }

        int len = lseek(fd, 0, SEEK_END);
        void* ptr = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);  //创建内存映射区
        if (ptr == MAP_FAILED) {
            perror("[mmap fail] ");
            exit(1);
        }

        ((char *)ptr)[0] = 'a';
        ((char *)ptr)[1] = 'b';
        ((char *)ptr)[2] = 'c';
        
        printf("%s\n", (char*)ptr);  //释放内存映射区
        munmap(ptr, len);
        close(fd);

        return 0;
    }
  ```

## 15/12/2020

+ Coding
  > [Linux进程间通信-->Message Queues](http://beej.us/guide/bgipc/html/single/bgipc.html#audience)
  
  ```C++
  #include <stdio.h>
  #include <stdlib.h>
  #include <errno.h>
  #include <string.h>
  #include <sys/types.h>
  #include <sys/ipc.h>
  #include <sys/msg.h>

  struct my_msgbuf {
      long mtype;
      char mtext[200];
  };

  int main(void)
  {
      struct my_msgbuf buf;
      int msqid;
      key_t key;

      if ((key = ftok("msgQueueWrite.cpp", 'B')) == -1) {
          perror("ftok");
          exit(1);
      }

      if ((msqid = msgget(key, 0644 | IPC_CREAT)) == -1) {
          perror("msgget");
          exit(1);
      }
      
      printf("Enter lines of text, ^D to quit:\n");

      buf.mtype = 1; /* we don't really care in this case */

      while(fgets(buf.mtext, sizeof buf.mtext, stdin) != NULL) {
          int len = strlen(buf.mtext);

          /* ditch newline at end, if it exists */
          if (buf.mtext[len-1] == '\n') buf.mtext[len-1] = '\0';

          if (msgsnd(msqid, &buf, len+1, 0) == -1) /* +1 for '\0' */
              perror("msgsnd");
      }

      if (msgctl(msqid, IPC_RMID, NULL) == -1) {
          perror("msgctl");
          exit(1);
      }

      return 0;
  }
  ```
  
## 16/12/2020

+ Coding
  > [Linux进程间通信-->Message Queues](http://beej.us/guide/bgipc/html/single/bgipc.html#audience)
  
  ```C++
  #include <stdio.h>
  #include <stdlib.h>
  #include <errno.h>
  #include <sys/types.h>
  #include <sys/ipc.h>
  #include <sys/msg.h>

  struct my_msgbuf {
      long mtype;
      char mtext[200];
  };

  int main(void)
  {
      struct my_msgbuf buf;
      int msqid;
      key_t key;

      if ((key = ftok("msgQueueWrite.cpp", 'B')) == -1) {  /* same key as kirk.c */
          perror("ftok");
          exit(1);
      }

      if ((msqid = msgget(key, 0644)) == -1) { /* connect to the queue */
          perror("msgget");
          exit(1);
      }
      
      printf("spock: ready to receive messages, captain.\n");

      for(;;) { /* Spock never quits! */
          if (msgrcv(msqid, &buf, sizeof buf.mtext, 0, 0) == -1) {
              perror("msgrcv");
              exit(1);
          }
          printf("spock: \"%s\"\n", buf.mtext);
      }

      return 0;
  }
  ```

## 17/12/2020

+ Coding
    > [Linux进程间通信-->shareMemory](http://beej.us/guide/bgipc/html/single/bgipc.html#audience)

    ```C++
    #include <stdio.h>
    #include <unistd.h>
    #include <stdlib.h>
    #include <string.h>
    #include <sys/types.h>
    #include <sys/ipc.h>
    #include <sys/shm.h>

    #define SHM_SIZE 1024  /* make it a 1K shared memory segment */

    int main(int argc, char *argv[])   
    {
        key_t key;
        int shmid;
        char *data;
        int mode;

        if (argc > 2) {
            fprintf(stderr, "usage: shmdemo [data_to_write]\n");
            exit(1);
        }

        /* make the key: */
        if ((key = ftok("shareMem.cpp", 'R')) == -1) {
            perror("ftok");
            exit(1);
        }

        /* connect to (and possibly create) the segment: */
        if ((shmid = shmget(key, SHM_SIZE, 0644 | IPC_CREAT)) == -1) {
            perror("shmget");
            exit(1);
        }

        /* attach to the segment to get a pointer to it: */
        data = (char *)shmat(shmid, (void *)0, 0);
        if (data == (char *)(-1)) {
            perror("shmat");
            exit(1);
        }

        /* read or modify the segment, based on the command line: */
        if (argc == 2) {
            printf("writing to segment: \"%s\"\n", argv[1]);
            strncpy(data, argv[1], SHM_SIZE);
        } else {
            int i = 20;
            while(--i){
                printf("segment contains: \"%s\"\n", data);
                sleep(1);
            }
            shmctl(shmid, IPC_RMID, NULL);
        }
        /* detach from the segment: */
        if (shmdt(data) == -1) {
            perror("shmdt");
            exit(1);
        }

        return 0;
    }
    ```

* * *

## 18/12/2020

+ Coding
    > [Linux进程间通信-->Memory Mapped File](http://beej.us/guide/bgipc/html/single/bgipc.html#audience)

    ```C++
    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <unistd.h>
    #include <sys/types.h>
    #include <sys/mman.h>
    #include <sys/stat.h>
    #include <errno.h>

    int main(int argc, char *argv[])    
    {
        int fd, offset;
        char *data;
        struct stat sbuf;

        if (argc != 2) {
            fprintf(stderr, "usage: mmapdemo offset\n");
            exit(1);
        }

        if ((fd = open("mmapdemo.cpp", O_RDONLY)) == -1) {
            perror("open");
            exit(1);
        }

        if (stat("mmapdemo.cpp", &sbuf) == -1) {
            perror("stat");
            exit(1);
        }

        offset = atoi(argv[1]);
        if (offset < 0 || offset > sbuf.st_size-1) {
            fprintf(stderr, "mmapdemo: offset must be in the range 0-%d\n", sbuf.st_size-1);
            exit(1);
        }
        
        data = (char *)mmap((caddr_t)0, sbuf.st_size, PROT_READ, MAP_SHARED, fd, 0);
        if (data == (caddr_t)(-1)) {
            perror("mmap");
            exit(1);
        }

        printf("byte at offset %d is '%c'\n", offset, data[offset]);

        return 0;
    }
    ```

* * *

## 19/12/2020

+ Coding
    > [Linux进程间通信-->Sockets in Server](http://beej.us/guide/bgipc/html/single/bgipc.html#audience)

    ```C++
    #include <stdio.h>
    #include <stdlib.h>
    #include <errno.h>
    #include <string.h>
    #include <sys/types.h>
    #include <sys/socket.h>
    #include <sys/un.h>
    #include <unistd.h>

    #define SOCK_PATH "echo_socket"

    int main(void)
    {
        int s, s2, t, len;
        struct sockaddr_un local, remote;
        char str[100];

        if ((s = socket(AF_UNIX, SOCK_STREAM, 0)) == -1) {
            perror("socket");
            exit(1);
        }

        local.sun_family = AF_UNIX;
        strcpy(local.sun_path, SOCK_PATH);
        unlink(local.sun_path);
        len = strlen(local.sun_path) + sizeof(local.sun_family);
        if (bind(s, (struct sockaddr *)&local, len) == -1) {
            perror("bind");
            exit(1);
        }

        if (listen(s, 5) == -1) {
            perror("listen");
            exit(1);
        }

        for(;;) {
            int done, n;
            printf("Waiting for a connection...\n");
            t = sizeof(remote);
            if ((s2 = accept(s, (struct sockaddr *)&remote, (socklen_t*)(&t))) == -1) {
                perror("accept");
                exit(1);
            }

            printf("Connected.\n");

            done = 0;
            do {
                n = recv(s2, str, 100, 0);
                if (n <= 0) {
                    if (n < 0) perror("recv");
                    done = 1;
                }

                printf("receive from client> %s", str);

                if (!done) 
                    if (send(s2, str, n, 0) < 0) {
                        perror("send");
                        done = 1;
                    }
            } while (!done);

            close(s2);
        }

        return 0;
    }
    ```

* * *

## 20/12/2020

+ Coding
    > [Linux进程间通信-->Sockets in Client](http://beej.us/guide/bgipc/html/single/bgipc.html#audience)

    ```C++
    #include <stdio.h>
    #include <stdlib.h>
    #include <errno.h>
    #include <string.h>
    #include <sys/types.h>
    #include <sys/socket.h>
    #include <sys/un.h>
    #include <unistd.h>

    #define SOCK_PATH "echo_socket"

    int main(void)
    {
        int s, t, len;
        struct sockaddr_un remote;
        char str[100];

        if ((s = socket(AF_UNIX, SOCK_STREAM, 0)) == -1) {
            perror("socket");
            exit(1);
        }

        printf("Trying to connect...\n");

        remote.sun_family = AF_UNIX;
        strcpy(remote.sun_path, SOCK_PATH);
        len = strlen(remote.sun_path) + sizeof(remote.sun_family);
        if (connect(s, (struct sockaddr *)&remote, len) == -1) {
            perror("connect");
            exit(1);
        }

        printf("Connected.\n");

        while(printf("> "), fgets(str, 100, stdin), !feof(stdin)) {
            if (send(s, str, strlen(str), 0) == -1) {
                perror("send");
                exit(1);
            }

            if ((t=recv(s, str, 100, 0)) > 0) {
                str[t] = '\0';
                printf("server send back> %s", str);
            } else {
                if (t < 0) perror("recv");
                else printf("Server closed connection\n");
                exit(1);
            }
        }

        close(s);

        return 0;
    }
    ```

+ Breaking
    > [An Introduction to GCC](https://www.linuxtopia.org/online_books/an_introduction_to_gcc/index.html)

    ```C++
    #include <stdio.h>
    int main (void)
    {
        #ifdef TEST
            printf ("Test mode\n");
        #endif
        printf ("Running...\n");
        return 0;
    }
    ```

    The gcc option -DNAME defines a preprocessor macro NAME from the command line. If the program above is compiled with the command-line option -DTEST, the macro TEST will be defined and the resulting executable will print both messages:

    ```sh
    $ gcc -Wall -DTEST dtest.c
    $ ./a.out
    Test mode
    Running...
    ```

    If the same program is compiled without the -D option then the "Test mode" message is omitted from the source code after preprocessing, and the final executable does not include the code for it:

    ```sh
    $ gcc -Wall dtest.c
    $ ./a.out
    Running...
    ```
