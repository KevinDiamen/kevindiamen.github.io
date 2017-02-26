# Nginx 配置 HTTPS

- [介绍](#introduction)
- [可信任的 SSL 证书](#installation)
- [自签名 SSL 证书](#person-ssl)

<a name="introduction"></a>
## 介绍
SSL证书通过在客户端浏览器和Web服务器之间建立一条SSL安全通道（Secure socket layer(SSL)安全协议是由Netscape Communication公司设计开发。该安全协议主要用来提供对用户和服务器的认证；对传送的数据进行加密和隐藏；确保数据在传送中不被改变，即数据的完整性，现已成为该领域中全球化的标准。

## 可信任的 SSL 证书
配置 HTTPS 要用到私钥 example.key 文件和 example.crt 证书文件，申请证书文件的时候要用到 example.csr 文件，OpenSSL 命令可以生成 example.key 文件和 example.csr 证书文件。

+ CSR：Cerificate Signing Request，证书签署请求文件，里面包含申请者的 DN（Distinguished Name，标识名）和公钥信息，在第三方证书颁发机构签署证书的时候需要提供。证书颁发机构拿到 CSR 后使用其根证书私钥对证书进行加密并生成 CRT 证书文件，里面包含证书加密信息以及申请者的 DN 及公钥信息
+ Key：证书申请者私钥文件，和证书里面的公钥配对使用，在 HTTPS 『握手』通讯过程需要使用私钥去解密客戶端发來的经过证书公钥加密的随机数信息，是 HTTPS 加密通讯过程非常重要的文件，在配置 HTTPS 的時候要用到

使用 OpenSSl命令可以在系统当前目录生成 example.key 和 example.csr 文件：

	openssl req -new -newkey rsa:2048 -sha256 -nodes -out example_com.csr -keyout example_com.key -subj "/C=CN/ST=ShenZhen/L=ShenZhen/O=Example Inc./OU=Web Security/CN=example.com"
	
下面是上述命令相关字段含义：

+ C：Country ，单位所在国家，为两位数的国家缩写，如： CN 就是中国
+ ST 字段： State/Province ，单位所在州或省
+ L 字段： Locality ，单位所在城市 / 或县区
+ O 字段： Organization ，此网站的单位名称;
+ OU 字段： Organization Unit，下属部门名称;也常常用于显示其他证书相关信息，如证书类型，证书产品名称或身份验证类型或验证内容等;
+ CN 字段： Common Name ，网站的域名;

生成 csr 文件后，提供给 CA 机构，签署成功后，就会得到一個 example.crt 证书文件，SSL 证书文件获得后，就可以在 Nginx 配置文件里配置 HTTPS 了。

#### 配置 Nginx
```
server {
    #ssl参数
    listen              443 ssl;
    server_name         example.com;
    #证书文件
    ssl_certificate     example.com.crt;
    #私钥文件
    ssl_certificate_key example.com.key;
    #...
}
```

## 自签名 SSL 证书

```
openssl genrsa -des3 -out example.key 1024
```
```
openssl req -new -subj /C=US/ST=Mars/L=iTranswarp/O=iTranswarp/OU=iTranswarp/CN=example.com -key example.key -out example.csr
```
```
mv example.key example.origin.key
openssl rsa -in example.origin.key -out example.key
```
```
openssl x509 -req -days 3650 -in example.csr -signkey example.key -out $DOMAIN.crt
```
