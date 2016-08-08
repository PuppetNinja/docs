# yocto_errors

tags: yocto note

---

 /home/tab/work/yocto/build/guitar/tmp/work-shared/guitar/kernel-source/arch/arm/include/asm/memory.h: In function '__virt_to_idmap':
| /home/tab/work/yocto/build/guitar/tmp/work-shared/guitar/kernel-source/arch/arm/include/asm/memory.h:242:6: error: 'arch_virt_to_idmap' undeclared (first use in this function)
|   if (arch_virt_to_idmap)
|       ^
| /home/tab/work/yocto/build/guitar/tmp/work-shared/guitar/kernel-source/arch/arm/include/asm/memory.h:242:6: note: each undeclared identifier is reported only once for each function it appears in
| /home/tab/work/yocto/build/guitar/tmp/work-shared/guitar/kernel-source/arch/arm/include/asm/memory.h:243:10: error: implicit declaration of function 'arch_virt_to_idmap' [-Werror=implicit-function-declaration]
|    return arch_virt_to_idmap(x);
|           ^
| cc1: some warnings being treated as errors
| make[3]: *** [arch/arm/kernel/asm-offsets.s] Error 1
| make[2]: *** [prepare0] Error 2
| make[1]: *** [sub-make] Error 2
| make: *** [all] Error 2
| WARNING: exit code 1 from a shell command.
| ERROR: oe_runmake failed
| ERROR: Function failed: do_compile (log file is located at /home/tab/work/yocto/build/guitar/tmp/work/guitar-poky-linux-gnueabi/linux-yocto-custom/3.10.94+gitAUTOINC+80f00977c6-r0/temp/log.do_compile.647)
ERROR: Task 55 (/home/tab/work/yocto/poky/meta-guitar/recipes-kernel/linux/linux-yocto-custom.bb, do_compile) failed with exit code '1'
NOTE: Tasks Summary: Attempted 1362 tasks of which 1277 didn't need to be rerun and 1 failed.
Waiting for 0 running tasks to finish:

Summary: 1 task failed:
  /home/tab/work/yocto/poky/meta-guitar/recipes-kernel/linux/linux-yocto-custom.bb, do_compile
Summary: There was 1 WARNING message shown.
Summary: There were 2 ERROR messages shown, returning a non-zero exit code.
➜  guitar    

---------------
# sssssssssssss

 <fredcadete> you could add your own .bb file in your layer, and then use some mechanism like PREFERRED_VERSION_u-boot or PREFERRED_PROVIDER_virtual/bootloader to force your version

<redengin> the machconfig inside the bsp layer is usually how you control uboot configuration

<gGary Thomas> Try looking at some of the <MACHINE>.conf files in meta-fsl-arm
where you should be able to see how to set up the U-Boot build.

