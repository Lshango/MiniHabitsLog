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

***********************************

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

***********************************

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

***********************************

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

***********************************

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

***********************************
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

***********************************

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

***********************************

## 05/12/2020

+ Coding
  
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

***********************************

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

+ Breaking
  