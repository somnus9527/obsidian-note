---
tags:
  - 知识笔记/电脑相关
---
#### 前置条件安装scoop

>[!Tip] 安装Scoop命令
>`Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')`

>[!Warning] 异常处理
>- 如果有权限问题,可以尝试 `Set-ExecutionPolicy RemoteSigned -scope CurrentUser`
>- 在公司网络/电脑环境下,有时会有禁止使用admin安装的限制, 测试使用`iex (new-object net.webclient).downloadstring('https://raw.githubusercontent.com/lukesampson/scoop/master/bin/install.ps1')`该命令可以安装
>- scoop安装成功之后需要`scoop bucket add extras`增加额外的包管理地址,scoop自带的地址比较少

1. `scoop install ios-webkit-debug-proxy`
2. `npm install remotedebug-ios-webkit-adapter -g`
3. 打开safari的网页检查器
4. 安装itunes,并打开
5. usb链接iphone,手机和pc上都会出现弹窗,确认就行
6. 启动: `remotedebug_ios_webkit_adapter --port=9000`
7. chrome中打开chrome://inspect/,在configure中配置localhost:9000
8. 手机浏览器中打开需要调试的页面即可

#### 注意点,IOS版本大于11,需要额外操作:
1. ios-webkit-debug-proxy的github上下载最新zip包,地址: [ios-webkit-debug-proxy](https://github.com/google/ios-webkit-debug-proxy/releases)
2. 在`%AppData%\npm\node_modules\remotedebug-ios-webkit-adapter\node_modules\`下创建文件夹`vs-libimobile`,%AppData%一般在`C:\Users\USER_NAME\AppData\Roaming\`
3. 解压之前下载的zip包,并将整个文件夹拷入`%AppData%\npm\node_modules\remotedebug-ios-webkit-adapter\node_modules\vs-libimobile\ios-webkit-debug-proxy-1.8-win64-bin`,注意,这里的`ios-webkit-debug-proxy-1.8-win64-bin`应该改成自己下载的zip包解压之后的文件夹名字
4. 找到`%AppData%\npm\node_modules\remotedebug-ios-webkit-adapter\out\adapters\iosAdapter.js`文件,打开
5. 找到代码
>[!Note] 
>```javascript
>const proxy = process.env.SCOOP ?
path.resolve(__dirname, process.env.SCOOP + '/apps/ios-webkit-debug-proxy/current/ios_webkit_debug_proxy.exe') :
path.resolve(__dirname, process.env.USERPROFILE + '/scoop/apps/ios-webkit-debug-proxy/current/ios_webkit_debug_proxy.exe');
>```
>注释掉并改成：(注意地址修改)
>```javascript
>const proxy = path.resolve(__dirname, '../../node_modules/vs-libimobile/ios-webkit-debug-proxy-1.8-win64-bin/ios_webkit_debug_proxy.exe');
>```
6. 操作完成之后,可按上面的步骤开始调试