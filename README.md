# springboot-demo-https

### Reference Documentation
1. 生成私鑰(假設密碼為123123)
```
 openssl genrsa -des3 -out server.key 2048
```
2.生成csr
```
openssl req -new -key server.key -out server.csr
```
3.生成簽證用的CA
```
openssl req -new -x509 -key server.key -out server.crt -days 3650
```
4.拿CA對csr做簽證
```
openssl x509 -req -days 3650 -in server.csr \
      -CA server.crt -CAkey server.key \
      -CAcreateserial -out server-selfsign.crt
```

6.將公私鑰合併成PKCS12格式( import.p12 ) 
```
openssl pkcs12 -export -in server-selfsign.crt -inkey server.key \
               -out import.p12 -name tomcat 
```
7.再將import.p12匯入keystore(假設金鑰庫密碼為123456)
```
keytool -importkeystore -destkeystore keystore.jks -srckeystore import.p12 -srcstoretype PKCS12
```
8.設定application.properties
```
server.port=443

server.ssl.key-store-type=PKCS12
server.ssl.key-store=classpath:keystore.jks
server.ssl.key-store-password=123456
server.ssl.key=password=123123
server.ssl.key-alias=tomcat
```

Client端產生truststore
```
keytool -importcert -alias cakey -file server-selfsign.crt -keystore truststore.jks
```