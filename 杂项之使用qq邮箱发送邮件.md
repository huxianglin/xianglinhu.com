# 杂项之使用qq邮箱发送邮件
## 本节内容
1. 特殊设置
2. 测试代码

## 1. 特殊设置
之前QQ邮箱直接可以通过smtp协议发送邮件，不需要进行一些特殊的设置，但是最近使用QQ邮箱测试的时候发现以前使用的办法无法奏效了。。。于是上网查了查，QQ对这方面做了一些限制，必须使用授权码才能登陆邮箱。官方链接在这：
[http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256](http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256 "QQ邮箱设置三方登陆")
按照上面的官方文档配置好之后就可以使用QQ邮箱发送邮件了，下面是使用方法。

## 2. 测试代码
```python
#!/usr/bin/env python
# encoding:utf-8
# __author__: send_email
# date: 2016/12/19 11:50
# blog: http://huxianglin.cnblogs.com/ http://xianglinhu.blog.51cto.com/

from email.mime.text import MIMEText
from email.header import Header
from smtplib import SMTP_SSL
import random


class Send_email:
    def __init__(self,mail_host="smtp.qq.com",mail_user="123456789@qq.com  # 这里填的是你的发件箱的邮箱名",mail_pass="这里填的不是邮箱密码，而是开启服务后的16位授权码"):
        self.mail_host=mail_host
        self.mail_user=mail_user
        self.mail_pass=mail_pass

    def send_mail(self,email):
        random_str="".join([str(random.randint(0,9)) for i in range(6)])  # 生成6位的0-9的随机数字，并转换成字符串
        mailInfo = {
                "from":self.mail_user,         #"发信人用户名@qq.com",
                "to": email,                   #"收信人用户名@qq.com",
                "hostname":"smtp.qq.com",      #qq的smtp服务器
                "username":self.mail_user,     #"账户名",
                "password":self.mail_pass,     #"密码",
                "mailsubject":"注册验证码",     #"邮件标题",
                "mailtext":random_str,
                "mailencoding":"utf-8"
                        }

        msg=MIMEText(mailInfo["mailtext"])  # 里面放需要发送的内容  #,"text",mailInfo["mailencoding"]  # 这些加上无法收到验证码
        msg['Subject']=Header(mailInfo["mailsubject"],mailInfo["mailencoding"])  # 邮件标题内容
        msg["from"] = mailInfo["from"]  # 发件人
        msg["to"] = mailInfo["to"]  # 收件人
        # server = smtplib.SMTP(self.mail_host, 25) # 这一行是以前的QQ邮箱可以直接使用smtp发送邮件
        server = SMTP_SSL(mailInfo["hostname"])  # 现在的QQ邮箱必须要SSL支持才能发送邮箱，并且不能填邮箱密码，需要在邮箱设置中打开支持POP3/SMTP功能，并获取到16位的授权码
        server.set_debuglevel(1)  # 设置debug等级 如果不想看详细日志，可以把日志级别调高一点，不需要看日志的话把这行删除就行
        server.ehlo(mailInfo["hostname"])  # 设置smtp邮箱服务器地址
        server.login(self.mail_user, self.mail_pass)  # 通过用户名和设置的授权码登录
        server.sendmail(mailInfo["from"], mailInfo["to"], msg.as_string())  # 将打包的信息发送给对方，可以将对方地址设置成一个列表或元祖，相当于群发邮件
        server.quit()  # 退出发送邮件

if __name__ == "__main__":
    obj=Send_email()
    obj.send_mail("987654321@qq.com")

```