# gulimall

# 服务器信息


公网ip:

## 环境配置

1. mysq

   ip:
   username=root
   password=Xf123456!

   

   ```
   docker pull mysql
   mkdir -p /mydata/mysql/{conf,data,log}
   ```

```shell
   ##可以先创建my.conf再docker run
   vi /mydata/mysql/conf/my.conf
   
   ##低版本
   docker run -p 3306:3306 --name mysql5.7 -v /mydata/mysql/log:/var/log/mysql -v /mydata/mysql/data:/var/lib/mysql -v /mydata/mysql/conf:/etc/mysql -e MYSQL_ROOT_PASSWORD=Xf123456! -d mysql:last
   ##高版本
   docker run -p 3306:3306 --name mysql \
   -v /mydata/mysql/conf:/etc/mysql/conf.d \
   -v /mydata/mysql/logs:/var/log/mysql \
   -v /mydata/mysql/data:/var/lib/mysql \
   -e MYSQL_ROOT_PASSWORD=root \
   -d mysql:8.0.33
   
   docker restart mysql
```

   my.conf

   >[client]
   > default-character-set=utf8
   > [mysql]
   > default-character-set=utf8
   > [mysqld]
   > init_connect='SET collation_connection = utf8_unicode_ci'
   > init_connect='SET NAMES utf8'
   > collation-server=utf8_unicode_ci
   > character-set-server=utf8
   > skip-character-set-client-handshake
   > skip-name-resolve

   MySQL8.0.33开启远程连接
```shell
##进入容器内部
docker exec -it mysql /bin/bash
##登录
mysql -uroot -p
##选择mysql数据库
use mysql;
##授予权限 %表示host不受限制，如果是本机使用localhost
grant all privileges on *.* to 'root'@'%';
##网上很多是下面这条有可能报错
##GRANT ALL PRIVILEGES ON *.* 'root'@'%' identified by '密码' WITH GRANT OPTION;
##刷新权限
flush privileges;

##如果出现Client does not support authentication protocol requested by server;
##原因是mysql 8以上默认使用的是caching_sha2_password身份验证机制，之前用的是mysql_native_password。
##修改密码的加密方式
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '密码';

update user set host='%' where user='root' ;
```


2. redis

   docker pull redis

   ```shell
   mkdir -p /mydata/redis/conf
   touch /mydata/redis/conf/redis.conf
   
   docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data  -v/mydata/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf
   ```

   redis.conf中加入密码配置

   > requirepass Xf123456!

