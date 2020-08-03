### 前言:

- 本文讲述 Electron + Vue + signTool.exe/signCode.exe 在windows下，打包加入签名过程中遇到的各种坑;（win10环境）

### 理由:

- 过程中很多信息东拼西凑，没找到一套完整的教程，故在这里作总结，记录下来，也希望别人脱坑。

- 市面上有很多很多签名生成脚本（商业化产物需要花钱）可用，但毕竟是别人自己的。

- 在此自己尝试，走通整个签名生成流程

### 准备部分:

- 需要知道

  ```shell
  官网有关windows签名：
  [https://www.electronjs.org/docs/tutorial/code-signing](https://www.electronjs.org/docs/tutorial/code-signing) 
  【官网上针对windows的签名几乎没怎么提到，前因后果没讲明白，一头雾水】
  ```

  ```
  electronr进行签名与公证:
  [https://www.cnblogs.com/mmykdbc/p/11468908.html](https://www.cnblogs.com/mmykdbc/p/11468908.html) 
  【该网站有讲解windows下build需要的参数，但使用过程中发现时间戳的网址无法使用】
  ```

  ```
  通过stackoverflow找到了解决时间戳网址无法访问的方法：
  [https://stackoverflow.com/questions/27762109/do-i-need-to-use-my-certificate-authorities-timestamp-server](https://stackoverflow.com/questions/27762109/do-i-need-to-use-my-certificate-authorities-timestamp-server) 

  后来发现网站也不一定可用，最后找到个比较靠谱的答案：
  [https://www.cnblogs.com/mmykdbc/p/12221833.html](https://www.cnblogs.com/mmykdbc/p/12221833.html) 
  ```

  故最终的vue.config.js文件中，win相关的配置如下，
  ```
  "win": {
   "icon": "build/icons/icon.ico",
   "target": [
     {
       "target": "nsis",
       "arch": ["x64", "ia32"]
     }
   ],
   "verifyUpdateCodeSignature": false,  
   "signingHashAlgorithms": [ // 支持的签名
     "sha256",
     "sha1"
   ],
   "signDlls": true,  // 是否对.dll文件进行签名
   "rfc3161TimeStampServer": "http://time.certum.pl/", // 时间戳
   "certificateFile": "xxx.pfx",  // 自己生成的pfx证书（最好是购买的证书）
   "certificatePassword": "xxxxxx"  // 私匙
  },  
  ```


### 自己生成签名:

- sign全家桶，已经放项目中了
  
  cert2spc.exe

  certmgr.exe

  makecat.exe

  makecert.exe  是命令下生成证书

  pvk2pfx.exe 

  signcode.exe  是添加证书

  signtool.exe  是命令行添加证书

  ```
  签名流程，在微软下分成两种实现方式：
  1、signTool.exe：纯命令行方式实现文件签名
  2、signCode.exe：命令行+客户端，两者结合方式实现文件签名
  ```

  ```
  添加数字签名教程：
  [https://jingyan.baidu.com/article/59a015e3a0e807f79488658e.html](https://jingyan.baidu.com/article/59a015e3a0e807f79488658e.html) 
  ```

  ```
  微软官方SignTool.exe（签名工具）讲解，包括对应参数
  [https://docs.microsoft.com/zh-cn/dotnet/framework/tools/signtool-exe](https://docs.microsoft.com/zh-cn/dotnet/framework/tools/signtool-exe) 
  ```

  ```
  SignTool.exe使用过程用到的命令
  [https://www.jianshu.com/p/cc085fbb2a21](https://www.jianshu.com/p/cc085fbb2a21) 
  ```

  ```
  以下方式采用signTool，使用到makecert.exe、cert2spc.exe、pvk2pfx.exe、signtool.exe

  ①makecert.exe -sv mykey.pvk -ss store -n "CN=xxxxxx" -b 01/01/2020 -e 12/31/2100 -$ "individual" -r mycert.cer

  【命令输入后，需要用户自己输入自定义密码】

  【mykey.pvk的mykey ———— 是自定义的，命令输入后会生成该自定义文件】

  【mycert.cer的mycert ———— 是自定义的，命令输入后会生成该自定义文件】

  【-b 01/01/2020 -e 12/31/2100 ———— 指定了开始时间到结束时间】

  【-$ "individual" -r ———— 上网搜索后，发现如果不加会报错】

  【"CN=xxxxxx" ———— 等于号右边写上自己公司名称（或组织名称）】

  ②cert2spc.exe mycert.cer mycert.spc

  【命令输入后，将cert转换成spc】

  【mycert.spc ———— 是自定义的，命令输入后会生成该自定义文件，用于后面签名生成】

  ③pvk2pfx.exe -pi xxxxx -pvk mykey.pvk -spc mycert.spc -pfx mycert.pfx -f

  【命令输入后，将pvk转换成pfx】

  【-pi xxxxx ———— 指定密码（跟一开始设置的密码一样）】

  【-pvk mykey.pvk ———— 生成证书，所必要的pvk文件】

  【-spc mycert.spc ———— 生成证书，所必要的spc文件】

  【-pfx mycert.pfx ———— 是自定义的，命令输入后会生成该自定义文件】

  ④signtool.exe sign /f mycert.pfx /p xxxxx xxxx.dll

  【输入命令后，会提示用户指定文件签名成功】

  【/f mycert.pfx ———— 指定证书（该案例中提到的证书是个人的，不被官方认可）】

  【/p xxxxx ———— 指定密码（跟一开始设置的密码一样）】

  【xxxx.dll ———— 需要添加签名的文件，可以dll、exe】
  ```

  ```
  以下方式采用signCode，使用到makecert.exe、cert2spc.exe、pvk2pfx.exe、signCode.exe
  
  前面几步都是一样，仅最后一步不采用signTool，而是采用signCode

  参考地址：
  [https://jingyan.baidu.com/article/59a015e3a0e807f79488658e.html](https://jingyan.baidu.com/article/59a015e3a0e807f79488658e.html) 
  ```

  ```
  The signer's certificate is not valid for signing。
  最后一步对文件添加签名，遇到这个报错，花了两天解决的。
  解决方法：
  [https://zhidao.baidu.com/question/511378480.html](https://zhidao.baidu.com/question/511378480.html) 
  【在makecert 中加上-$ "individual" -r】
  ```

