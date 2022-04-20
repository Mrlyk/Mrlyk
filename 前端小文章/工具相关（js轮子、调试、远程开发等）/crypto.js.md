# CryptoJs

> 前端加密库，支持各种常用的加密方式，下面简称 CJ。

[GIthub仓库](https://github.com/brix/crypto-js)

`npm install crypto-js`

## AES 加密

AES 加密是一种对称加密方式，通过前后端协商好的密钥进行加密和解密。

AES 加密分为 aes-128 、aes-256 ，视传入的 key 的长度而定。下面是使用示例

```js
importy CryptoJs from 'Crypto-js'

// 加密
encrypt (data, keyStr) {
    keyStr = keyStr || GLOBAL_CONFIG.AES_KEY; // 获取密钥
    const key = CryptoJS.enc.Utf8.parse(keyStr); // 把 key 从 utf-8 转码成 CJ 接收的数据类型 WordArray。也可以从 base64，16 进制字符串转过来
    const iv = CryptoJS.enc.Utf8.parse(this.iv); // utf-8 转码 偏移向量，CBC 的混淆模式需要这个偏移量。EBC 则不需要，也是由前后端协商产生
    const encrypted = CryptoJS.AES.encrypt(data, key, {
      iv,
      mode: CryptoJS.mode.CBC,
      padding: CryptoJS.pad.Pkcs7
    });
    return encrypted.toString(); // 默认返回 base64 格式的加密数据，接收一个回调函数来指定返回的编码格式 encrypted.toString(CryptoJs.enc.Utf8)
  },
  // 解密
  decrypt (data, keyStr) {
    keyStr = keyStr || GLOBAL_CONFIG.AES_KEY;
    const key = CryptoJS.enc.Utf8.parse(keyStr);
    const iv = CryptoJS.enc.Utf8.parse(this.iv);
    const decrypt = CryptoJS.AES.decrypt(data, key, {
      iv,
      mode: CryptoJS.mode.CBC,
      padding: CryptoJS.pad.Pkcs7
    });
    return decrypt.toString(CryptoJS.enc.Utf8); // 返回 utf8 格式的字符
  },
```

