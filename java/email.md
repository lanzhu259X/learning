# JAVA 发送邮件

> 本例子中使用是使用 javax.mail jar 包建立邮箱客户端，进行发送邮件。

maven 的依赖:

```xml
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>mail</artifactId>
    <version>1.4.7</version>
</dependency>
```

### 本例子中使用的腾讯企业邮箱

> 在腾讯企业邮箱中，需要在 `设置` -> `账户` 中开启客户端验证和获取对应的客户端验证授权密码.

具体开发代码如下:

```java
package com.yiban.sharing.utils;

import com.alibaba.fastjson.JSON;
import com.sun.mail.util.MailSSLSocketFactory;
import lombok.extern.slf4j.Slf4j;

import javax.activation.DataHandler;
import javax.activation.FileDataSource;
import javax.mail.*;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeMessage;
import javax.mail.internet.MimeMultipart;
import java.io.File;
import java.security.GeneralSecurityException;
import java.util.Date;
import java.util.List;
import java.util.Properties;

@Slf4j
public class EmailClientUtil {

    private static final Properties E_PROPS;
    private static final Authenticator AUTHENTICATOR;
    private static final String FROM_MAIL = "xxxxx@yibanjf.com";

    static {
        E_PROPS = new Properties();
        E_PROPS.setProperty("mail.transport.protocol", "smtp");
        E_PROPS.setProperty("mail.smtp.host", "smtp.exmail.qq.com");
        E_PROPS.setProperty("mail.smtp.port", "465");
        E_PROPS.setProperty("mail.smtp.auth", "true");
        MailSSLSocketFactory sf = null;
        try {
            sf = new MailSSLSocketFactory();
            sf.setTrustAllHosts(true);
        } catch (GeneralSecurityException e) {
            log.error("Mail SSL init Fail.", e);
        }
        E_PROPS.put("mail.smtp.ssl.enable", "true");
        E_PROPS.put("mail.smtp.ssl.socketFactory", sf);

        AUTHENTICATOR = new Authenticator() {
            @Override
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication("xxxxx@yibanjf.com", "你的客户端授权码");
            }
        };
    }

    public static Session getSession() {
        return Session.getDefaultInstance(E_PROPS, AUTHENTICATOR);
    }

    /**
     * 邮件发送，内容可以是文本信息或者HTML信息
     * @param session 如果一次发送多个邮件时，不用每个邮件都创建一个session，用一个就好
     * @param subject 主题
     * @param toArr 发送给谁
     * @param text 邮件内容
     * @param files 附件
     */
    public static void sendMail(Session session, String subject, List<String> toArr, String text, List<File> files) {
        if (toArr == null || toArr.isEmpty()) {
            log.warn("send mail but to users is empty.");
            return;
        }
        try {
            MimeMessage message = createMessage(session, subject, toArr, text, files);
            Transport transport = session.getTransport();
            transport.connect();
            transport.sendMessage(message, message.getRecipients(Message.RecipientType.TO));
            transport.close();
            log.info("send mail to:{} success.", JSON.toJSONString(toArr));
        }catch (Exception e) {
            log.error("send mail fail.", e);
        }
    }

    private static MimeMessage createMessage(Session session, String subject, List<String> toList, String text, List<File> files) throws Exception {
        InternetAddress[] toArr = new InternetAddress[toList.size()];
        for (int i = 0; i < toList.size(); i++) {
            toArr[i] = new InternetAddress(toList.get(i));
        }
        MimeMessage message = new MimeMessage(session);
        message.setFrom(new InternetAddress(FROM_MAIL));
        message.setRecipients(Message.RecipientType.TO, toArr);
        message.setSubject(subject);
        message.setSentDate(new Date());

        MimeBodyPart textPart = new MimeBodyPart();
        textPart.setContent(text, "text/html;charset=UTF-8");
        MimeMultipart allPart = new MimeMultipart();
        allPart.addBodyPart(textPart);

        if (files != null && !files.isEmpty()) {
            for (File file : files) {
                MimeBodyPart part = new MimeBodyPart();
                part.setDataHandler(new DataHandler(new FileDataSource(file)));
                part.setFileName(file.getName());
                allPart.addBodyPart(part);
            }
        }
        message.setContent(allPart);
        message.saveChanges();
        return message;
    }
}
```
