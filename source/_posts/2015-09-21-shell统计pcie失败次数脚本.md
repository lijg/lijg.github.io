---
title: shell统计pcie失败次数脚本
date: 2015-09-21 19:08
tags: PCIe
categories: Linux
---

shell脚本中，统计pcie检测失败次数。

```
# 记录文件
PASS=/root/pass.txt
FAIL=/root/fail.txt

# FPGA设备VID
VID="10ee"

# 重启延时时间
BOOTDLY=3
            
# 如不存在则创建文件                                               
touch ${PASS}                                                                   
touch ${FAIL}                                                                   
                 
# 统计已有成功和失败次数                                                               
PASS_CNT=`wc -l ${PASS} | awk '{print $1}'`                                     
FAIL_CNT=`wc -l ${FAIL} | awk '{print $1}'`                                     
                    
# 打印次数                            
echo "===================="                                                     
echo "PASS: ${PASS_CNT}, FALI: ${FAIL_CNT}"                                     
echo "===================="                                                     

# 加载PCIE驱动
echo 1 > /sys/deferred_pcie/write 

# 检测FPGA                                                  
lspci | grep $VID                    

# 根据grep的结果判断是否有FPGA设备
if [ $? -eq 0 ]; then                                                           
    echo `date +"%Y-%m-%d %H:%M:%S"` `expr ${PASS_CNT} + 1` | tee -a ${PASS}    
else                                                                            
    echo `date +"%Y-%m-%d %H:%M:%S"` `expr ${FAIL_CNT} + 1` | tee -a ${FAIL}    
fi                                                 

# 延时BOOTDLY秒，如果没有任何输入则重启，否则暂停
read -p "Press Any Key to STOP!!! " -t $BOOTDLY Continue                           
                                                                            
if [ $? -eq 1 ]; then                                                       
    reboot                                                                  
fi                             
```