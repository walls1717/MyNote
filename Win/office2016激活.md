# KMS 命令激活

1. 首先查看Office2016安装目录在哪里，如果是默认安装，没有修改路径，是在 `C:\Program Files\Microsoft Office\Office16 `目录下，64位系统装32位office路径是 `C:\Program Files (x86)\Microsoft Office\Office16`，具体路径还得自行查看；

   ![image-20200809132805374](https://gitee.com/walls1717/images/raw/master/202210121924036.png)

2. 接着右键点击开始图标，选择【Windows PowerShell(管理员)】，或者【命令提示符(管理员)】；

   ![image-20200809132830948](https://gitee.com/walls1717/images/raw/master/202210121924037.png)

3. 打开命令提示符，复制这个命令，在命令窗口鼠标右键，会自动粘贴，按回车进入office2016安装路径，如果你不是在这个目录，需手动修改，目录错误，后面就无法执行激活操作；

   * 32位系统装32位office或者64位系统装64位office命令：

     `cd "C:\Program Files\Microsoft Office\Office16"`

   * 64位系统装32位office命令：

     `cd "C:\Program Files (x86)\Microsoft Office\Office16"`

4. 接着复制下面这个命令，在命令窗口鼠标右键自动粘贴命令，按回车执行，安装office2016专业增强版密钥，**如果提示无法找到脚本文件，说明第3步打开的路径错误，需要执行另一个命令**；

   ```
   cscript ospp.vbs /inpkey:XQNVK-8JYDB-WJ9W3-YJ8YR-WFG99
   
   【Office Professional Plus 2016：XQNVK-8JYDB-WJ9W3-YJ8YR-WFG99】
   
   【Office Standard 2016：JNRGM-WHDWX-FJJG3-K47QV-DRTFM】
   ```

   ![image-20200809133008723](https://gitee.com/walls1717/images/raw/master/202210121924038.png)

5. 接着复制这个命令，右键自动粘贴，按回车执行，设置kms服务器；

   `cscript ospp.vbs /sethst:kms.03k.org`

6. 最后执行这个命令，按回车激活office2016；

   `cscript ospp.vbs /act`

   ![image-20200809133134184](https://gitee.com/walls1717/images/raw/master/202210121924039.png)

7. 如果要查询office2016激活状态，执行这个命令。

   `cscript ospp.vbs /dstatus`

   ![image-20200809133116108](https://gitee.com/walls1717/images/raw/master/202210121924040.png)

   也可以记录下来自己的 SKU ID ，输入 `slmgr /xpr d450596f-894d-49e0-966a-fd39ed4c4c64` 查看具体到期时间。

   ![image-20200809133638841](https://gitee.com/walls1717/images/raw/master/202210121924041.png)

   ![image-20200809133620009](https://gitee.com/walls1717/images/raw/master/202210121924042.png)

## 激活密钥（可以尝试）

XNTT9-CWMM3-RM2YM-D7KB2-JB6DV

BHXN7-MQB36-MTHQ4-8MHKV-CYT97

3XJTG-YNBMY-TBH9M-CWB2Y-YWRHH

6TCQ3-NBBJ2-RTJCM-HFRKV-G6PQV

CGGR9-NQYC7-KRRGM-K4Y8J-XW3K7

3GXXR-NT7BJ-9DRBB-M9FYC-CKCQV