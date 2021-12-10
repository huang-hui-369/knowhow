# Apache POI

Apache POI 官网： http://poi.apache.org/index.html 。

结构：

HSSF － 提供读写Microsoft Excel格式档案的功能。excel 2003 版本（块，但是不能读写超过65536行记录）

XSSF － 提供读写Microsoft Excel OOXML格式档案的功能。excel 207 版本（慢，但是能读写超过65536行记录）

SXSSF － 提供读写Microsoft Excel OOXML格式档案的功能。excel 207 版本(XSSF升级版，比XSSF快，能读写超过65536行记录)

HWPF － 提供读写Microsoft Word格式档案的功能。

HSLF － 提供读写Microsoft PowerPoint格式档案的功能。

HDGF － 提供读写Microsoft Visio格式档案的功能。

# EasyExcel

GitHub 地址： https://github.com/alibaba/easyexcel

EasyExcel 官网： https://www.yuque.com/easyexcel/doc/easyexcel

EasyExcel 是阿里巴巴开源的一个 excel处理框架，以使用简单、节省内存著称。

EasyExcel 能大大减少内存占用的主要原因是在解析 Excel 时没有将文件数据一次性全部加载到内存中，而是从磁盘上一行行读取数据，逐个解析。