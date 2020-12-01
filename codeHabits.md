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
