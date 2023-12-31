# typora + PicGo + lankong 图床自动图片上传

## PicGo 下载

首先在文件\偏好设置\图像里设置上传服务，这里自带了 PicGo 选项

点击下载按钮，跳转到下载页面，或者直接访问地址 [https://molunerfinn.com/PicGo](https://molunerfinn.com/PicGo)

也可以去 github 下载最新的beat版本 [https://github.com/Molunerfinn/PicGo/releases](https://github.com/Molunerfinn/PicGo/releases)

![image-20231227164359449](https://photo.daydaysup.com/st/2023/12/28/658cef28488d2.png)

## PicGo 配置

### 安装 lankong 插件

#### 1、在线安装，

在插件设置搜索 lankong

![image-20231227171825812](https://photo.daydaysup.com/st/2023/12/28/658cef283f751.png)

如果安装不上可以到用户目录看下日志，或者在picgo 设置里就可以直接打开日志文件，有可能是 npm 网络问题，无法直接下载成功。这种情况可以手动安装

#### 2、手动安装

到官网下载插件手动安装 [https://github.com/hellodk34/picgo-plugin-lankong](https://github.com/hellodk34/picgo-plugin-lankong)

进入到自己的 picgo 用户目录，路径为 C:\Users\Administrator\AppData\Roaming\picgo

在该目录下直接 git clone

 git clone https://github.com/hellodk34/picgo-plugin-lankong.git

或者下载 zip 包解压，插件的名字叫 picgo-plugin-lankong

然后在当前目录执行 npm 命令安装

 npm install picgo-plugin-lankong

如果报错 `Cannot read properties of null (reading 'isDescendantOf')`，一般是网络问题，改用 cnpm 安装就行了

![image-20231227180654538](https://photo.daydaysup.com/st/2023/12/28/658cef2beae2b.png)

 cnpm install picgo-plugin-lankong

## 3、图床设置

我这里介绍的是 lankong 图床，其他图床设置参考官方手册

### 1、获取 token

#### 使用 curl(**推荐**)

```
 curl --location --request POST 'https://photo.daydaysup.com/api/v1/tokens' \  
 --form 'email=305578965@qq.com' \  
 --form 'password=123456'
```

返回信息，获取到 token 值，记录下来

 ```
 {  
     "status": true,  
     "message": "success",  
     "data": {  
         "token": "1|rDWVoGJdy2asWmnK2342FV0nFgklvG79WaHbMNv"  
     }  
 }

```
#### 使用 postman 工具

参数格式就参考 curl 的内容

### 2、图床设置

![image-20231227183155972](https://photo.daydaysup.com/st/2023/12/28/658cef2fdf9ec.png)

- Server 地址
    
    - `https://image.example.com` ✅️
        
    - `https://image.example.com/` ❌️
        
- 填写 `Auth Token` 使用 `Bearer` 拼接，`Bearer` + 空格 + 步骤1 中获取的 token 值
    
- `Strategy ID`，存储策略 ID，默认存储策略的用户，请留空；除非你知道具体 ID，否则请留空
    
- `Album ID`，相册 ID
    
- `Permission`，图片权限，公开还是私有，默认是私有
    
- `Ignore certificate error` 开关，默认关闭，请保持关闭，除非你遇到 `certificate has expired` 等证书报错才需要考虑将其开启。由于有些站点使用 Let's Encrypt 颁发的免费证书，有效期只有 90 天，在测试上传中遇到了 `certificate has expired` 错误，打开开关 `Ignore certificate error` 即可成功上传
    
- `Sync Delete` 同步删除选项，只支持 `V2`，开启后在 PicGo 相册中删除图片可同步删除图床上的文件，默认关闭
    

### 3、相册ID无效的问题

关于设置 Album ID 不生效的问题，开源版本的代码相册ID功能没啥用，解决方法可以参考 issues #495

 ```
 // 参考 issues 495  
 // if ($albumId = $user->configs->get(UserConfigKey::DefaultAlbum)) {  
 //     if ($user->albums()->where('id', $albumId)->exists()) {  
 //         $image->album_id = $albumId;  
 //     }  
 // }  
 ​  
 if ($request->has('album_id')) {  
     $image->album_id = $request->input('album_id');  
 } else {  
     // 图片保存至默认相册(若有)  
     if ($albumId = $user->configs->get(UserConfigKey::DefaultAlbum)) {  
         if ($user->albums()->where('id', $albumId)->exists()) {  
             $image->album_id = $albumId;  
         }  
     }  
 }
```