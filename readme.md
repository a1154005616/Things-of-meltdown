//首先是KVM hypervisor在实体机上的搭建，因为一般情况下我们用的系统都是Windows，因此这里需要安装一个双系统
//在这里我安装的是Ubuntu18.04.6，因为经过多个版本的测试发现这个版本的Ubuntu兼容性较好
//在真机上安装Ubuntu18.04.6可以使用软碟通通过U盘安装，启动盘制作好以后进入BIOS选择U盘启动，接下来等待系统安装就好
//假设到这里Ubuntu已经安装完成，接下来开始在真机上配置KVM
//首先在Ubuntu终端安装虚拟机管理器virt-manager,在此之前最好先更新系统包
sudo apt-get update
sudo apt-get install virt-manager
//安装完成后，启动虚拟机管理器
sudo virt-manager
//可以看到打开的虚拟机管理器界面，选择添加连接，选择KVM，接下来选中准备好的Ubuntu 16.04.7包（因为此项目中客户机我们选择的是Ubuntu16），进行安装即可，安装完成后真机上的KVM就装好了
//接下来是到此为止我的项目进展:
//这是Linux源码网站：https://elixir.bootlin.com/linux/latest/source/mm/mmap.c
//这是Linux时间片管理的文档：https://blog.csdn.net/arm7star/article/details/78648718
//这是阴影钩子的教学文档：https://github.com/tandasat/DdiMon
迄今为止我对阴影钩子相关入口的了解如下：
用户进程被定时器中断，调用__irq_usr函数，__irq_usr调用定时器中断处理函数更新当前进程的时间片，检查当前进程是否需要被抢占，设置抢占标识，中断退出；
退出中断之前，先判断_TIF_WORK_MASK标识是否被设置，该标识包括重新调度标识，有设置则调用do_work_pending函数，该函数检查重新调度标识，然后调用schedule进行进程切换。
在查找__irq_usr函数时发现它下面包含可能的时间片中断处理总过程：
__irq_usr:usr_entry // 中断相关上下文保存
kuser_cmpxchg_check
irq_handler // 中断处理(在这里逐级调用到定时器函数，更新当前进程的虚拟运行时间，设置重新调度标志...)
get_thread_info tsk // 获取当前进程thread_info(里面有进程重新调度标志等)
mov why, #0
b ret_to_user_from_irq // 跳转到返回用户态的函数
根据对irq_handler的描述猜测我们要找的函数入口就在irq_handler中，接下来的工作就是从irq_handler中找到入口函数
从学长那里了解到的关于放阴影钩子的另一个位置：
可考虑的另一种方法：如果是时间片用尽的话，一定会到schedule 函数这里，task_tick就是做检测时间片用尽的相关函数，在这个位置放hook
//以上就是我目前为止的所有进度，如果有相关的问题欢迎联系我，QQ：1154005616

### Running the PoC 

```bash

root@spectre$ gcc -o spectre -std=c99 spectre.c
root@spectre$ ./spectre

```
