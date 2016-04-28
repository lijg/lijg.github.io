---
title: Android进入Recovery模式流程分析
date: 2014-11-14 19:52:14
tags: Recovery
categories: Android
---

# 1 介绍
Recovery模式指的是一种可以对安卓机内部的数据或系统进行修改的模式，也叫工程模式。在这个模式下我们可以刷入新的安卓系统，或者对已有的系统进行备份或升级，也可以在此恢复出厂设置。在IMX6开发板上要进入Recovery模式，可以在关机时同时按下关机键+音量下进入。
Fastboot是一种比recovery更底层的刷机模式。就是使用USB数据线连接手机的一种刷机模式。相对于用Recovery模式卡刷来说，线刷更可靠，安全。在官方IMX6开发板上只能通过adb命令行进入（adb reboot bootloader），在此修改为在关机时同时按下关机键+音量下进入。

# 2 Bootloader
## 2.1 启动流程
进入recovery模式，这个工作是由bootloader完成。IMX6开发板使用的uboot。uboot每次启动的时候都要执行``uboot/lib_arm/board.c``的代码：
```
void start_armboot(void) {

...
/* 省略无关代码 */

#ifdef CONFIG_ANDROID_RECOVERY
  check_recovery_mode();    // 检查是否进入recovery模式
#endif
 
#if defined(CONFIG_CMD_NET)
  #if defined(CONFIG_NET_MULTI)
    puts ("Net:   ");
  #endif
    eth_initialize(gd->bd);
  #if defined(CONFIG_RESET_PHY_R)
    debug ("Reset Ethernet PHY\n");
    reset_phy();
  #endif
#endif
#ifdef CONFIG_FASTBOOT
  check_fastboot_mode();    // 检查是否进入fastboot模式
#endif

/* main_loop() can return to retry autoboot, if so just run it again. */
for (;;) {
  main_loop ();
}

/* NOTREACHED - no way out of command loop except booting */

```
可以看到进入recovery模式的关键是``check_recovery_mode();``函数;进入fastboot模式的关键是``check_fastboot_mode();``函数。而且进入recovery模式的优先级较高。
## 2.2 检查是否进入recovery模式
``check_recovery_mode()``位于``uboot/board/freescale/common/recovery.c``:
```
/* export to lib_arm/board.c */
void check_recovery_mode(void)
{
    if (check_key_pressing())
        setup_recovery_env();
    else if (check_recovery_cmd_file()) {
        puts("Recovery command file founded!\n");
        setup_recovery_env();
    }
}
```
可以看到有两种方法可以进入recovery模式。
## 2.3 扫描按键
``check_key_pressing()``函数也在``uboot/board/freescale/common/recovery.c``：
```
#define PRESSED_VOL_DOWN    0x01
#define PRESSED_POWER       0x02
#define RECOVERY_KEY_MASK (PRESSED_VOL_DOWN | PRESSED_POWER)

inline int test_key(int value, struct kpp_key_info *ki)
{
    return (ki->val == value) && (ki->evt == KDepress);
}

int check_key_pressing(void)
{
    struct kpp_key_info *key_info = NULL;
    int state = 0, keys, i;

    int ret = 0;

    mxc_kpp_init();
    /* due to glitch suppression circuit,
       wait sometime to let all keys scanned. */
    udelay(1000);
    keys = mxc_kpp_getc(&key_info);

    printf("Detecting VOL_DOWN+POWER key for recovery(%d:%d) ...\n",
        keys, keys ? key_info->val : 0);
    if (keys > 1) {
        for (i = 0; i < keys; i++) {
            if (test_key(CONFIG_POWER_KEY, &key_info[i]))
                state |= PRESSED_POWER;
            else if (test_key(CONFIG_VOL_DOWN_KEY, &key_info[i]))
                state |= PRESSED_VOL_DOWN;
        }
    }
    if ((state & RECOVERY_KEY_MASK) == RECOVERY_KEY_MASK)
        ret = 1;
    if (key_info)
        free(key_info);
    return ret;
}
```
通过代码可以看出该函数是检测POWER键和音量下键，如果同时按下则返回1;否则，返回0.
## 2.4 检查标志位和音量下
``check_recovery_cmd_file()``函数位于``uboot/board/freescale/mx6q_sabresd/mx6q_sabresd.c``：
```
int check_recovery_cmd_file(void)
{
    int button_pressed = 0; 
    int recovery_mode = 0; 

    recovery_mode = check_and_clean_recovery_flag();

    /* Check Recovery Combo Button press or not. */
    mxc_iomux_v3_setup_pad(MX6X_IOMUX(PAD_GPIO_5__GPIO_1_5));

    gpio_direction_input(GPIO_VOL_DN_KEY);

    if (gpio_get_value(GPIO_VOL_DN_KEY) == 0) { /* VOL_DN key is low assert */
        button_pressed = 1; 
        printf("Recovery key pressed\n");
    }    

    return recovery_mode || button_pressed;
}
```
该函数也有两种方法返回1：一种是启动是VOL_DN按键被按下;另一种是通过``check_and_clean_recovery_flag()``函数返回1.
## 2.5 检查并清空标志位
该函数位于``uboot/cpu/arm_cortexa8/mx6/generic.c``：
```
#define ANDROID_RECOVERY_BOOT  (1 << 7)
/* check if the recovery bit is set by kernel, it can be set by kernel
 * issue a command '# reboot recovery' */
int check_and_clean_recovery_flag(void)
{
    int flag_set = 0; 
    u32 reg; 
    reg = readl(SNVS_BASE_ADDR + SNVS_LPGPR);

    flag_set = !!(reg & ANDROID_RECOVERY_BOOT);

    /* clean it in case looping infinite here.... */
    if (flag_set) {
        reg &= ~ANDROID_RECOVERY_BOOT;
        writel(reg, SNVS_BASE_ADDR + SNVS_LPGPR);
    }    

    return flag_set;
}
```
可以看出，该函数通过读取``SNVS_BASE_ADDR+SNVS_LPGPR``寄存器的第7位来判断。如果是1,则将其置零并返回1.注释中可以看到，是kernel将其置1.
## 2.6 进入recovery模式流程图
至此，进入recovery模式的流程已完成，流程图如下：
![流程图1](http://192.168.1.15/usr/uploads/2015/06/705603735.png)

## 2.7 设置Recovery环境变量
下面分析一下``setup_recovery_env()``，看一下设置Recovery环境究竟进行了哪些动作，该函数位于``uboot/board/freescale/common/recovery.c``：
```
extern struct reco_envs supported_reco_envs[];

void setup_recovery_env(void)
{
    char *env, *boot_cmd;
    int bootdev = get_boot_device();

    printf("recovery on bootdev: %d\n", bootdev);
    boot_cmd = supported_reco_envs[bootdev].cmd;

    if (boot_cmd == NULL) {
        printf("Unsupported bootup device for recovery: dev: %d\n", bootdev);
        return;
    }   

    printf("setup env for recovery..\n");

    env = getenv("bootcmd_android_recovery");
    if (!env)
        setenv("bootcmd_android_recovery", boot_cmd);
    setenv("bootcmd", "run bootcmd_android_recovery");
}
```
从函数中可知，当用户设置了``bootcmd_android_recovery``环境变量后，使用用户自定义的变量;否则，使用supported_reco_envs[bootdev].cmd作为默认值。
现在追踪一下默认值是什么，``get_boot_device();``函数位于````，只是返回当前用的哪种方式启动，SD、MMC、I2C等。
## 2.8 获取启动设备
```
static inline void setup_boot_device(void)
{
    uint soc_sbmr = readl(SRC_BASE_ADDR + 0x4);
    uint bt_mem_ctl = (soc_sbmr & 0x000000FF) >> 4 ;
    uint bt_mem_type = (soc_sbmr & 0x00000008) >> 3;

    switch (bt_mem_ctl) {
    case 0x0: 
        if (bt_mem_type)
            boot_dev = ONE_NAND_BOOT;
        else 
            boot_dev = WEIM_NOR_BOOT;
        break;
    case 0x2: 
            boot_dev = SATA_BOOT;
        break;
    case 0x3: 
        if (bt_mem_type)
            boot_dev = I2C_BOOT;
        else 
            boot_dev = SPI_NOR_BOOT;
        break;
    case 0x4: 
    case 0x5: 
        boot_dev = SD_BOOT;
        break;
    case 0x6: 
    case 0x7: 
        boot_dev = MMC_BOOT;
        break;
    case 0x8 ... 0xf: 
        boot_dev = NAND_BOOT;
        break;
    default:
        boot_dev = UNKNOWN_BOOT;
        break;
    }    
}

enum boot_device get_boot_device(void)
{
    return boot_dev;
}
```
其中``SD_BOOT``等定义位于``uboot/include/asm-arm/arch-mx6/mx6.h``：
```
enum boot_device {
    WEIM_NOR_BOOT,
    ONE_NAND_BOOT,
    PATA_BOOT,
    SATA_BOOT,
    I2C_BOOT,
    SPI_NOR_BOOT,
    SD_BOOT,
    MMC_BOOT,
    NAND_BOOT,
    UNKNOWN_BOOT,
    BOOT_DEV_NUM = UNKNOWN_BOOT,
};
```
因此，当使用SD卡启动时，``get_boot_device() == 6``;MMC启动时，``get_boot_device() == 7``.
## 2.9 默认环境变量
返回``setup_recovery_env()``函数中，``supported_reco_envs``的定义位于:``/uboot/cpu/arm_cortexa8/mx6/generic.c``:
```
struct reco_envs supported_reco_envs[BOOT_DEV_NUM] = {
    {    
        .cmd = CONFIG_ANDROID_RECOVERY_BOOTCMD_MMC,
        .args = CONFIG_ANDROID_RECOVERY_BOOTARGS_MMC,
    },   
    {    
        .cmd = NULL,
        .args = NULL,
    },   
    {    
        .cmd = NULL,
        .args = NULL,
    },   
    {    
        .cmd = NULL,
        .args = NULL,
    },   
    {    
        .cmd = CONFIG_ANDROID_RECOVERY_BOOTCMD_MMC,
        .args = CONFIG_ANDROID_RECOVERY_BOOTARGS_MMC,
    },   
    {    
        .cmd = CONFIG_ANDROID_RECOVERY_BOOTCMD_MMC,
        .args = CONFIG_ANDROID_RECOVERY_BOOTARGS_MMC,
    },   
    {    
        .cmd = CONFIG_ANDROID_RECOVERY_BOOTCMD_SD,
        .args = CONFIG_ANDROID_RECOVERY_BOOTARGS_MMC,
    },   
    {    
        .cmd = CONFIG_ANDROID_RECOVERY_BOOTCMD_MMC,
        .args = CONFIG_ANDROID_RECOVERY_BOOTARGS_MMC,
    },   
#ifdef CONFIG_CMD_NAND
    {    
        .cmd = CONFIG_ANDROID_RECOVERY_BOOTCMD_NAND,
        .args = CONFIG_ANDROID_RECOVERY_BOOTARGS_NAND,
    },   
#else
    {    
        .cmd = NULL,
        .args = NULL,
    },   
#endif
};
```
可以看出，从SD卡启动时，cmd返回``CONFIG_ANDROID_RECOVERY_BOOTCMD_SD``，该宏定义位于``include/configs/mx6q_sabresd_android.h``：
```
#define CONFIG_ANDROID_RECOVERY_BOOTCMD_MMC  \
    "booti mmc3 recovery"
#define CONFIG_ANDROID_RECOVERY_BOOTCMD_SD  \
    "booti mmc2 recovery"
```
## 2.10 设置Recovery环境流程图
设置Recovery启动环境的流程图如下所示：
![流程图2](http://192.168.1.15/usr/uploads/2015/06/579893692.png)

# 3 内核
在uboot中，启动时会检查SNVS_LPGPR[7]是否为1。如果是则将其清空并进入Recovery模式。
## 3.1 SNVS_LGPR读写
SNVS(Secure Non-Volatile Storage, 安全非易失存储)是IMX6中的一部分非易失的存储器，即使系统掉电也可以保存。正好适合用来防治这个标志位。在IMX6的datasheet中关于该位的介绍如下：
> The SNVS _LP General Purpose Register provides a 32 bit read write register, which can
be used by any application for retaining 32 bit data during a power-down mode.

SNVS_LPGPR[7]是由kernel在需要时置1的，代码位于``kernel/arch/arm/mach-mx6/system.c``：
```
#define ANDROID_RECOVERY_BOOT  (1 << 7)
#define ANDROID_FASTBOOT_BOOT  (1 << 8)
void do_switch_recovery(void)
{
    u32 reg;

    reg = __raw_readl(MX6Q_SNVS_BASE_ADDR + SNVS_LPGPR);
    reg |= ANDROID_RECOVERY_BOOT;
    __raw_writel(reg, MX6Q_SNVS_BASE_ADDR + SNVS_LPGPR);
}
```
从函数可以看出，调用时会将SNVS_LPGPR[7]置1.
## 3.2 重启流程
当用户输入reboot命令重启时，会使用系统调用``sys_reboot``，位于``kernel/kernel/sys.c``：
```
/*
 * Reboot system call: for obvious reasons only root may call it,
 * and even root needs to set up some magic numbers in the registers
 * so that some mistake won't make this reboot the whole machine.
 * You can also set the meaning of the ctrl-alt-del-key here.
 *
 * reboot doesn't sync: do that yourself before calling this.
 */
SYSCALL_DEFINE4(reboot, int, magic1, int, magic2, unsigned int, cmd,
        void __user *, arg)
{
    char buffer[256];
    int ret = 0;

    /* We only trust the superuser with rebooting the system. */
    if (!capable(CAP_SYS_BOOT))
        return -EPERM;

    /* For safety, we require "magic" arguments. */
    if (magic1 != LINUX_REBOOT_MAGIC1 ||
        (magic2 != LINUX_REBOOT_MAGIC2 &&
                    magic2 != LINUX_REBOOT_MAGIC2A &&
            magic2 != LINUX_REBOOT_MAGIC2B &&
                    magic2 != LINUX_REBOOT_MAGIC2C))
        return -EINVAL;

    /* Instead of trying to make the power_off code look like
     * halt when pm_power_off is not set do it the easy way.
     */
    if ((cmd == LINUX_REBOOT_CMD_POWER_OFF) && !pm_power_off)
        cmd = LINUX_REBOOT_CMD_HALT;

    mutex_lock(&reboot_mutex);
    switch (cmd) {
    case LINUX_REBOOT_CMD_RESTART:
        kernel_restart(NULL);
        break;

    case LINUX_REBOOT_CMD_CAD_ON:
        C_A_D = 1;
        break;

    case LINUX_REBOOT_CMD_CAD_OFF:
        C_A_D = 0;
        break;

    case LINUX_REBOOT_CMD_HALT:
        kernel_halt();
        do_exit(0);
        panic("cannot halt");

    case LINUX_REBOOT_CMD_POWER_OFF:
        kernel_power_off();
        do_exit(0);
        break;

    case LINUX_REBOOT_CMD_RESTART2:
        if (strncpy_from_user(&buffer[0], arg, sizeof(buffer) - 1) < 0) {
            ret = -EFAULT;
            break;
        }
        buffer[sizeof(buffer) - 1] = '\0';

        kernel_restart(buffer);
        break;

#ifdef CONFIG_KEXEC
    case LINUX_REBOOT_CMD_KEXEC:
        ret = kernel_kexec();
        break;
#endif

#ifdef CONFIG_HIBERNATION
    case LINUX_REBOOT_CMD_SW_SUSPEND:
        ret = hibernate();
        break;
#endif

    default:
        ret = -EINVAL;
        break;
    }
    mutex_unlock(&reboot_mutex);
    return ret;
}
```
从函数可以看出，当用户输入reboot recovery时，会调用``kernel_restart("recovery")``.
``kernel_restart``又调用``machine_restart()``：
```
/**
 *  kernel_restart - reboot the system
 *  @cmd: pointer to buffer containing command to execute for restart
 *      or %NULL
 *
 *  Shutdown everything and perform a clean reboot.
 *  This is not safe to call in interrupt context.
 */
void kernel_restart(char *cmd)
{
    kernel_restart_prepare(cmd);
    if (!cmd)
        printk(KERN_EMERG "Restarting system.\n");
    else 
        printk(KERN_EMERG "Restarting system with command '%s'.\n", cmd);
    kmsg_dump(KMSG_DUMP_RESTART);
    machine_restart(cmd);
}
```
``kernel_restart()``调用``machine_restart``,位于``kernel/arch/arm/kernel/process.c``:
```
void arm_machine_restart(char mode, const char *cmd)
{
    /* Flush the console to make sure all the relevant messages make it
     * out to the console drivers */
    arm_machine_flush_console();

    /* Disable interrupts first */
    local_irq_disable();
    local_fiq_disable();

    /*  
     * Tell the mm system that we are going to reboot -
     * we may need it to insert some 1:1 mappings so that
     * soft boot works.
     */
    setup_mm_for_reboot(mode);

    /* Clean and invalidate caches */
    flush_cache_all();

    /* Turn off caching */
    cpu_proc_fin();

    /* Push out any further dirty data, and ensure cache is empty */
    flush_cache_all();

    /*  
     * Now call the architecture specific reboot code.
     */
    arch_reset(mode, cmd);

    /*  
     * Whoops - the architecture was unable to reboot.
     * Tell the user!
     */
    mdelay(1000);
    printk("Reboot failed -- System halted\n");
    while (1);
}
```
在这里调用了``arch_reset(mode, cmd=“recovery”)``，位于``kernel/arch/arm/plat-mxc/system.c``:
```
static void arch_reset_special_mode(char mode, const char *cmd)
{
    if (strcmp(cmd, "download") == 0)
        do_switch_mfgmode();
    else if (strcmp(cmd, "recovery") == 0)
        do_switch_recovery();
    else if (strcmp(cmd, "fastboot") == 0)
        do_switch_fastboot();
}

/*
 * Reset the system. It is called by machine_restart().
 */
void arch_reset(char mode, const char *cmd)
{
    unsigned int wcr_enable;

    arch_reset_special_mode(mode, cmd);
    
    ...
    /* 省略无关代码 */
}
```
在这里，终于调用到了``do_switch_recovery()``，将SNVS_LGPR[7]置为1.
## 3.3 调用关系
![流程图3](http://192.168.1.15/usr/uploads/2015/06/2967304704.png)

完。