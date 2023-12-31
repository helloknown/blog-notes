# 如何给网站添加免费的 https 证书

本文主要介绍如何通过`ohttps`获取一个免费的，可以无限续期的 https 证书

[OHTTPS](https://ohttps.com/)致力于为用户提供免费HTTPS证书申请、自动更新、自动部署等服务，**所有HTTPS证书均由[Let's Encrypt](https://letsencrypt.org/)颁发**，受所有主流浏览器信任。

通过如下的操作步骤，你即可生成一个证书。

1. 注册 ohttps

2. 创建证书

   2.1 免授权方式

   2.2 DNS 授权方式

3. 证书管理

## 注册ohttps

打开官网 https://ohttps.com ，点击右上角的注册即可

![image-20231225214236345](https://photo.daydaysup.com/st/2023/12/27/658c15e4d754f.png)



注册完成后，会直接登录成功，并跳转到创建证书的界面，先不急着操作，按照下面的步骤再开始做

## 创建证书

### 添加域名

添加一个域名，在泛域名中，用通配符 `*` 来表示

![image-20231225212353973](https://photo.daydaysup.com/st/2023/12/27/658c16013aabe.png)

### 验证域名

这里提供了两种方式

#### 免 DNS 授权

点击下一步，这里如果直接点击验证解析记录的按钮，则提示**您的DNS域名解析记录中不存在免授权申请证书所需的CNAME类型解析记录**

![image-20231225212316786](https://photo.daydaysup.com/st/2023/12/27/658c160857c0b.png)

3、这里按照提示信息，我们需要将下面的内容拷贝出来，然后在 cloudflare 中创建一条 DNS 解析记录

![image-20231225215631622](https://photo.daydaysup.com/st/2023/12/27/658c160b21b4d.png)

![image-20231225215552823](https://photo.daydaysup.com/st/2023/12/27/658c2d6f28416.png)

最后点击验证解析记录。通过之后就可以点击创建证书了



#### DNS 授权

##### 添加授权

使用 DNS 授权的方式，需要先在管理台中的 DNS 授权菜单中添加授权

我这里使用的是 cloudflare，其他的云服务都是类似的操作，每一个官方都有详细的参考文档 https://ohttps.com/docs/dns

![image-20231225213839911](https://photo.daydaysup.com/st/2023/12/27/658c160e2f25e.png)



##### 创建证书

接着和2.1的步骤一样，添加域名，然后点击 DNS 授权模式，选中 cloudflare ，下拉框选择刚刚添加的授权，然后点击授权验证，验证通过后就可以点击创建证书按钮了

![image-20231225220229239](https://photo.daydaysup.com/st/2023/12/27/658c160e582e6.png)



## 证书管理

上述步骤完成之后，就可以进入到证书管理看到刚刚创建的证书记录

点击配置，可以设置到期自动更新时间，这里保持默认就好

![image-20231225223244542](https://photo.daydaysup.com/st/2023/12/27/658c161256fac.png)



进行详情界面可以查看具体的证书文件信息，下载需要的证书信息

![image-20231225223403398](https://photo.daydaysup.com/st/2023/12/27/658c1615766da.png)

![image-20231225223427265](https://photo.daydaysup.com/st/2023/12/27/658c161a6b9b8.png)
