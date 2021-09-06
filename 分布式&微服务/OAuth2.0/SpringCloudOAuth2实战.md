



OAuth2建表

```mysql
CREATE TABLE `oauth_client_details`  (
  `client_id` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `resource_ids` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `client_secret` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `scope` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `authorized_grant_types` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `web_server_redirect_uri` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `authorities` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `access_token_validity` int(11) NULL DEFAULT NULL,
  `refresh_token_validity` int(11) NULL DEFAULT NULL,
  `additional_information` varchar(4096) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `autoapprove` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`client_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

插入一条数据

```mysql
INSERT INTO `oauth_client_details` VALUES ('client', NULL, '123456', 'all', 'password,refresh_token', '', NULL, NULL, NULL, NULL, NULL);

```



## 参看

- [深入理解Spring Cloud Security OAuth2及JWT](https://www.jianshu.com/p/cb886f995e86) 
- [Spring Security 与 OAuth2（介绍）](https://www.jianshu.com/p/68f22f9a00ee)
- [【我在拉勾训练营学技术】OAuth2+JWT 实现权限验证](https://cloud.tencent.com/developer/article/1774442?from=article.detail.1497573) （基于内存，不存储到mysql的方式实现**认证服务器**；代码注释比较详细）

