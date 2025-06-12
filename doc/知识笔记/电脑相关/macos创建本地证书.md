---
tags:
  - 知识笔记/电脑相关
---

>[!Info] 创建证书（证书生成在执行目录）
> ```sh
> openssl req -x509 -newkey rsa:2048 -sha256 -days 3650 \
>   -nodes \
>   -keyout somnuszyy9527.local-key.pem \
>   -out somnuszyy9527.local.pem \
>   -subj "/CN=somnuszyy9527.local" \
>   -addext "subjectAltName=DNS:somnuszyy9527.local"
> ```


> [!Info] 让系统信任证书（在证书所在目录执行）
> ```sh
> sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain somnuszyy9527.local.pem
> ```

