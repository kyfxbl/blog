title: 配置https双向认证，以及用soapui调试
date: 2013-09-24 11:04
categories: web 
---
本文介绍HTTPS双向认证的配置和调测
<!--more-->

# 环境信息 

服务端启动jboss-4.2.3，IP是10.78.125.111。启动以后在本地调用，本地的IP是10.78.125.222 

# 单向认证 

单向认证比较简单，用以下命令： 

```
keytool -genkey -alias 111 -keyalg RSA -keystore 111.keystore -validity 3650
```

名字和姓氏要填域名或者IP名：10.78.125.111。其他可以随便填 

这步操作以后，得到111.keystore 将111.keystore放到jboss以下目录：

```
%JBOSS_HOME%\server\default\conf\
```

然后配置server.xml文件

```
<Connector port="443" protocol="HTTP/1.1" SSLEnabled="true"
               maxThreads="150" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" keystoreFile="conf/111.keystore" 
               keystorePass="changeit" keystoreType="jks" />
```

这样单向认证就配置好了，它不要求验证客户端信息，也就是所有客户端都可以访问到 如果是用浏览器访问的话，会提示“此网站的安全证书有问题，是否继续访问”。因为这个证书是用最简单的办法自己做的，没有经过权威CA的签署，所以一般浏览器是不承认其安全性的 

如果是用代码访问的话，则还需要额外的步骤 首先用以下命令： 

```
keytool -export -alias 111 -file 111.cer -keystore 111.keystore
```

以上得到一个111.cer，然后把111.cer给到客户端，客户端用以下命令： 

```
keytool -import -alias 111 -file 111.cer -keystore server.keystore
```

得到server.keystore，把这个文件作为客户端代码的truststore，才能正常访问到。可以理解为因为是用代码来访问服务端，没有用户手工确认的过程，所以需要把证书加进来进行确认 

# 双向认证 

双向认证就比较复杂，因为需要客户端的校验，也就是说不是随便什么客户端都能访问到的 

那本地想要调用到服务端，就也需要做证书，同样先用这个命令： 

```
keytool -genkey -alias 222 -keyalg RSA -keystore 222.keystore -validity 3650 得到222.keystore
```

然后

```
keytool -export -alias 222 -file 222.cer -keystore 222.keystore
```

得到222.cer，把222.cer发给服务端，服务端用以下命令： 

```
keytool -import -alias 222 -file 222.cer -keystore client.keystore 
```

得到了client.keystore，这里面就包含了10.78.125.222的证书信息，可以用以下命令查看：

```
keytool -list -v -keystore client.keystore
```
 
然后将client.keystore放到

```
%JBOSS_HOME%\server\default\conf\
```

再配置server.xml文件

```
<Connector port="443" protocol="HTTP/1.1" SSLEnabled="true"
               maxThreads="150" scheme="https" secure="true"
               clientAuth="true" sslProtocol="TLS" keystoreFile="conf/111.keystore" 
               keystorePass="changeit" keystoreType="jks" truststoreFile="conf/client.keystore" truststorePass="changeit" truststoreType="jks" />
```

这样就配置好双向认证的服务端了 在客户端用代码调用服务端的话，就要把server.keystore作为truststore，把222.keystore作为keystore，如果是用cxf的话，配置文件大概是这样：

```
<http:conduit name="https://10.78.125.111:443/.*">

		<http:tlsClientParameters disableCNCheck="true"
			secureSocketProtocol="SSL">

			<!-- 对方的证书 -->
			<sec:trustManagers>
				<sec:keyStore type="JKS" password="changeit"
					file="/opt/certificates/server.keystore" />
			</sec:trustManagers>

			<!-- 己方的证书 -->
			<sec:keyManagers keyPassword="changeit">
				<sec:keyStore type="JKS" password="changeit"
					file="/opt/certificates/222.keystore" />
			</sec:keyManagers>

			<sec:cipherSuitesFilter>
				<sec:include>.*_EXPORT_.*</sec:include>
				<sec:include>.*_EXPORT1024_.*</sec:include>
				<sec:include>.*_WITH_DES_.*</sec:include>
				<sec:include>.*_WITH_NULL_.*</sec:include>
				<sec:exclude>.*_DH_anon_.*</sec:exclude>
			</sec:cipherSuitesFilter>

		</http:tlsClientParameters>

	</http:conduit>
```

如果是用soupui或者浏览器来访问，在下面说 

# 用SoapUI来调用 

用SoapUI来调用的话，如果是单向认证，则不需要额外的操作，可以直接调用。如果是双向认证，需要导入本地证书，也就是222.keystore 

选择SSL Settings，导入222.keystore，输入密码即可 

# 用浏览器来访问 

如果是单向认证，不需要导入证书也可以访问，只是会提示“此网站的安全证书有问题，是否继续访问”。 

如果是双向认证，需要导入证书才能访问，否则会提示

```
SSL节点无法核实您的证书，错误码： ssl_error_bad_cert_alert
```