### electron处理签名

- 需要知道的:

  在自己动手生成签名成功，把该踩的坑踩完后，electron剩下的工作就如鱼得水了
  ```
  需要从上面步骤中获取到xxx.pfx，也就是自己生成的证书放在vue.config.js同目录中

  electron会自动遍历所有需要加证书的文件，循环遍历并添加证书
  ```
![build结果](https://raw.githubusercontent.com/victor-fa/electron-build-sign/master/img/electron-sign.jpg)

### 其他注意

- 需要知道的:
  
  ```
  win + R， 输入certmgr.msc， 可查看本地安装的证书
  ```

  ```
  如何安装windows SDK？
  [https://www.zhihu.com/question/60148198?sort=created](https://www.zhihu.com/question/60148198?sort=created) 

  下载地址？
  [https://developer.microsoft.com/zh-cn/windows/downloads/windows-10-sdk/](https://developer.microsoft.com/zh-cn/windows/downloads/windows-10-sdk/) 
  ```

  ```
  makecert.exe生成一个自签名的证书
  [https://www.cnblogs.com/Herzog3/p/5734033.html](https://www.cnblogs.com/Herzog3/p/5734033.html) 
  ```

  ```
  signcode.exe和signtool.exe之间的主要区别是什么？
  [https://www.orcode.com/question/131639_kf6488.html](https://www.orcode.com/question/131639_kf6488.html) 
  ```
  ```
  electron配置nsis
  [https://www.electron.build/configuration/nsis.html#custom-nsis-script](https://www.electron.build/configuration/nsis.html#custom-nsis-script) 
  [https://www.electron.build/generated/nsisoptions](https://www.electron.build/generated/nsisoptions) 
  ```
  ```
  signCode下载地址？
  [https://www.zhaodll.com/dll/softdown.asp?softid=330201&iz2=f258f798acf62bef9cf2023a1f6c831f](https://www.zhaodll.com/dll/softdown.asp?softid=330201&iz2=f258f798acf62bef9cf2023a1f6c831f) 
  ```
  ```
  cert2spc.exe、certmgr.exe、makecat.exe、makecert.exe、pvk2pfx.exe、signtool.exe工具位置？
  C:\Program Files (x86)\Windows Kits\10\bin\10.0.19041.0\x64
  【网上很多提到的位置都不对，自己找了下在上面提到的位置】
  ```

  ```
  十大免费时间戳服务器地址
  [https://www.cnblogs.com/mmykdbc/p/12221833.html](https://www.cnblogs.com/mmykdbc/p/12221833.html) 
  【发现很多提供参考的教程，在build过程中报错网站无法访问，故自己找到免费时间戳以避免该情况发生】
  ```
  