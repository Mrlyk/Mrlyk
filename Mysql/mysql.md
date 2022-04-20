# Mysql

> 关系型数据库

## 本地启动服务

本地使用的 brew 安装的 mysql

启动：`brew services start mysql`
重启：`brew services restart mysql`
停止：`brew services stop mysql`
配置文件路径：`/usr/local/etc`

## 一、mysql 相关命令

### 1.1 用户相关

**显示密码策略**
` show global variables like 'validate%';`

**修改密码策略**
-- 密码验证策略低要求(0或LOW代表低级) 
`set global validate_password.policy=0; `

-- 密码至少要包含的小写字母个数和大写字母个数 
`set global validate_password.mixed_case_count=0; `

-- 密码至少要包含的数字个数
`set global validate_password.number_count=0; `

 -- 密码至少要包含的特殊字符数 
`set global validate_password.special_char_count=0; `

 -- 密码长度 
`set global validate_password.length=6; `   

-- **更改密码**
` ALTER user 'root'@'localhost' IDENTIFIED BY '123456';`
*ps:次安装以root身份执行`mysql_secure_installation`即可更改密码*

**新增用户**

CREATE USER [user_name]@'host' IDENTIFIED BY 'password';

举个例子：`create user liaoyk@'%' identified by '960926';`

% - 表示任意 ip 都可以访问

**赋予权限**

GRANT privileges ON [databasename].[tablename] TO 'username'@'host';

- privileges：用户的操作权限，如`SELECT`，`INSERT`，`UPDATE`等，如果要授予所的权限则使用`ALL`

举个例子：`grant all on test.* to liaoyk@'%'`;

### 1.2 mysql 服务相关

**查看启动端口** 修改端口需要修改配置文件（指定port=xxxx），重启服务
` show global variables like 'port';`

## 二、数据库操作常用命令

*ps: "[ ]"中的参数代表可选或自定义的*

**新建数据库**
`create database [IF NOT EXISTS] [database_name]`

**连接数据库**
`mysql -u root -p`

**创建表**

```mysql
CREATE TABLE `app`
(
	id int NOT NULL AUTO_INCREMENT,
	app_name char(50) NOT NULL,
	app_version char(50) NOT NULL,
	app_publish_data timestamp NOT NULL,
	PRIMARY KEY (id)
)ENGINE=INNODB;
```



### 2.1 CRUD （增删改查）

