# 实验二：用户及权限管理
## 王涛 18软工三班 201810414319 
## 实验目的
掌握用户管理、角色管理、权根维护与分配的能力，掌握用户之间共享对象的操作技能。
## 实验内容：
Oracle有一个开发者角色resource，可以创建表、过程、触发器等对象，但是不能创建视图。本训练要求：

- 在pdborcl插接式数据中创建一个新的本地角色con_res_view，该角色包含connect和resource角色，同时也包含CREATE VIEW权限，这样任何拥有con_res_view的用户就同时拥有这三种权限。

- 创建角色之后，再创建用户new_user，给用户分配表空间，设置限额为50M，授予con_res_view角色。

- 最后测试：用新用户new_user连接数据库、创建表，插入数据，创建视图，查询表和视图的数据。

## 实验步骤：
### 1.以system登录到pdborcl，创建角色con_res_wt2356和用户wt2356，并授权和分配空间：
***
CREATE ROLE con_res_wt2356;
GRANT connect,resource,CREATE VIEW TO con_res_wt2356;

CREATE USER wt2356 IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;

ALTER USER wt2356 QUOTA 50M ON users;
GRANT con_res_wt2356 TO wt2356;
### 运行结果截图
![运行结果一](p1.png)

### 2.新用户wt2356连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。
show user;
CREATE TABLE mytable (id number,name varchar(50));
INSERT INTO mytable(id,name)VALUES(1,'zhang');
INSERT INTO mytable(id,name)VALUES (2,'wang');
CREATE VIEW myview AS SELECT name FROM mytable;
SELECT * FROM myview;
GRANT SELECT ON myview TO hr;

### 运行结果截图：
![运行结果二](p2.png)
![运行结果三](p3.png)

### 3.用户hr连接到pdborcl，查询wt2356授予它的视图myview
select * from wt2356.myview;

### 运行结果截图
![运行结果四](p4.png)


## 查看数据库的使用情况：
查看表空间的数据库文件，以及每个文件的磁盘占用情况。

SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';

### 运行结果：
![运行结果](p5.png)

SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
 free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
 where  a.tablespace_name = b.tablespace_name;

 ### 运行结果：
 ![运行结果](p6.png)