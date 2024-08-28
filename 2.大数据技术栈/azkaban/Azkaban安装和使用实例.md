# Azkaban安装和使用实例
https://blog.csdn.net/weixin_35852328/article/details/79327996

```
创建SSL配置 
命令: keytool -keystore keystore -alias jetty -genkey -keyalg RSA 

-keystore 指定秘钥库位置
-alias 指定秘钥别名
-genkey 产生key
-keyalg 算法

输入keystore密码： //秘钥库口令
再次输入新密码: 
您的名字与姓氏是什么？ 
[Unknown]： 
您的组织单位名称是什么？ 
[Unknown]： 
您的组织名称是什么？ 
[Unknown]： 
您所在的城市或区域名称是什么？ 
[Unknown]： 
您所在的州或省份名称是什么？ 
[Unknown]： 
该单位的两字母国家代码是什么 
[Unknown]： CN 
CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=CN 正确吗？ 
[否]： y

输入的主密码 
（如果和 keystore 密码相同，按回车）：  //秘钥口令
再次输入新密码: 

完成上述工作后,将在当前目录生成 keystore 证书文件,将keystore 考贝到 azkaban web服务器根目录中的bin目录下.如:cp keystore azkaban/webserver/bin

```