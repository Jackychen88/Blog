---
title: 从REPOSRC表读取SAP程序源码
date: 2019-01-27 15:51:16
tags: [ABAP]
---

# 思考
SAP 有标准的关键key **READ REPORT**来获取程序的源码，但我很好奇这个key底层是如何工作的，很好奇SAP到底是以何种方式来保存源码的，因此有以下的调查

# 数据库
首先想到肯定SAP是把代码存放在数据库的，不管是加密的，压缩的，还是编译的肯定需要DB存储的。通过各种方法找到了源码疑似存放的数据库
+ **REPOSRC** :DATA栏位存放了代码压缩后的二进制数据
  ![REPOSRC](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190127094036.png)
+ **REPOLOAD**:LDATA和QDATA分别存放了压缩后的编译代码二进制数据和未压缩的编译代码二进制
  ![REPOLOAD](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190127094133.png)

# 还原方法
1. 首先我判断出来REPOLOAD是SAP内部编译的数据，进行了解压转换编码处理后，得到的是编译代码，无法还原成我们能理解的ABAP语言（SAP原厂内部肯定能通过编译代码反编译源码，但SAP是不开放的）
2. 那么就考虑从REPOSRC的DATA栏位来还原代码吧，信息肯定保存在DATA栏位，并且肯定是压缩过的（毕竟栏位描述就叫Compressed）那么肯定有一个压缩算法
3. 通过bing查到了[SAP源码压缩方式](https://www.daniel-berlin.de/devel/sap-dev/decompress-abap-source-code/)
   + ``The code stored in REPOSRC-DATA is actually compressed using the LZH algorithm (Lempel-Ziv plus Huffman coding), which is used by the SAP DB MaxDB database too``
   + Lempel-Ziv压缩方式具体是什么不清楚，好在外国大牛自己用C++写了个解压工具，我们直接拿来用就可以了
4. 下载[解压算法工具](https://github.com/daberlin/sap-reposrc-decompressor/releases)，解压zip文件
5. 打开Visual Sudio（我的是2013），点击文件->新建->项目创建一个win32的控制台命令程序<br>
  ![创建一个win32的控制台命令程序](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190127132150.png)
  ![空项目](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190127132321.png)
6. 在资源管理器的源文件中右键选中源文件，点击添加->现有项把刚才下载解压文件中的.cpp导入进去，然后在资源文件夹上右键同样导入刚才解压文件中lib的所有资源<br>
  ![导入解压工具](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190127132435.png)
7. 点击执行，生成exe执行文件,至此解压工具生成完毕，也可以直接[下载解压工具exe](https://www.daniel-berlin.de/wp-content/uploads/2015/08/sap-reposrc-decompressor-1.1.1.zip)<br>
  ![生成解压exe](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190127132826.png)
8. 将SAP数据库中的DATA栏位以二进制Bin的格式下载到本地文件中去，源码如下
```
*&---------------------------------------------------------------------*
*& Report ZS_REPOSRC_DOWNLOAD
*&---------------------------------------------------------------------*
*& Purpose: Download compressed source code from table REPOSRC
*& Author : Daniel Berlin
*& Version: 1.0.1
*& License: CC BY 3.0 (http://creativecommons.org/licenses/by/3.0/)
*&---------------------------------------------------------------------*

REPORT ZS_REPOSRC_DOWNLOAD.

DATA: V_FNAM TYPE RLGRAP-FILENAME,  " Local file name
      V_FILE TYPE STRING,           " Same, but as a string
      V_XSTR TYPE XSTRING,          " Source (compressed)
      V_XLEN TYPE I,                " Length of source
      T_XTAB TYPE TABLE OF X255.    " Source plugged into a table

PARAMETERS: REPORT TYPE PROGNAME DEFAULT SY-REPID           "#EC *
                  MATCHCODE OBJECT PROGNAME OBLIGATORY.

START-OF-SELECTION.

  " -- Select local file name
  WHILE V_FNAM IS INITIAL.
    V_FNAM = REPORT.

    CALL FUNCTION 'NAVIGATION_FILENAME_HELP'
      EXPORTING
        DEFAULT_PATH      = V_FNAM
        MODE              = 'S'
      IMPORTING
        SELECTED_FILENAME = V_FNAM.
  ENDWHILE.

  V_FILE = V_FNAM.

  " -- Fetch compressed source code
  SELECT SINGLE DATA INTO V_XSTR FROM REPOSRC
          WHERE PROGNAME = REPORT AND R3STATE = 'A'.

  V_XLEN = XSTRLEN( V_XSTR ).

  " -- Plug source into a table
  CALL METHOD CL_SWF_UTL_CONVERT_XSTRING=>XSTRING_TO_TABLE
    EXPORTING
      I_STREAM = V_XSTR
    IMPORTING
      E_TABLE  = T_XTAB
    EXCEPTIONS
      OTHERS   = 1.

  " -- Download to local file
  CALL FUNCTION 'GUI_DOWNLOAD'
    EXPORTING
      FILETYPE     = 'BIN'
      FILENAME     = V_FILE
      BIN_FILESIZE = V_XLEN
    TABLES
      DATA_TAB     = T_XTAB
    EXCEPTIONS
      OTHERS       = 0.
```

9. 将程序执行后获取到的文件放入解压exe工具同一个目录，Ctrl+R CMD打开命令行界面，CD至对应路径，运行以下代码后，会生成解压后的源码内容
  ![](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190127133943.png)
```
call "%VS120COMNTOOLS%"vsvars32.bat
SAP_decompress.exe YTEST_JK76 YTEST_JK76.txt-u
```
![](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190127135302.png)