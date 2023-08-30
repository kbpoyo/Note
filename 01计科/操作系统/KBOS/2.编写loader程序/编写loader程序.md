
# loader的任务

> mbr完成cpu的初始化后从磁盘的**0磁头-0柱面-2扇区**处将loader程序读取到内存的0x8000处，并跳转到该处执行。

## start.S的任务

- 生成16位模式的跳转指令，跳转到**loader_16**下执行
- 生成保护模式下第一段32位模式的程序
- 在第一段32位模式中清除cpu16位模式下残余的流水线
- 重新初始化各个寄存器为临时段描述符的选择子
- 远跳转到**loader_32**程序下执行
### start.S

```c
  	.code16					//生成16位模式下的指令
 	.text					//指定之后的内容是属于代码段
	.global _start			//声明全局变量，允其他文件使用符号_start(即地址)
	.extern loader_entry 	//告诉编译器loader_entry函数在其他文件中

_start:
	jmp loader_entry 

	.code32 //生成32位模式下的指令
	.text
	.global protect_mode_entry
	.extern load_kernel //32位loader程序入口

//保护模式的入口函数
protect_mode_entry:
	//cs寄存器已进入32位模式，还需要将其他段寄存器全部设置为32位模式
	//已经进入了保护模式，所以cpu正在执行的16位模式的流水线失效
	//重置各个段寄存器，存储选择子，0x10即指向第3个段描述符(数据段)
	//第三个段描述符所指的段基址为0，所以继续平坦模式
	//且第三个段描述符的属性位的G位设置为1，标志了该段为32位程序，其他位也设置了正确值
	//临时的内核数据段描述符:{0xffff, 0x0000, 0x9200, 0x00cf}
	mov $0x10, %ax
	mov %ax, %ds
	mov %ax, %ss
	mov %ax, %es
	mov %ax, %fs
	mov %ax, %gs

	//跳转到32位的loader程序中，jmp指令可以清空流水线
	jmp $8, $load_kernel //直接跳转 cs:8， ip:load_kernel, 8为1000(b), 即选择此时GDT中的第2个段描述符(代码段)

```

## loader_16的任务

> loader_16.c 将会被编译为16位模式的指令，运行在实模式下。

- 检测系统当前空闲并可用的内存块
- 记录可用的内存块的起始地址和大小
- 进入保护模式
	- 关闭中断，防止cpu响应中断
	- 打开A20地址线，使cpu可以访问更大的地址空间(32位为4GB)
	- 构建加载临时的GDT表，使进入保护模式后能正确寻址
	- 设置CR0寄存器，标志CPU在保护模式下运行
- 远跳转到保护模式下的第一段程序下执行

### loader.h

> **loader.h**中声明了用于检测系统可用内存的结构体，其中的字段用于记录cpu每次探测后回填的结果数据。


```c
#ifndef LOADER_H
#define LOADER_H

#include "common/boot_info.h"
#include "common/cpu_instr.h"
#include "common/types.h"

//用于检测有效内存块的结构体
typedef struct SMAP_entry {
 
	uint32_t BaseL;   // base address uint64_t, 即基础寻址空间为64位
	uint32_t BaseH;
	uint32_t LengthL; // length uint64_t
	uint32_t LengthH;
	uint32_t Type;    // entry Type，值为1时表明为我们可用的RAM空间
	uint32_t ACPI;    // extended, bit0=0时表明此条目应当被忽略
 
}__attribute__((packed)) SMAP_entry_t; //不对该结构体进行字节对齐优化

/**
 * @brief  进入保护模式
 * 
 */
void protect_mode_entry(void);

#endif

```

### loader_16.c

> 探测并记录系统可用内存块后，完成进入保护模式的准备步骤，并进入保护模式，跳转到保护模式下第一段程序执行。

