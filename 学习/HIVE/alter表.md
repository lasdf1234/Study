ALTER表
https://www.cnblogs.com/shujuxiong/p/9766639.html
ras-api的编辑表对应的alter语句
alter table replace columns(...
这种alter语句可以替换所有的列，符合ras-api的编辑表方法
ex.
alter table test replace columns(a int)

注意，alter表，必须先use 数据库，否则会报错
![](https://github.com/lasdf1234/Study/tree/master/%E5%AD%A6%E4%B9%A0/HIVE/image/111.png)

注意：
alter表，分区是不能编辑的，除非把表的结构TableType改为VIRTUAL_VIEW才可以
