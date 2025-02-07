# U-Boot-Boot-Process

第一行程式碼從u-boot.lds開始，u-boot.map為uboot映射文件，其中 __image_copy_start為uboot起始的執行地址0x87800000，因此uboot需要放到sd卡中的0x87800000才能執行。

 - _start 設定中斷向量表，設定SVC mode, 並將_start=0x87800000設定到VBAR中斷向量偏移中
   
 - 初始化cp15

 - 設定sp指針到內部ocram 0x00900000 (128 kB)，初始化內部ram，地址為0x00900000~0x0091FFFF，初始化後的內存分配圖

 ![ ](https://drive.google.com/uc?export=view&id=1PX3JBC49hQ6eqfR8bKEJCnapuprLy_cE)

 - 進入main函數，main會再將sp指針偏移

 ![ ](https://drive.google.com/uc?export=view&id=1MBx_N2X7tAe9nYsOtSi-JZbHAZweEylE)

 - 將sp指針指向外部DDR後執行main中的四個主要函數
     - board_init_f函數，初始化外部設備和gd的各成員變量，如外設時鐘、IO、UART
  
     - relocate_code函數，將uboot複製到DDR最後面的區域防止linux kernal覆蓋掉uboot，uboot對全域變量的讀取不會使用直接讀取，而是使用第三方偏移地址label進行讀取，因此在複製過程中會紀錄他的偏移量，在讀取過程會重新定位來解決執行地址和儲存地址不同的問題
  
     - relocate_vectors函數，將重新定位後的向量偏移量寫入VBAR
  
     - board_init_r函數，初始化開發板內部設定如malloc、I2C、stdio、EMMC or NAND
  
 - 3秒倒數計時結束如果沒有按按鍵會進入run_main_loop函數，run_main_loop會讀取bootdelay 和 bootcmd ，若按按鍵會進入cli_loop函數，cli_loop為命令處理函數，會將輸入的內容parsing並與.u_boot_list比對，再執行對應的do_xxx。.u_boot_list裡面儲存了所有可以執行的命令

 - 執行bootcmd，鏡像檔案ZImage中會儲存一個LINUX_ARM_ZIMAGE_MAGIC 就是 ARM Linux系統魔術數，使用bootz時會使用這個數字來判斷該地址是否有ZImage，設備樹會向linux內核傳送參數

 - 執行kernel_entry函數，進入ZImage並執行linux kernel