```c
//生成16位模式下的程序代码
__asm__(".code16gcc"); 

#include "loader.h"

//实例化记录系统信息的结构体
boot_info_t boot_info;

/**
 * @brief  
 * 在显示器上打印字符串msg 
 * @param msg 
 */
static void show_msg(const char* msg) {
     char c;
     while ((c = *(msg++)) != '\0') {          
          //asm()会对内联汇编进行优化，可能会导致不确定的结果，所以使用__asm__ __volatile__()
          __asm__ __volatile__(
               "mov $0xe, %%ah;"
               " mov %[rgs], %%al;"
               "int $0x10"::[rgs]"r"(c) //"r" 将rgs映射到任意寄存器, 
                                        //且有了输入参数后，真正的寄存器要用%%前缀进行访问
          );
     }
     
}

/**
 * @brief  
 * 检测系统当前可用的内存块的地址和大小
 */
static void detect_memory(void) {
     show_msg("try to detect memory:\r\n");

     SMAP_entry_t smap_entry;//记录每一次探测的结果
     
     boot_info.ram_region_count = 0; //将有效内存卡数量初始化为0

     //传入参数
     SMAP_entry_t *entry = &smap_entry; //记录的信息将回填到结构体中

     //传出参数
      uint32_t signature = 0, bytes =0, contID = 0; //分别为ax, cx, bx的传出参数

     //逐个检测内存块，有效则装入数组中
     for (int i = 0; i < BOOT_RAM_REGION_MAX; ++i) {

          //调用内联汇编进行一次内存块探测
          __asm__ __volatile__ ("int  $0x15" 
				: "=a"(signature), "=c"(bytes), "=b"(contID) //传出参数
				: "a"(0xE820), "b"(contID), "c"(24), "d"(0x534D4150), "D"(entry)); //传入参数

          //判断所探测的内存块是否有效
          if (signature != 0x534d4150) { //无效直接退出
               show_msg("failed\r\n");
               return;
          }

          if (bytes > 20 && (entry->ACPI & 0x0001) == 0) continue; //ACPI位为0，则内存块无效应当忽略	
		
          if (entry->Type == 1) { //Type = 1 当前内存块有效
               //由于内存寻址空间较小，读取低32位即可
               boot_info.ram_region_cfg[boot_info.ram_region_count].start = entry->BaseL;  
               boot_info.ram_region_cfg[boot_info.ram_region_count].size = entry->LengthL;
               boot_info.ram_region_count++;
          }

          if (contID == 0) break; //contID为0则探测结束
     } 

     show_msg("detect success\r\n");

}


//临时的全局描述符表GDT,里面每一个段描述都是八个字节大小
uint16_t gdt_table[][4] = {
     {0, 0, 0, 0},                      //第0个段描述符无效，防止未初始化选择子时选中该段描述符
     {0xffff, 0x0000, 0x9a00, 0x00cf},  //临时的内核代码段描述符
     {0xffff, 0x0000, 0x9200, 0x00cf},  //临时的内核数据段描述符
};


/**
 * @brief  进入保护模式
 * 
 */
static void enter_protect_mode(void){
     //1.关闭中断，设置eflags对应的位即可，在进入保护模式时不接收中断调用
     cli();

     //2.以此打开A20Gate, 读取92端口，并将其第二位设置为1
     uint8_t v = inb(0x92);
     outb(0x92, v | 0x2);

     //3.加载全局描述符
     lgdt((uint32_t)gdt_table, sizeof(gdt_table));

     //4.开启保护模式的使能位，设置cr0寄存器的第0位PE设置为1
     uint32_t cr0 = read_cr0();
     write_cr0(cr0 | 0x1);

     //5.远跳转到32位的loader程序，并清空原来的16位指令流水线
     far_jump(8, (uint32_t)protect_mode_entry);
}

/**
 * @brief 完成实模式下为进入保护模式的初始化工作 
 * 
 */
void loader_entry(void) {
     show_msg("..........loading.........\r\n");

     //1.检测可用内存块
     detect_memory();         
     
     //2.进入保护模式
     enter_protect_mode();    
     
}


```

## loader_32的作用

- 用LBA模式，将磁盘上内核的elf文件读取到内存的指定位置
- 对内核的elf文件进行解析，将可加载的段拷贝到elf文件指定的位置
- 跳转到内核程序执行

### loader_32.c

