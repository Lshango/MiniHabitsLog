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

# 30/11/2020

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
