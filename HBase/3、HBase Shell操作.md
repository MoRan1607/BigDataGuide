三、HBase Shell操作
---
### 1、基本操作
1）进入HBase客户端命令行  
```txt
    [drift@hadoop102 hbase]$ bin/hbase shell
```  
2）查看帮助命令  
```txt
    hbase(main):001:0> help
```  
3）查看当前数据库中有哪些表  
```txt
    hbase(main):002:0> list
```  

### 2、表的操作
1）创建表  
```txt
    hbase(main):002:0> create 'student','info'
```  
2）插入数据到表  
```txt
    hbase(main):003:0> put 'student','1001','info:sex','male'
    hbase(main):004:0> put 'student','1001','info:age','18'
    hbase(main):005:0> put 'student','1002','info:name','Janna'
    hbase(main):006:0> put 'student','1002','info:sex','female'
    hbase(main):007:0> put 'student','1002','info:age','20'
```  
3）扫描查看表数据  
```txt
    hbase(main):008:0> scan 'student'
    hbase(main):009:0> scan 'student',{STARTROW => '1001', STOPROW  => '1001'}
    hbase(main):010:0> scan 'student',{STARTROW => '1001'}
```  
4）查看表结构  
```txt
    hbase(main):011:0> describe 'student'
```  
5）更新指定字段的数据  
```txt
    hbase(main):012:0> put 'student','1001','info:name','Nick'
    hbase(main):013:0> put 'student','1001','info:age','100'
```  
6）查看“指定行”或“指定列族:列”的数据  
```txt
    hbase(main):014:0> get 'student','1001'
    hbase(main):015:0> get 'student','1001','info:name'
```  
7）统计表数据行数  
```txt
    hbase(main):021:0> count 'student'
```  
8）删除数据  
```txt
    删除某rowkey的全部数据：
    hbase(main):016:0> deleteall 'student','1001'
    删除某rowkey的某一列数据：
    hbase(main):017:0> delete 'student','1002','info:sex'
```  
9）清空表数据  
```txt
    hbase(main):018:0> truncate 'student'
    提示：清空表的操作顺序为先disable，然后再truncate。
```  
10）删除表  
```txt
    首先需要先让该表为disable状态：
    hbase(main):019:0> disable 'student'
    然后才能drop这个表：
    hbase(main):020:0> drop 'student'
    提示：如果直接drop表，会报错：ERROR: Table student is enabled. Disable it first.
```  
11）变更表信息  
```txt
    将info列族中的数据存放3个版本：
    hbase(main):022:0> alter 'student',{NAME=>'info',VERSIONS=>3}
    hbase(main):022:0> get 'student','1001',{COLUMN=>'info:name',VERSIONS=>3}
```  