3. docker

   1.拉取镜像

   ```
   docker pull nacos/nacos-server:2.0.3
   ```

   2.创建数据库nacos_config

   ```mysql
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = config_info   */
   /******************************************/
   CREATE TABLE `config_info` (
     `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
     `data_id` varchar(255) NOT NULL COMMENT 'data_id',
     `group_id` varchar(255) DEFAULT NULL,
     `content` longtext NOT NULL COMMENT 'content',
     `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
     `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
     `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
     `src_user` text COMMENT 'source user',
     `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
     `app_name` varchar(128) DEFAULT NULL,
     `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
     `c_desc` varchar(256) DEFAULT NULL,
     `c_use` varchar(64) DEFAULT NULL,
     `effect` varchar(64) DEFAULT NULL,
     `type` varchar(64) DEFAULT NULL,
     `c_schema` text,
     `encrypted_data_key` text NOT NULL COMMENT '秘钥',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = config_info_aggr   */
   /******************************************/
   CREATE TABLE `config_info_aggr` (
     `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
     `data_id` varchar(255) NOT NULL COMMENT 'data_id',
     `group_id` varchar(255) NOT NULL COMMENT 'group_id',
     `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
     `content` longtext NOT NULL COMMENT '内容',
     `gmt_modified` datetime NOT NULL COMMENT '修改时间',
     `app_name` varchar(128) DEFAULT NULL,
     `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';
   
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = config_info_beta   */
   /******************************************/
   CREATE TABLE `config_info_beta` (
     `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
     `data_id` varchar(255) NOT NULL COMMENT 'data_id',
     `group_id` varchar(128) NOT NULL COMMENT 'group_id',
     `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
     `content` longtext NOT NULL COMMENT 'content',
     `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
     `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
     `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
     `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
     `src_user` text COMMENT 'source user',
     `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
     `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
     `encrypted_data_key` text NOT NULL COMMENT '秘钥',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = config_info_tag   */
   /******************************************/
   CREATE TABLE `config_info_tag` (
     `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
     `data_id` varchar(255) NOT NULL COMMENT 'data_id',
     `group_id` varchar(128) NOT NULL COMMENT 'group_id',
     `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
     `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
     `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
     `content` longtext NOT NULL COMMENT 'content',
     `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
     `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
     `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
     `src_user` text COMMENT 'source user',
     `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = config_tags_relation   */
   /******************************************/
   CREATE TABLE `config_tags_relation` (
     `id` bigint(20) NOT NULL COMMENT 'id',
     `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
     `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
     `data_id` varchar(255) NOT NULL COMMENT 'data_id',
     `group_id` varchar(128) NOT NULL COMMENT 'group_id',
     `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
     `nid` bigint(20) NOT NULL AUTO_INCREMENT,
     PRIMARY KEY (`nid`),
     UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
     KEY `idx_tenant_id` (`tenant_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = group_capacity   */
   /******************************************/
   CREATE TABLE `group_capacity` (
     `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
     `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
     `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
     `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
     `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
     `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
     `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
     `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
     `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
     `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_group_id` (`group_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = his_config_info   */
   /******************************************/
   CREATE TABLE `his_config_info` (
     `id` bigint(64) unsigned NOT NULL,
     `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
     `data_id` varchar(255) NOT NULL,
     `group_id` varchar(128) NOT NULL,
     `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
     `content` longtext NOT NULL,
     `md5` varchar(32) DEFAULT NULL,
     `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',
     `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',
     `src_user` text,
     `src_ip` varchar(20) DEFAULT NULL,
     `op_type` char(10) DEFAULT NULL,
     `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
     `encrypted_data_key` text NOT NULL COMMENT '秘钥',
     PRIMARY KEY (`nid`),
     KEY `idx_gmt_create` (`gmt_create`),
     KEY `idx_gmt_modified` (`gmt_modified`),
     KEY `idx_did` (`data_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';
   
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = tenant_capacity   */
   /******************************************/
   CREATE TABLE `tenant_capacity` (
     `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
     `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
     `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
     `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
     `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
     `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
     `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
     `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
     `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
     `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_tenant_id` (`tenant_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';
   
   
   CREATE TABLE `tenant_info` (
     `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
     `kp` varchar(128) NOT NULL COMMENT 'kp',
     `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
     `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
     `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
     `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
     `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
     `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
     KEY `idx_tenant_id` (`tenant_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';
   
   CREATE TABLE users (
   	username varchar(50) NOT NULL PRIMARY KEY,
   	password varchar(500) NOT NULL,
   	enabled boolean NOT NULL
   );
   
   CREATE TABLE roles (
   	username varchar(50) NOT NULL,
   	role varchar(50) NOT NULL,
   	constraint uk_username_role UNIQUE (username,role)
   );
   
   CREATE TABLE permissions (
       role varchar(50) NOT NULL,
       resource varchar(512) NOT NULL,
       action varchar(8) NOT NULL,
       constraint uk_role_permission UNIQUE (role,resource,action)
   );
   
   INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);
   
   INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
   ```

   3.启动nacos

   ```shell
   #解释版 复制下面的版本注意\后面不能有空格
   docker run -d \
   -e MODE=standalone \ # 使用单机模式
   -e SPRING_DATASOURCE_PLATFORM=mysql \ # 数据库类型
   -e MYSQL_SERVICE_HOST=localhost \ # 数据库地址
   -e MYSQL_SERVICE_USER=root \ # 数据库账号
   -e MYSQL_SERVICE_PASSWORD=pyongsen \ # 数据库密码
   -e MYSQL_SERVICE_DB_NAME=nacos_config \ # 数据库名称
   -e JVM_XMS=256m \
   -e JVM_XMX=256m \
   -e JVM_XMN=256m \
   -p 8848:8848 \ #端口映射
   -v /lbs/nacos/logs:/nacos/logs \ #挂载
   --name nacos \ #容器名 
   --restart=always \ # 自动启动
   nacos/nacos-server:latest #镜像名:版本（是最新的版本直接镜像名即可）
   
   
   docker run -d \
   -e MODE=standalone \
   -e SPRING_DATASOURCE_PLATFORM=mysql \
   -e MYSQL_SERVICE_HOST=106.13.196.135 \
   -e MYSQL_SERVICE_USER=root \
   -e MYSQL_SERVICE_PASSWORD=Xf123456! \
   -e MYSQL_SERVICE_DB_NAME=nacos_config \
   -e JVM_XMS=256m \
   -e JVM_XMX=256m \
   -e JVM_XMN=256m \
   -p 8848:8848 \
    -v /lbs/nacos/logs:/nacos/logs \
   --name nacos \
   --restart=always \
   nacos/nacos-server:2.0.3
   ```

   

# OpenFeign远程调用

​	本项目为springboot、maven项目

1. 引入OpenFeign

   ```xml
   <dependency>  
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. 编写一个接口,加入注解@FeignClient，告诉springCloud这个接口需要调用远程服务，声明接口的每一个方法是调用远程服务的哪个请求

   生产者：

   被消费者调用的接口

   ```java
   import java.util.Map;
   import org.springframework.web.bind.annotation.RestController;
   import com.atguigu.gulimall.coupon.entity.CouponEntity;
   import com.atguigu.common.utils.R;
   /**
    * 优惠券信息
    *
    * @author xiongfeng
    * @email xf88023@163.com
    * @date 2022-08-14 21:39:33
    */
   @RestController
   @RequestMapping("coupon/coupon")
   public class CouponController {
   
       @RequestMapping("/member/list")
       public R membercoupons(){
           CouponEntity couponEntity = new CouponEntity();
           couponEntity.setCouponName("打骨折");
           return R.ok("coupons").put("coupons",couponEntity);
       }
   }
   ```

   消费者：

   Feign包中的接口

   ```java
   @FeignClient("gulimall-coupon")
   public interface CouponFeignService {
   
       @RequestMapping("/coupon/coupon/member/list")
       public R membercoupons();
   }
   ```

   测试接口:/member/member/coupons

   ```java
   package com.atguigu.gulimall.member.controller;
   
   import com.atguigu.gulimall.member.feign.CouponFeignService;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import com.atguigu.gulimall.member.entity.MemberEntity;
   import com.atguigu.common.utils.R;
   
   /**
    * 会员
    *
    * @author xiongfeng
    * @email xf88023@163.com
    * @date 2022-08-14 21:48:20
    */
   @RestController
   @RequestMapping("member/member")
   public class MemberController {
   
       @Autowired
       private CouponFeignService couponFeignService;
       @RequestMapping("/coupons")
       public R test(){
           MemberEntity memberEntity = new MemberEntity();
           memberEntity.setNickname("zhangsan");
           R membercoupons = couponFeignService.membercoupons();
           Object coupons = membercoupons.get("coupons");
           return R.ok().put("member",memberEntity).put("coupons",coupons);
       }
   }
   ```



1. 开启远程调用

   ```java
   @EnableFeignClients(basePackages = "com.atguigu.gulimall.member.feign")
   @SpringBootApplication
   public class GulimallMemberApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(GulimallMemberApplication.class, args);
       }
   
   }
   ```
