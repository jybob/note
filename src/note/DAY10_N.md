# 1. 处理密码加密

用户的密码必须被加密后再存储到数据库，否则，就存在用户账号安全问题。

用户使用的原始密码通常称之为“原文”或“明文”，经过算法的运算，得到的结果通常称之为“密文”。

在处理密码加密时，不可以使用任何**加密算法**，因为所有加密算法都是可以被逆向运算的，也就是说，当密文、算法、加密参数作为已知条件的情况下，是可以根据密文计算得到原文的！

> 提示：加密算法通常是用于保障数据传输过程的安全的，并不适用于存储下来的数据安全！

对存储的密码进行加密处理，通常使用**消息摘要算法**！

消息摘要算法的特点：

- 消息（原文、原始数据）相同，则摘要相同
- 无论消息多长，每个算法的摘要结果长度固定
- 消息不同，则摘要极大概率不会相同

**注意：消息摘要算法是不可逆向运算的算法！即你永远不可能根据摘要（密文）逆向计算得到消息（原文）！**

常见的消息摘要算法有：

- SHA系列：`SHA-1`、`SHA-256`、`SHA-384`、`SHA-512`
- MD家族：`MD2`、`MD4`、`MD5`

Spring框架内有`DigestUtils`的工具类，提供了MD5的API，例如：

```java
package cn.tedu.csmall.product;

import org.junit.jupiter.api.Test;
import org.springframework.util.DigestUtils;

public class MessageDigestTests {

    @Test
    public void testMd5() {
        String rawPassword = "123456";
        String encodedPassword = DigestUtils
                .md5DigestAsHex(rawPassword.getBytes());
        System.out.println("原文：" + rawPassword);
        System.out.println("密文：" + encodedPassword);
        System.out.println();

        rawPassword = "123456";
        encodedPassword = DigestUtils
                .md5DigestAsHex(rawPassword.getBytes());
        System.out.println("原文：" + rawPassword);
        System.out.println("密文：" + encodedPassword);
        System.out.println();

        rawPassword = "1234567890ABCDEFGHIJKLMN";
        encodedPassword = DigestUtils
                .md5DigestAsHex(rawPassword.getBytes());
        System.out.println("原文：" + rawPassword);
        System.out.println("密文：" + encodedPassword);
    }

}
```

以上测试的运行结果为：

```
原文：123456
密文：e10adc3949ba59abbe56e057f20f883e

原文：123456
密文：e10adc3949ba59abbe56e057f20f883e

原文：1234567890ABCDEFGHIJKLMN
密文：41217c45889b5378c3dad3879d7bfac9
```

> 提示：在项目中添加`commons-codec`的依赖项，可以使用更多消息摘要算法的API。

未完待续！



