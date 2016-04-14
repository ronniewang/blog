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

IMAP全称Internet Message Access Protocol，IMAP与POP3有很多相似的特性，也可以从服务器上下载邮件到客户端，但IMAP还有更多的额外功能，IMAP是服务端不删除邮件的，所以比POP3需要更多的磁盘和cpu资源

IMAP的more端口是143
