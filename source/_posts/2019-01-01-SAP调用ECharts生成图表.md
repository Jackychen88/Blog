---
title: SAP调用ECharts生成图表
date: 2019-01-01 11:03:26
tags: [ABAP,EChart]
---

# 获取ECharts
你可以通过以下几种方式获取 ECharts：
1. 从官网下载界面选择你需要的版本下载，根据开发者功能和体积上的需求，我们提供了不同打包的下载，如果你在体积上没有要求，可以直接下载完整版本。开发环境建议下载源代码版本，包含了常见的错误提示和警告。
2. 在 ECharts 的 GitHub 上下载最新的 release 版本，解压出来的文件夹里的 dist 目录里可以找到最新版本的 echarts 库。
3. 通过 npm 获取 echarts，npm install echarts --save，详见“在 webpack 中使用 echarts”
4. **cdn 引入，你可以在 [cdnjs](https://cdnjs.com/libraries/echarts)，[npmcdn](https://unpkg.com/echarts@4.2.0-rc.2/dist/) 或者国内的 [bootcdn](https://www.bootcdn.cn/echarts/) 上找到 ECharts 的最新版本。**

# 引入 ECharts
ECharts 3 开始不再强制使用 AMD 的方式按需引入，代码里也不再内置 AMD 加载器。因此引入方式简单了很多，只需要像普通的 JavaScript 库一样用 script 标签引入。
![26](https://note.youdao.com/yws/public/resource/6d8875e5a015a2531827e3d1b3043da5/xmlnote/709391C1B57748BCAE410B6FA17A79CC/14286)
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <!-- 引入 ECharts 文件（从上记第四点的网站那个可以直接导入） -->
    <!--<script src="echarts.min.js"></script> -->
    <script src="https://cdn.bootcss.com/echarts/4.2.0-rc.2/echarts-en.common.js"></script>
</head>
</html>
```

# 绘制一个简单的图表
在绘图前我们需要为 ECharts 准备一个具备高宽的 DOM 容器。
```html
<body>
    <!-- 为 ECharts 准备一个具备大小（宽高）的 DOM -->
    <div id="main" style="width: 600px;height:400px;"></div>
</body>
```

然后就可以通过 echarts.init 方法初始化一个 echarts 实例并通过 setOption 方法生成一个简单的柱状图，下面是完整代码。
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>ECharts</title>
    <!-- 引入 echarts.js -->
    <script src="https://cdn.bootcss.com/echarts/4.2.0-rc.2/echarts-en.common.js"></script>
</head>
<body>
    <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
    <div id="main" style="width: 600px;height:400px;"></div>
    <script type="text/javascript">
        // 基于准备好的dom，初始化echarts实例
        var myChart = echarts.init(document.getElementById('main'));

        // 指定图表的配置项和数据
        var option = {
            title: {
                text: 'ECharts 入门示例'
            },
            tooltip: {},
            legend: {
                data:['销量']
            },
            xAxis: {
                data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
            },
            yAxis: {},
            series: [{
                name: '销量',
                type: 'bar',
                data: [5, 20, 36, 10, 10, 20]
            }]
        };

        // 使用刚指定的配置项和数据显示图表。
        myChart.setOption(option);
    </script>
</body>
</html>
```

效果</br>
![27](https://note.youdao.com/yws/public/resource/6d8875e5a015a2531827e3d1b3043da5/xmlnote/93C88427E117413DAB6D1141EF0D85F5/14293)

你也可以直接进入 [ECharts Gallery](http://www.echartsjs.com/gallery/editor.html?c=doc-example/getting-started) 中查看编辑示例

# EChart 结合SAP报表的显示sample
```
*&---------------------------------------------------------------------*
*& Report  YTEST_JK63
*&
*&---------------------------------------------------------------------*
*&
*&
*&---------------------------------------------------------------------*
REPORT YTEST_JK63.

CLASS LCL_HTML_SHOWER DEFINITION
  FINAL
  CREATE PRIVATE .

  PUBLIC SECTION.
    CLASS-METHODS
      FACTORY RETURNING VALUE(OR_HTML_SHOWER) TYPE REF TO LCL_HTML_SHOWER.

    METHODS
      SHOW_HTML.

  PRIVATE SECTION.
    TYPES:
      BEGIN OF TY_DATA,
        NAME TYPE ZNAME,
        QTY  TYPE char1,
      END OF TY_DATA,
      TTY_DATA TYPE STANDARD TABLE OF TY_DATA." WITH UNIQUE KEY NAME.


    DATA:
      V_HTML TYPE STRING,
      T_DATA TYPE TTY_DATA.

    METHODS:
      CONSTRUCTOR.
    METHODS
      EDIT_DATA.
ENDCLASS.

CLASS LCL_HTML_SHOWER IMPLEMENTATION.
  METHOD FACTORY.
    CREATE OBJECT OR_HTML_SHOWER.
  ENDMETHOD.

  METHOD EDIT_DATA.
    T_DATA = VALUE #(
      ( NAME = '张三' QTY = '3' )
      ( NAME = '李四' QTY = '4' )
      ( NAME = '王五' QTY = '5' )
      ( NAME = '赵六' QTY = '6' )
      ( NAME = '石七' QTY = '7' )
      ( NAME = '张八' QTY = '8' )
    ).
  ENDMETHOD.

  METHOD CONSTRUCTOR.
*<!DOCTYPE html>
*<html>
*<head>
*    <meta charset="utf-8">
*    <title>ECharts</title>
*    <!-- 引入 echarts.js -->
*<script src="https://cdn.bootcss.com/echarts/4.2.0-rc.2/echarts-en.common.js"></script>
*</head>
*<body>
*    <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
*    <div id="main" style="width: 600px;height:400px;"></div>
*    <script type="text/javascript">
*        // 基于准备好的dom，初始化echarts实例
*        var myChart = echarts.init(document.getElementById('main'));
*
*        // 指定图表的配置项和数据
*        var option = {
*            title: {
*                text: 'ECharts 入门示例'
*            },
*            tooltip: {},
*            legend: {
*                data:['销量']
*            },
*            xAxis: {
*                data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
*            },
*            yAxis: {},
*            series: [{
*                name: '销量',
*                type: 'bar',
*                data: [5, 20, 36, 10, 10, 20]
*            }]
*        };
*
*        // 使用刚指定的配置项和数据显示图表。
*        myChart.setOption(option);
*    </script>
*</body>
*</html>

    EDIT_DATA( ).
    ME->V_HTML =
|<!DOCTYPE html>| &&
|<html>| &&
|<head>| &&
|    <meta charset="utf-8">| &&
|    <title>ECharts</title>| &&
|<script src="https://cdn.bootcss.com/echarts/4.2.0-rc.2/echarts-en.common.js"></script>| &&
|</head>| &&
|<body>| &&
|    <div id="main" style="width: 600px;height:400px;"></div>| &&
|    <script type="text/javascript">| &&
|        var myChart = echarts.init(document.getElementById('main'));| &&

|        var option = \{ | &&
|            title: \{| &&
|                text: 'ECharts 入门示例'| &&
|            \},| &&
|            tooltip: \{\},| &&
|            legend: \{| &&
|                data:['销量']| &&
|            \},| &&
|            xAxis: \{| &&
|           data:[|   &&
            REDUCE string( INIT list type string
                                sep = ``
                           FOR wa in me->T_DATA
                           NEXT list = list && |{ sep }"{ wa-name }"|
                                sep = `,`
            ) &&
|]| &&
|           \},| &&
|            yAxis: \{\},| &&
|            series: [\{| &&
|                name: '销量',| &&
|                type: 'bar',| &&
|           data:[|   &&
            REDUCE string( INIT list type string
                                sep = ``
                           FOR wa in me->T_DATA
                           NEXT list = list && |{ sep }"{ wa-qty }"|
                                sep =  `,`
            ) &&
|]| &&

|            \}]| &&
|        \};| &&
|        myChart.setOption(option);| &&
|    </script>| &&
|</body>| &&
|</html>|.


  ENDMETHOD.


  METHOD SHOW_HTML.
    CL_ABAP_BROWSER=>SHOW_HTML(
      HTML_STRING  = ME->V_HTML
      BUTTONS      = CL_ABAP_BROWSER=>NAVIGATE_HTML
      CONTEXT_MENU = ABAP_TRUE
      CHECK_HTML   = ABAP_FALSE ).
  ENDMETHOD.

ENDCLASS.


START-OF-SELECTION.

  DATA:
    LO_HTML_SHOWER TYPE REF TO LCL_HTML_SHOWER.

  LO_HTML_SHOWER = LCL_HTML_SHOWER=>FACTORY( ).

  LO_HTML_SHOWER->SHOW_HTML( ).
```