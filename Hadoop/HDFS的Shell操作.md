## HDFS的Shell操作
### 1、基本语法  
    bin/hadoop fs 具体命令  

### 2、常用命令  
#### 1、–ls：查看指定目录下内容
    hadoop fs –ls [文件目录]  
       eg：hadoop fs –ls /user/wangkai.pt  
#### 2、–cat：显示文件内容
    hadoop dfs –cat [file_path]
       eg:hadoop fs -cat /user/wangkai.pt/data.txt
#### 3、–put：将本地文件存储至hadoop
    hadoop fs –put [本地地址] [hadoop目录]
       eg：hadoop fs –put /home/t/file.txt  /user/t   
       (file.txt是文件名)
#### 4、–put：将本地文件夹存储至hadoop
    hadoop fs –put [本地目录] [hadoop目录]
       eg：hadoop fs –put /home/t/dir_name /user/t
       (dir_name是文件夹名)
#### 5、-get：将hadoop上某个文件down至本地已有目录下
    hadoop fs -get [文件目录] [本地目录]
       eg：hadoop fs –get /user/t/ok.txt /home/t
#### 6、–rm：删除hadoop上指定文件或文件夹
    hadoop fs –rm [文件地址]
       eg：hadoop fs –rm /user/t/ok.txt
#### 7、删除hadoop上指定文件夹（包含子目录等）
    hadoop fs –rm [目录地址]
       eg：hadoop fs –rm /user/t
#### 8、–mkdir：在hadoop指定目录内创建新目录
    eg：hadoop fs –mkdir /user/t
#### 9、-touchz：在hadoop指定目录下新建一个空文件
    使用touchz命令：
    eg：hadoop  fs  -touchz  /user/new.txt
#### 10、–mv：将hadoop上某个文件重命名
    使用mv命令：
    eg：hadoop  fs  –mv  /user/test.txt  /user/ok.txt   （将test.txt重命名为ok.txt）
#### 11、–getmerge：将hadoop指定目录下所有内容保存为一个文件，同时down至本地
    eg：hadoop fs –getmerge /user /home/t
#### 12、将正在运行的hadoop作业kill掉
    eg：hadoop job –kill  [job-id]
#### 13、-help：输出这个命令参数
    eg：hadoop fs -help rm
#### 14、-moveFromLocal：从本地剪切粘贴到HDFS
    eg：hadoop fs  -moveFromLocal  ./kongming.txt  /sanguo/shuguo
#### 15、-appendToFile：追加一个文件到已经存在的文件末尾
    eg：hadoop fs -appendToFile liubei.txt /sanguo/shuguo/kongming.txt
#### 16、-chgrp 、-chmod、-chown：Linux文件系统中的用法一样，修改文件所属权限
    eg：hadoop fs  -chmod  666  /sanguo/shuguo/kongming.txt
    eg：hadoop fs  -chown  atguigu:atguigu   /sanguo/shuguo/kongming.txt
#### 17、-copyFromLocal：从本地文件系统中拷贝文件到HDFS路径去
    eg：hadoop fs -copyFromLocal README.txt /
#### 18、-copyToLocal：从HDFS拷贝到本地
    eg：hadoop fs -copyToLocal /sanguo/shuguo/kongming.txt ./
#### 19、-cp ：从HDFS的一个路径拷贝到HDFS的另一个路径
    eg：hadoop fs -cp /sanguo/shuguo/kongming.txt /zhuge.txt
#### 20、-tail：显示一个文件的末尾
    eg：hadoop fs -tail /sanguo/shuguo/kongming.txt
#### 21、-rmdir：删除空目录
    eg：hadoop fs -mkdir /test
    eg：hadoop fs -rmdir /test
#### 22、-du：统计文件夹的大小信息
    eg：hadoop fs -du -s -h /user/atguigu/test
        2.7 K  /user/atguigu/test
    eg：hadoop fs -du  -h /user/atguigu/test
        1.3 K  /user/atguigu/test/README.txt
        15     /user/atguigu/test/jinlian.txt
        1.4 K  /user/atguigu/test/zaiyiqi.txt
#### 23、-setrep：设置HDFS中文件的副本数量
    eg：hadoop fs -setrep 10 /sanguo/shuguo/kongming.txt