```c
#include "loader.h"
#include "common/elf.h"
extern boot_info_t boot_info;

/**
 * @brief  以LBA模式读取磁盘(有48位PIO与28位PIO, 此处使用48位PIO)
 * 
 * @param sector 读取的分区号
 * @param sector_count 读取的分区数量
 * @param buf 缓冲区第一个字节的地址
 */
static void read_disk(uint32_t sector, uint16_t sector_count, uint8_t *buf) {
    
    //1.设置以LBA模式进行读取，即将磁盘看作一片连续的扇区
    outb(0x1F6, 0xE0 | (0x0 << 4));              //0xE0 将寄存器第6位置1进入LBA模式，0x0将第4位置0指定驱动器号为主盘
                                                            //现在一个通道上只有一个盘，默认当作主盘
                    
    //2.初始化各个端口寄存器的高8位
    outb(0x1F2, (uint8_t)(sector_count >> 8));  //读取扇区数的高8位
                                                //6字节LBA值，先初始化第456个字节
                                                //我暂时只用到了4个字节的LBA值, 所以第5 6个字节置0即可
    outb(0x1F3, (uint8_t)(sector >> 24));       //LBA4 
    outb(0x1F4, 0x00);                          //LBA5
    outb(0x1F5, 0x00);                          //LBA6

    //3.初始化各个端口寄存器的低8位
    outb(0x1F2, (uint8_t)sector_count);         //读取扇区数的低8位
    outb(0x1F3, (uint8_t)sector);               //LBA1
    outb(0x1F4, (uint8_t)(sector >> 8));        //LBA2
    outb(0x1F5, (uint8_t)(sector >> 16));       //LBA3

    //4.将读取扇区命令 （0x24） 发送到端口 0x1F7
    outb(0x1F7, 0x24);

    //5.读取状态端口寄存器，判断是否可读取,若可以则读取，否则阻塞等待
    uint16_t *data_buf = (uint16_t*) buf;       //数据缓冲区，以后每次会读取16位数据
    while (sector_count--) {
        while ((inb(0x1F7) & 0x88) != 0x8) {};  //取出状态寄存器3位和7位
                                                //若!=0x8即DRQ位(3位)为0，即非就绪状态
                                                //或者BSY(7位)为1，即忙碌状态，都不可读取

        for (int i = 0; i < SECTOR_SIZE / 2; ++i) {
            *(data_buf++) = inw(0x1F0);          //从数据端口寄存器中读取16位数据，即2个字节
        }
        
    }
    
}

/**
 * @brief   解析内存中地址为 file_start_addr 处的elf文件头，并将.text,.rodata,.data,.bss段
 *          拷贝到内存中的目的地址，elf文件会将(.text, .rodata)，(.data, .bss)各放在一个段
 * 
 * @param file_start_addr 已加载到内存中的elf文件的起始地址
 * @return uint32_t   返回所拷贝的程序段的入口地址
 */
static uint32_t reload_elf_file(uint8_t *file_start_addr ) {
    //1.强转为Elf32_Ehdr, 使该结构体可以访问到之后的52字节, 即elf header所包含的内容
    Elf32_Ehdr *elf_hdr = (Elf32_Ehdr*)file_start_addr; 

    //2.判断为访问的内存区域是否为elf文件,若不是则直接返回0
    if (elf_hdr->e_ident[0] != 0x7f 
        || elf_hdr->e_ident[1] != 'E' 
        || elf_hdr->e_ident[2] != 'L' 
        || elf_hdr->e_ident[3] != 'F') return 0;

    //3.解析program hear即段的头信息数组
    for (int i = 0; i < elf_hdr->e_phnum; ++i) {
        //4.以 Elf32_Phdr 内存大小为单位，逐个读取段的头信息，e_phoff 为其起始地址相对于elf文件的偏移量
        Elf32_Phdr *phdr = (Elf32_Phdr*)(file_start_addr + elf_hdr->e_phoff) + i;

        //5.判断是否为可加载的程序段, 不是则直接忽略
        if (phdr->p_type != PT_LOAD) continue;

        //6.找到可加载程序段在内存中的位置, p_offset为其起始地址相对于elf文件的偏移量
        uint8_t *src = file_start_addr + phdr->p_offset;

        //7.加载到内存中的目的地址
        uint8_t *dest = (uint8_t*)phdr->p_vaddr;

        //8.逐个字节拷贝到对应位置
        for (int i = 0; i < phdr->p_filesz; ++i) {
            *(dest++) = *(src++);
        }

        //9.若加载的是.data段和.bss段，则.bss段因为未初始化，所以在elf文件中并为为其预留空间
        //但在内存中需要为其分配空间，还需要分配空间大小为 (p_memsz - p_filesz),置0即可
        for (int i = 0; i < phdr->p_memsz - phdr->p_filesz; ++i) {
            *(dest++) = 0;
        }
    }

    //10.返回解析的程序段的入口地址
    return elf_hdr->e_entry;
}

/**
 * @brief  读取错误代码进行异常处理
 * 
 * @param err_code 
 */
static void die(uint8_t err_code) {
    //TODO:暂时什么都不做，直接卡死电脑
    for(;;){}
}

/**
 * @brief  32位loader程序的入口函数
 * 
 */
void load_kernel(void) {
    //1.从磁盘100号分区读取内核，一共读取250kb，到内存中地址为SYS_KERNEL_LOAD_ADDR的地方
    read_disk(100, 500, (uint8_t*)SYS_KERNEL_LOAD_ADDR);            

    //2.解析内存中地址为SYS_KERNEL_LOAD_ADDR处的elf文件头     
    uint32_t kernel_entry = reload_elf_file((uint8_t*)SYS_KERNEL_LOAD_ADDR); 
    
    //3.若函数执行失败，返回的入口地址为0，进行错误处理
    if (kernel_entry == 0) { 
        die(-1);
    }               

    //4.将boot_info记录的信息传递给已拷贝到确定内存中的内核初始化函数
    ((void(*)(boot_info_t*))kernel_entry)(&boot_info);    
}

```

