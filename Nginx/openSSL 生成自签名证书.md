### OpenSSL 生成自签名证书

#### 1. 简介
OpenSSL 是一个开源工具包，用于实现 SSL/TLS 协议。在本地开发环境中，常用于生成自签名证书以启用 HTTPS。

#### 2. 核心生成命令
在终端执行以下命令生成私钥 ([key.pem](key.pem)) 和证书 ([cert.pem](cert.pem))：

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -nodes -subj "/CN=localhost"
```

#### 3. 关键术语说明

*   **为什么是 `req`？**
    `req` 是 "Request" 的缩写。它是 OpenSSL 中专门用于处理 **PKCS#10 证书请求** 的子命令。
    - 默认情况下，它用于生成一个证书签名请求 (CSR)，然后发送给 CA（证书颁发机构）签名。
    - 但它也具备直接生成自签名证书的能力。

*   **为什么是 `-x509`？**
    `X.509` 是国际电信联盟 (ITU-T) 制定的一种非常通用的 **数字证书标准格式**。
    - 在命令中加入 `-x509` 标志，是告诉 OpenSSL：“我不需要生成 CSR 请求文件，请直接根据 X.509 标准给我生成一个**自签名证书**”。
    - 如果不加这个参数，OpenSSL 默认会生成一个 `.csr` 文件。

#### 4. 参数详解
*   `-newkey rsa:4096`: 同时生成新的 RSA 4096 位私钥。
*   `-keyout`: 指定私钥的保存路径。
*   `-out`: 指定证书的保存路径。
*   `-sha256`: 使用 SHA-256 摘要算法。
*   `-days 365`: 证书有效期为 365 天。
*   `-nodes`: (No DES) 不对私钥进行加密，Node.js 读取时无需输入密码。
*   `-subj "/CN=localhost"`: 指定证书主体信息，`CN` (Common Name) 设为 `localhost`。

#### 5. 在 Node.js 中使用
在 [index.mjs](index.mjs) 中引用生成的证书文件：

```javascript
const options = {
    key: fs.readFileSync(path.join(__dirname, 'key.pem')),
    cert: fs.readFileSync(path.join(__dirname, 'cert.pem'))
};

https.createServer(options, app).listen(PORT, () => {
    console.log(`Server is running on https://localhost:${PORT}`);
});
```

#### 6. 注意事项
*   **浏览器警告**: 由于是自签名证书，浏览器会提示“您的连接不是私密连接”。开发时点击“高级” -> “继续前往”即可。
*   **安全性**: 自签名证书仅建议用于本地开发或内网测试。
*   **验证证书**: 使用以下命令查看证书详情：
    ```bash
    openssl x509 -in cert.pem -text -noout
    ```