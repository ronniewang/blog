# SMTP, POP3, IMAP

## 概述

SMTP, POP3还有IMAP都是用来递送邮件的TCP/IP协议，如果你使用一个邮件服务器，例如hMailServer，你需要了解他们都是干嘛的，每个协议都代表了一种信息传递的一系列规则

## SMTP

SMTP全称Simple Mail Transfer Protocol，SMTP有两个功能
* 用于将email从客户端发送到服务器，例如使用Outlook
* 用于把邮件从一个服务器递送到另一个服务器

SMTP默认端口是25

## POP3

POP3全称Post Office Protocol，POP3可以允许客户端从服务器下载邮件，POP3很简单，除了下载之外不提供其他功能
原POP3协议，在客户端下载所有邮件后，邮件会从服务器端删除
改进的POP3协议，也可实现不删除
POP3默认端口为110

## IMAP

IMAP stands for Internet Message Access Protocol. IMAP shares many similar features with POP3. It, too, is a protocol that an email client can use to download email from an email server. However, IMAP includes many more features than POP3. The IMAP protocol is designed to let users keep their email on the server. IMAP requires more disk space on the server and more CPU resources than POP3, as all emails are stored on the server. IMAP normally uses port 143. Here is more information about IMAP.

## Examples

Suppose you use hMailServer as your email server to send an email to bill@microsoft.com.
You click Send in your email client, say, Outlook Express.
Outlook Express delivers the email to hMailServer using the SMTP protocol.
hMailServer delivers the email to Microsoft's mail server, mail.microsoft.com, using SMTP.
Bill's Mozilla Mail client downloads the email from mail.microsoft.com to his laptop using the POP3 protocol (or IMAP).
