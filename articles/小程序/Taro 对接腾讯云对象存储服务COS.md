Taro 是一套遵循 React 语法规范的 多端开发 解决方案，所以一开始打算使用`cos-js-sdk-v5`，结果发现一直userAgent报错，代理出错，查看源码`cos-js-sdk-v5/lib/request.js`，才发现了自己忽略了一个致命错误。

`cos-js-sdk-v5`使用的是`jquery`请求，而小程序是无法使用`jquery`的，主要是因为：小程序的页面逻辑是在`JsCore`中运行，`JsCore`是一个没有窗口对象的环境，所以脚本中不能使用`window`对象，也无法在脚本中操作组件，而`jquery`会使用到`window`对象和`document`对象，所以无法使用。

进而，使用腾讯云对象储存服务对接小程序的[cos-wx-sdk-v5](https://github.com/tencentyun/cos-wx-sdk-v5),按照文档直接拖到项目了，运行正确，文件成功上传到腾讯云，但回调`function (err, data)`始终不运行。怎么办，只有一点点排查，在原生中是正常的，那么只能是Taro编译的时候出问题了，所以我们这样解决，在Taro编译时，忽略`cos-wx-sdk-v5`文件，我们在`config/index.js`配置如下：
```js
...
weapp: {
    compile: {
      exclude:['src/libs/cos-wx-sdk-v5.js'] // 忽略的文件位置数组
    },
    module: {
      postcss: {
        autoprefixer: {
          enable: true,
          config: {
            browsers: [
              'last 3 versions',
              'Android >= 4.1',
              'ios >= 8'
            ]
          }
        },
        pxtransform: {
          enable: true,
          config: {

          }
        },
        url: {
          enable: true,
          config: {
            limit: 10240 // 设定转换尺寸上限
          }
        },
        cssModules: {
          enable: false, // 默认为 false，如需使用 css modules 功能，则设为 true
          config: {
            namingPattern: 'module', // 转换模式，取值为 global/module
            generateScopedName: '[name]__[local]___[hash:base64:5]'
          }
        }
      }
    }
  },
...
```
同时奉上部分代码实现：
```js

import { UPLOADER_SERVER_URL } from "../vendor/lib/urls";
import net from '../vendor/index'
import {log, GUID, get_suffix, showLoading} from "./utils";
import COS from '../libs/cos-wx-sdk-v5'

var Bucket = 'XXX'
var Region = 'ap-shanghai'

var getAuthorization = function (options, callback) {
  net.request({
    url:UPLOADER_SERVER_URL,
    method: 'GET',
    success (res) {
      log('getAuthorization', res)
      let token = res.token
      callback({
        TmpSecretId: token.credentials.tmpSecretId,
        TmpSecretKey: token.credentials.tmpSecretKey,
        XCosSecurityToken: token.credentials.sessionToken,
        ExpiredTime: parseInt(token.expiredTime)*1000, // SDK 在 ExpiredTime 时间前，不会再次调用 getAuthorization
      })
    }, fail (err) {
      log('getAuthorization err', err)
    }
  })
}

let cos = new COS({
  // path style 指正式请求时，Bucket 是在 path 里，这样用途相同园区多个 bucket 只需要配置一个园区域名
  // ForcePathStyle: true,
  getAuthorization: getAuthorization,
})

const cosUploader = {
  cos,
  uploaderFile (filePath) {
    return new Promise((resolve, reject) => {
      var filename = 'product-images/' + GUID(32, 62) + get_suffix(filePath)
      showLoading('上传中')
      cos.postObject({
        Bucket: Bucket,
        Region: Region,
        Key: filename,
        FilePath: filePath,
        onProgress: function (info) {
          console.log(JSON.stringify(info));
        }
      }, function (err, data) {
        log('err || data', err || data)
        if (err) {
          reject(err)
        } else {
          resolve(data)
        }
      })
    })
  },
  uploaderFiles (files, callback) {
    let arr = files.map((item) => {
      return this.uploaderFile(item)
    })
    Promise.all(arr).then(function (res) {
      callback(null, res)
    }, function (err) {
      callback(err, null)
    })
    return arr
  }
}

export default cosUploader
```