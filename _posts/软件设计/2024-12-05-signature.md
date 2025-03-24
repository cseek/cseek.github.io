---
title: 使用 c++ 实现对文件签名和验证
category: [软件设计]
tags: [signature, openssl]
---

> 之前在实现 SOTA 升级之前使用 Python 实现了签名认证的功能，今天采用 c++ 重构之前的代码，方便集成到 SOTA 代码里。
{: .prompt-info }

## 代码
`ec_sig.cpp`

```c++
#include <iostream>
#include <fstream>
#include <vector>
#include <openssl/evp.h>
#include <openssl/pem.h>
#include <openssl/ec.h>
#include <openssl/err.h>

// 错误处理函数
void handle_errors(const char* context = nullptr) {
    char buf[1024];
    ERR_error_string_n(ERR_get_error(), buf, sizeof(buf));
    std::cerr << "Error [" << (context ? context : "general") << "]: " << buf << std::endl;
    exit(1);
}

EVP_PKEY* load_ec_private_key(const char* key_path) {
    BIO* bio = BIO_new_file(key_path, "r");
    if (!bio) handle_errors("open private key");
    
    EVP_PKEY* pkey = PEM_read_bio_PrivateKey(bio, nullptr, nullptr, nullptr);
    BIO_free(bio);
    
    if (!pkey || EVP_PKEY_id(pkey) != EVP_PKEY_EC) {
        std::cerr << "Invalid ECDSA private key format" << std::endl;
        exit(1);
    }
    return pkey;
}

EVP_PKEY* load_ec_public_key(const char* key_path) {
    BIO* bio = BIO_new_file(key_path, "r");
    if (!bio) handle_errors("open public key");
    
    EVP_PKEY* pkey = PEM_read_bio_PUBKEY(bio, nullptr, nullptr, nullptr);
    BIO_free(bio);
    
    if (!pkey || EVP_PKEY_id(pkey) != EVP_PKEY_EC) {
        std::cerr << "Invalid ECDSA public key format" << std::endl;
        exit(1);
    }
    return pkey;
}

bool ecdsa_sign_file(const char* file_path, 
                    const char* key_path,
                    const char* sig_path) {
    EVP_PKEY* pkey = load_ec_private_key(key_path);
    EVP_MD_CTX* ctx = EVP_MD_CTX_new();
    if (!ctx) handle_errors("create sign context");

    if (EVP_DigestSignInit(ctx, nullptr, EVP_sha256(), nullptr, pkey) != 1)
        handle_errors("sign init");

    std::ifstream file(file_path, std::ios::binary);
    if (!file) {
        std::cerr << "Cannot open input file: " << file_path << std::endl;
        return false;
    }

    // 读取整个文件到内存（适用于小文件）
    file.seekg(0, std::ios::end);
    size_t file_size = file.tellg();
    file.seekg(0, std::ios::beg);
    
    std::vector<unsigned char> data(file_size);
    if (!file.read(reinterpret_cast<char*>(data.data()), file_size)) {
        std::cerr << "Read file failed: " << file_path << std::endl;
        return false;
    }

    size_t sig_len;
    if (EVP_DigestSign(ctx, nullptr, &sig_len, data.data(), data.size()) != 1)
        handle_errors("get signature length");

    std::vector<unsigned char> signature(sig_len);
    if (EVP_DigestSign(ctx, signature.data(), &sig_len, data.data(), data.size()) != 1)
        handle_errors("finalize signature");

    std::ofstream sig_file(sig_path, std::ios::binary);
    if (!sig_file) {
        std::cerr << "Cannot create signature file: " << sig_path << std::endl;
        return false;
    }
    sig_file.write(reinterpret_cast<char*>(signature.data()), sig_len);

    std::cout << "Signature generated (" << sig_len << " bytes)" << std::endl;

    EVP_MD_CTX_free(ctx);
    EVP_PKEY_free(pkey);
    return true;
}

bool ecdsa_verify_signature(const char* file_path,
                           const char* key_path,
                           const char* sig_path) {
    EVP_PKEY* pkey = load_ec_public_key(key_path);
    EVP_MD_CTX* ctx = EVP_MD_CTX_new();
    if (!ctx) handle_errors("create verify context");

    if (EVP_DigestVerifyInit(ctx, nullptr, EVP_sha256(), nullptr, pkey) != 1)
        handle_errors("verify init");

    std::ifstream file(file_path, std::ios::binary);
    if (!file) {
        std::cerr << "Cannot open input file: " << file_path << std::endl;
        return false;
    }

    file.seekg(0, std::ios::end);
    size_t file_size = file.tellg();
    file.seekg(0, std::ios::beg);
    
    std::vector<unsigned char> data(file_size);
    if (!file.read(reinterpret_cast<char*>(data.data()), file_size)) {
        std::cerr << "Read file failed: " << file_path << std::endl;
        return false;
    }

    std::ifstream sig_file(sig_path, std::ios::binary);
    if (!sig_file) {
        std::cerr << "Cannot open signature file: " << sig_path << std::endl;
        return false;
    }

    sig_file.seekg(0, std::ios::end);
    size_t sig_len = sig_file.tellg();
    sig_file.seekg(0, std::ios::beg);
    
    std::vector<unsigned char> signature(sig_len);
    if (!sig_file.read(reinterpret_cast<char*>(signature.data()), sig_len)) {
        std::cerr << "Read signature failed" << std::endl;
        return false;
    }

    int result = EVP_DigestVerify(ctx, signature.data(), sig_len, 
                                 data.data(), data.size());

    EVP_MD_CTX_free(ctx);
    EVP_PKEY_free(pkey);

    return result == 1;
}

int main() {
    const char* private_key = "ec_private.pem";
    const char* public_key = "ec_public.pem";
    const char* data_file = "document.bin";
    const char* sig_file = "signature.der"; // 改为 .der 扩展名

    // 生成签名
    if (ecdsa_sign_file(data_file, private_key, sig_file)) {
        std::cout << "\n--- 签名成功 ---" << std::endl;
    } else {
        std::cerr << "\n!!! 签名失败 !!!" << std::endl;
        return 1;
    }

    // 验证签名
    if (ecdsa_verify_signature(data_file, public_key, sig_file)) {
        std::cout << "\n=== 验证成功 ===" << std::endl;
    } else {
        std::cerr << "\n!!! 验证失败 !!!" << std::endl;
        return 1;
    }

    return 0;
}
```

## 验证步骤

1. 生成新密钥对
```bash
openssl ecparam -name prime256v1 -genkey -noout -out ec_private.pem
openssl ec -in ec_private.pem -pubout -out ec_public.pem
```

2. 创建测试文件
```bash
echo "Important Data $(date)" > document.bin
```

3. 编译并运行
```bash
g++ -std=c++11 -Wall ec_sig.cpp -o ec_sig -lssl -lcrypto
./ec_sig
```

4. 手动验证签名
```bash
# 生成签名
openssl dgst -sha256 -sign ec_private.pem -out openssl_sig.der document.bin
# 比较两个签名文件
diff signature.der openssl_sig.der
# 使用OpenSSL验证
openssl dgst -sha256 -verify ec_public.pem -signature signature.der document.bin
```

5. 输出
```bash
Signature generated (72 bytes)
--- 签名成功 ---
=== 验证成功 ===
```