1. *vfork*

   - 与 *fork* 一样都创建新的进程，他的是目的是执行一个程序。

   - 与 *fork* 的区别在于：

     - **它并不将父进程的地址空间复制到子进程中** 。在子进程 *exec/exit* 之前，和父进程共享地址空间，提高了工作效率。但是在在子进程 *exec/exit* 之前，子进程如果修改了数据、进行函数调用、返回都会带来未知的结果。
     - *vfork* 保证子进程比父进程先运行，在子进程 *exec/exit* 之后父进程才会运行。

   - 案例

     ```c
     // 将上面的修改如下：
     if((pid=vfork()) < 0){
         printf("fork() error.\n");
         exit(1);
     }
     else if(pid == 0){
         globvar++; // 会改变父进程的变量值
         var++;
         _exit(0);
     }
     // 去掉sleep(2)是因为vfork能保证子进程先运行
     ```

   - 结果对比：

     ```c
     # fork 
     szz@ubuntu:~/Study/SystemProgram/IO$ ./f
     a writte to stdout.
     before fork.
     pid=4304, globvar=11, var=89. # 子进程
     pid=4303, globvar=10, var=88. # 父进程
     
     # vfork
     szz@ubuntu:./vf
     a writte to stdout.
     before fork.
     pid=4301, globvar=11, var=89.# 父进程
     ```