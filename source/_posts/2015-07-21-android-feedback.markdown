---
layout: post
title: "Android-FeedBack"
date: 2014-04-16 21:24
comments: true
categories: Android
---

# 选择
之前开发OverHust的时候，需要用到用户反馈这一功能，于是就在考虑怎么实现**用户反馈**这一功能。

目前，常规方法如下：
> * 利用第三方服务，例如：[友盟](http://www.umeng.com/)

> * 调用用户已登录的邮箱帐号发送邮件

#### 利用第三方服务

`优点`：使用第三方服务，只需要调用接口，简单粗暴。

`缺点`：导入Library工程，自定义度不高。如果仅仅会用到第三方的用户反馈这一服务的话，更没必要导入别人的Library工程。

#### 调用用户已登录的邮箱帐号发送邮件

`优点`：直接调用系统的发送邮件服务，更加简单粗暴。

`缺点`：如果用户没有在系统中登录邮箱帐号，那就悲剧了。

<!--more-->

*详细了解上述两种方法各自的优劣之后,发现他们都不能很好的满足需求，因此，另谋出路。于是就在stackoverflow上发现可以用`JavaMail API`来实现。首先，不用导入一些其他Library工程，其次虽然也是用邮件服务，但是发信邮件帐号是在应用内指定的，与用户无关。当然，也是由于我自己想体验一下`JavaMail API`，之后开发应用还是更多的会用第三方服务，因为第三方服务有很多我用的着的功能。*

# 实现
具体实现是参照stackoverflow上的 [Sending Email in Android using JavaMail API without using the default/built-in app](http://stackoverflow.com/questions/2020088/sending-email-in-android-using-javamail-api-without-using-the-default-built-in-a)

首先，在工程中导入这三个jar, [mail.jar](http://javamail-android.googlecode.com/files/mail.jar) , [activation.jar](http://javamail-android.googlecode.com/files/activation.jar) , [additionnal.jar](http://javamail-android.googlecode.com/files/additionnal.jar).

之后，写几个工具类就ok了（附上我项目中的这几个类）。

[JSSEProvider,java](https://github.com/fanhongwei/OverHust/blob/master/OverHust/src/main/java/com/unique/overhust/Feedback/JSSEProvider.java)
```java
package com.unique.overhust.Feedback;

import java.security.AccessController;
import java.security.Provider;
/**
 * Created by fhw on 12/27/13.
 */
public final class JSSEProvider extends Provider {

    public JSSEProvider() {
        super("HarmonyJSSE", 1.0, "Harmony JSSE Provider");
        AccessController.doPrivileged(new java.security.PrivilegedAction<Void>() {
            public Void run() {
                put("SSLContext.TLS",
                        "org.apache.harmony.xnet.provider.jsse.SSLContextImpl");
                put("Alg.Alias.SSLContext.TLSv1", "TLS");
                put("KeyManagerFactory.X509",
                        "org.apache.harmony.xnet.provider.jsse.KeyManagerFactoryImpl");
                put("TrustManagerFactory.X509",
                        "org.apache.harmony.xnet.provider.jsse.TrustManagerFactoryImpl");
                return null;
            }
        });
    }
}
```

[GmailSender.java](https://github.com/fanhongwei/OverHust/blob/master/OverHust/src/main/java/com/unique/overhust/Feedback/GmailSender.java)
```java
package com.unique.overhust.Feedback;

import android.util.Log;

import javax.activation.DataHandler;
import javax.activation.DataSource;
import javax.mail.Authenticator;
import javax.mail.Message;
import javax.mail.PasswordAuthentication;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.security.Security;
import java.util.Properties;

/**
 * Created by fhw on 12/27/13.
 */
public class GmailSender extends Authenticator {
    private String mailhost = "smtp.gmail.com";
    private String user;
    private String password;
    private Session session;

    static {
        Security.addProvider(new JSSEProvider());
    }

    public GmailSender(String user, String password) {
        this.user = user;
        this.password = password;

        Properties props = new Properties();
        props.setProperty("mail.transport.protocol", "smtp");
        props.setProperty("mail.host", mailhost);
        props.put("mail.smtp.auth", "true");
        props.put("mail.smtp.port", "465");
        props.put("mail.smtp.socketFactory.port", "465");
        props.put("mail.smtp.socketFactory.class",
                "javax.net.ssl.SSLSocketFactory");
        props.put("mail.smtp.socketFactory.fallback", "false");
        props.setProperty("mail.smtp.quitwait", "false");

        session = Session.getDefaultInstance(props, this);
    }

    protected PasswordAuthentication getPasswordAuthentication() {
        return new PasswordAuthentication(user, password);
    }

    public synchronized void sendMail(String subject, String body, String sender, String recipients) throws Exception {
        try {
            MimeMessage message = new MimeMessage(session);
            DataHandler handler = new DataHandler(new ByteArrayDataSource(body.getBytes(), "text/plain"));
            message.setSender(new InternetAddress(sender));
            message.setSubject(subject);
            message.setDataHandler(handler);
            if (recipients.indexOf(',') > 0)
                message.setRecipients(Message.RecipientType.TO, InternetAddress.parse(recipients));
            else
                message.setRecipient(Message.RecipientType.TO, new InternetAddress(recipients));
            Transport.send(message);
            Log.e("gmailsender","ok");
        } catch (Exception e) {
            Log.e("ecption",""+e);
        }
    }

    public class ByteArrayDataSource implements DataSource {
        private byte[] data;
        private String type;

        public ByteArrayDataSource(byte[] data, String type) {
            super();
            this.data = data;
            this.type = type;
        }

        public ByteArrayDataSource(byte[] data) {
            super();
            this.data = data;
        }

        public void setType(String type) {
            this.type = type;
        }

        public String getContentType() {
            if (type == null)
                return "application/octet-stream";
            else
                return type;
        }

        public InputStream getInputStream() throws IOException {
            return new ByteArrayInputStream(data);
        }

        public String getName() {
            return "ByteArrayDataSource";
        }

        public OutputStream getOutputStream() throws IOException {
            throw new IOException("Not Supported");
        }
    }
}
```
最后，自己封装一个发送邮件的工具类。

[SendFeedback.java](https://github.com/fanhongwei/OverHust/blob/master/OverHust/src/main/java/com/unique/overhust/Feedback/SendFeedback.java)

```java
package com.unique.overhust.Feedback;

import android.content.Context;
import android.os.AsyncTask;
import android.os.Build;
import android.telephony.TelephonyManager;
import android.util.Log;

import com.unique.overhust.CommonUtils.ShareContext;

/**
 * Created by fhw on 12/27/13.
 */
public class SendFeedback {
    private String feedbackBody;
    private String feedbackContact;
    private String handsetinfo;
    private int mType;

    public SendFeedback(String fdBody, String fdContact, int type) {
        this.feedbackBody = fdBody;
        this.feedbackContact = fdContact;
        this.mType = type;
        getHandsetinfo();
        new SendEmailTask(feedbackBody, feedbackContact, mType).execute();
    }

    public String getHandsetinfo() {
        TelephonyManager tm = (TelephonyManager) ShareContext.getInstance().getSystemService(Context.TELEPHONY_SERVICE);
        handsetinfo = "手机型号:" + Build.MODEL +
                ",SDK版本:" + Build.VERSION.SDK +
                ",系统版本:" + Build.VERSION.RELEASE +
                ",手机厂商:" + Build.MANUFACTURER;
        try {
            handsetinfo = handsetinfo + ",设备ID:" + tm.getDeviceId();
        } catch (Exception e) {
            Log.e("exception",""+e);
        }
        return handsetinfo;
    }

    class SendEmailTask extends AsyncTask<Void, Void, Void> {
        private final String subject = "用户反馈";
        private final String installSubject = "安装信息";
        private final String recipients = "overhustdsnc@gmail.com";
        private String mFeedbackBody;
        private String mFeedbackContact;
        private int Type;

        public SendEmailTask(String feedbackBody, String feedbackContact, int type) {
            this.mFeedbackBody = feedbackBody;
            this.mFeedbackContact = feedbackContact;
            this.Type = type;
        }

        @Override
        protected Void doInBackground(Void... params) {
            try {
                if (Type == 1) {
                    GmailSender sender = new GmailSender("YourGmailAccount", "password");
                    sender.sendMail(subject, "反馈内容:\n" + mFeedbackBody + "\n\n" + "联系方式:\n" + mFeedbackContact + "\n\n手机信息:\n" + handsetinfo, recipients, recipients);
                }
                if (Type == 2) {
                    GmailSender sender = new GmailSender("YourGmailAccount", "password");
                    sender.sendMail(installSubject, mFeedbackBody+mFeedbackContact+"手机信息:\n" + handsetinfo, recipients, recipients);
                }
            } catch (Exception e) {
                Log.e("SendMail", e.getMessage(), e);
            }
            return null;
        }

        @Override
        protected void onPostExecute(Void result) {

        }
    }
}
```
