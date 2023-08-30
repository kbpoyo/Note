
# CPU执行BIOS

CPU在启动时强制设置 cs=0xf000, ip=0xfff0, 即通过段基址寻址访问内存中**0xffff0**处的内容，而**0xffff0~0xfffff**处两字节为条跳转指令，cpu跳转BIOS的起始地址执行BIOS程序。

# BIOS的任务

- POST 自检，检验主板，内存，各类外设
- 初始化各个硬件设备
- 在实模式下建立中断表、构建 BIOS 数据区、加载中断服务程序
- 读取**0磁头-0柱面-1扇区**处的MBR程序，并将其加载到**0x7c00**处。
- 跳转到**0x7c00**处执行**mbr程序** 


# MBR的任务

- 初始化cpu的更个段寄存器
- 通过软中断调用显卡驱动，在屏幕上显示字符
- 通过软中断调用磁盘驱动读取磁盘上的**loader**程序到**0x8000**
- 跳转到**loader**程序的起始地址

# 编写MBR

``` x86asm
	#include "boot.h"
	
  	.code16				//16位代码，务必加上
 	.text 
	.global _start		//链接的时候作为程序的入口地址，并提供给外部访问
	.extern boot_entry 	//告诉编译器boot_entry函数在其他文件中

// bios自动将磁盘0磁头0柱面1扇区加载到0x7c00处，而此处的代码段将放在最前面，即_start加载后为0x7c00，链接器在链接时需要用-e(或--entry)指明程序的入口地址或符号(函数或段名),若没有指明则默认为_start
_start:
	//初始化段寄存器为0,采用平坦模型访问内存，cs寄存器默认为0，其他其实也是默认为0
	mov $0, %ax
	mov %ax, %ds
	mov %ax, %ss
	mov %ax, %es
	mov %ax, %fs
	mov %ax, %gs
	mov $_start, %esp 	//初始化栈顶指针寄存器，即将0x7c00前面一段空间用做程序运行的栈空间，此处用了esp,即16位模式下用了32位模式的资源，指令前缀为反转前缀66

	//int 0x10 软中断调用显卡驱动，
	mov $0xe, %ah 	//ah = 0xe 表示显示字符
	mov $'S', %al 	// al 缓冲区记录显示内容
	int $0x10 		//进入中断程序

	mov $0xe, %ah 	
	mov $'\r', %al 	
	int $0x10 		

	mov $0xe, %ah 	
	mov $'\n', %al 	
	int $0x10 		




//int 0x13 软中断调用磁盘驱动读取磁盘
read_loader:
	mov $0x8000, %bx	// es:bx = 加载到内存中的地址,此时es = 0， 即直接将读取的内容加载到内存0x8000处 
	mov $0x2, %ah 		// ah = 0x2, al = 扇区数, 
	mov $64, %al		// 读取64个扇区, 即32kb
	mov $0x2, %cx 		// ch = 柱面，cl = 扇区编号， 在0柱面进行读取，bios从1开始为扇区编号，1扇区为mbr引导程序，所以从2扇区读取
	mov $0, %dh			// dh = 磁头，即盘面，当前从0盘面开始读取
	mov $0x80, %dl 		// dl = 驱动器，0x00~0x7f:软盘， 0x80~0xff:硬盘
	int $0x13 			//执行中断程序

	//出口参数, cf = 0表示操作成功否则失败，若成功则ah=0x00,al=传输的扇区数，否则ah=状态代码(参见功能号0x01)
	// cf为eflags寄存器的0号位，表示carry flag， 在运算过程中若最高位有进位或借位则置1
	jc read_loader		// jc/jb/jna 跳转条件为 cf = 1,即当前失败时跳转重新读取


	//加载完loader后跳转到loader入口程序执行
	jmp boot_entry

	// 链接指令：set(CMAKE_EXE_LINKER_FLAGS "-m elf_i386 -Ttext=0x7c00 --section-start=boot_end=0x7dfe")，将boot_end节加载的起始地址设为0x7dfe,加上之后的0x55,0xaa两个字节，刚好为512字节，即一个扇区的大小，0x55aa为mbr的标记
	
	.section boot_end, "ax"
	
boot_sig: .byte 0x55, 0xaa

```


# boot_entry方法跳转到loader程序

系统引导部分，启动时由硬件加载运行，然后完成对二级引导程序loader的加载boot扇区容量较小，仅512字节。由于mbr占用了不少字节，导致其没有多少空间放代码， 所以功能只能最简化,并且要开启最大的优化-os

```c
/**
 * 系统引导部分，启动时由硬件加载运行，然后完成对二级引导程序loader的加载
 * boot扇区容量较小，仅512字节。由于mbr占用了不少字节，导致其没有多少空间放代码，
 * 所以功能只能最简化,并且要开启最大的优化-os
 *
 */

__asm__(".code16gcc"); //生成16位模式下运行的程序(即偏移地址为16位大小)，但大小限制在64kb

#include "boot.h" 

#define LOADER_START_ADDR 0x8000 //loader在内存中的地址


/**
 * Boot的C入口函数
 * 只完成一项功能，即从磁盘找到loader文件然后加载到内容中，并跳转过去
 */
void boot_entry(void) {
    ((void (*)(void))LOADER_START_ADDR)(); //用函数指针直接调用0x8000处的函数
} 
```



