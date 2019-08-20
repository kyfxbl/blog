title: HTTPS双向认证及cxf调用
date: 2013-09-24 11:32
categories: java 
---
我方系统与另一个系统对接，需要走https协议。研究了几天，还是一知半解的，不过最终还是满足需求，在这里记录一下
<!--more-->

背景： 我方系统开放了web service供对方调用，对方系统也提供了web service给我方调用。对方要求通过https双向认证，我方只要求单向认证即可 

因为我们这边拿不到客户系统，所以我搭建了一个mock，用来模拟接口交互和https互通 

# 生成我方系统的keystore，并配置我方系统的https 

模拟桩搭建在某台机器上，比如ip地址是1.2.3.4，用以下命令： 

```
keytool -genkey -alias remedy -keyalg RSA -keystore remedy.keystore -validity 3650
```

这样会生成一个remedy.keystore文件 

同理，我方系统部署在另一台机器上，比如ip地址是1.2.3.5，用以下命令： 

```
keytool -genkey -alias wfm -keyalg RSA -keystore wfm.keystore -validity 3650
```

这样会生成一个wfm.keystore文件 因为我方系统只需要单向认证，所以有这个wfm.keystore就可以了。修改server.xml文件

```
<Connector port="443" protocol="HTTP/1.1" SSLEnabled="true" maxThreads="150" scheme="https" secure="true" clientAuth="false" sslProtocol="TLS" keystoreFile="conf/wfm.keystore" keystorePass="changeit" keystoreType="jks" />
```

完成了这步之后，我方系统的https就配置好了，因为我方系统只要求单向认证，比较简单 

# 从keystore导出双方的证书，并交换，配置模拟桩的https 

然后需要把刚才生成的2个keystore导出为证书，进行交换，用以下命令： 

```
keytool -export -alias wfm -file wfm.cer -keystore wfm.keystore 将wfm.keystore导出为wfm.cer
```
 
同理，remedy.keystore也需要导出 

```
keytool -export -alias remedy -file remedy.cer -keystore remedy_dummy.keystore
```
 
得到remedy.cer。由于模拟桩是需要双向认证的，所以要将wfm.cer提供给模拟桩，然后用以下命令导入为truststore 

```
keytool -import -alias wfm -file wfm.cer -keystore client.keystore
```
 
这样会得到一个client.keystore，其中包含了wfm的证书信息，可以用以下命令查看： 

```
keytool -list -v -keystore client.keystore
```
 
这时候，模拟桩已经有了一个自己的keystore，即remedy.keystore，以及一个客户端的truststore，即client.keystore，接下来需要在模拟桩的jboss里配置https 

修改server.xml文件

```
<Connector port="443" protocol="HTTP/1.1" SSLEnabled="true"
               maxThreads="150" scheme="https" secure="true"
               clientAuth="true" sslProtocol="TLS" keystoreFile="conf/remedy.keystore" 
               keystorePass="changeit" keystoreType="jks" truststoreFile="conf/client.keystore" truststorePass="changeit" truststoreType="jks" />
```

完成这步之后，模拟桩的https就也配置好了 

经过上述2步，我方系统的https已经配置完毕，得到wfm.cer的客户端，只要将wfm.cer导入到truststore，就可以调用我方系统了.模拟桩的https也已经配置完毕了，但是现在我方系统还调用不了模拟桩，因为模拟桩是要求双向认证的 

# 将模拟桩的证书导入到我方系统的truststore 

还是用以下命令：

```
keytool -import -alias remedy -file remedy.cer -keystore remedy.keystore
```
 
得到了remedy.keystore，可以将它作为truststore 

完成这步之后，双方的https都配置好了，也已经互换了证书，接下来可以进行客户端编程 

首先把已经准备好的wfm.keystore，以及remedy.keystore（刚刚生成的，作为truststore），放到某个目录中，然后在cxf配置文件中配置

```
<http:conduit name="*.http-conduit">

		<http:tlsClientParameters disableCNCheck="true"
			secureSocketProtocol="SSL">

			<!-- 对方的证书 -->
			<sec:trustManagers>
				<sec:keyStore type="JKS" password="changeit"
					file="/opt/inoc/wfm/certificates/remedy.keystore" />
			</sec:trustManagers>

			<!-- 己方的证书 -->
			<sec:keyManagers keyPassword="changeit">
				<sec:keyStore type="JKS" password="changeit"
					file="/opt/inoc/wfm/certificates/wfm.keystore" />
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
测试调用一下，就可以了 

# 总结 

## 服务端的配置 

如果是配置单向认证的https，只需要在jboss中放置自己的keystore，然后配置文件中的clientAuth设置为false 

如果是配置双向认证的https，则除了自己的keystore之外，还需要放置truststore，truststore是根据对方提供的证书导入的。然后配置文件中的clientAuth需要设置为true 

## keytool命令 

keytool -genkey 是生成自己的keystore 
keytool -export 是从keystore导出证书 
keytool -import 是将证书导入到truststore 

## 关于keystore和truststore 

无论是在jboss中的配置，还是cxf中的配置，只需要记住： keystore是配置自己的.keystore truststore是配置对方的.keystore