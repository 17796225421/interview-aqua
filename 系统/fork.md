1. *fork*

   - 用于创建子进程。

   - 在调用时，返回两次：子进程的返回值是0，父进程的返回值的新建子进程的ID。

   - 子进程是父进程的副本。子进程和父进程继续执行

      

     fork

      

     之后的指令。

     - 子进程获得父进程的 **数据空间、堆、栈的副本**
     - 共享的是：**文件描述符、mmap建立的映射区**
     - 子进程和父进程共享的是 **代码段**，*fork* 之后各自执行。
     - 父进程和子进程的执行顺序谁先谁后是未知的，是竞争的关系。

   - *COW*
     *COW* 即写时复制(*Copy-On-Write*)， **数据空间、堆、栈的副本**在创建子进程时并不创建副本。而是在父进程或者子进程修改这片区域时，内核为修改区域的那块内存制作一个副本，以提高效率。

   - fork

     fork

      

     失败的原因：

     - 系统中已经有太多的进程
     - 该实际用户Id的进程数超过了系统限制

   - 案例

     ```c
     #include<stdio.h>
     #include<stdlib.h>
     #include<unistd.h>
     
     int globvar = 10;
     char buf[] = "a writte to stdout.\n";
     
     int main(int argv, char* argc[]){
     
         int var;
         pid_t pid;
     
         var = 88;
         if(write(STDOUT_FILENO, buf, sizeof(buf)-1) != sizeof(buf)-1){
             printf("write error");
             exit(1);
         }
         printf("before fork.\n");
         // 创建子进程后，后面的代码，父进程和子进程独立运行。
         if((pid=fork()) < 0){
             printf("fork() error.\n");
             exit(1);
         }
         else if(pid == 0){
             globvar++; //子进程运行不改变父进程的值
             var++;
         }
         else
             sleep(2);
         printf("pid=%ld, globvar=%d, var=%d.\n",(long)getpid() , globvar, var);
         retdurn 0;
     }
     ```