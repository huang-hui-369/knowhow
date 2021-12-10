```sh
Linux# telnet mail.ash.jp 25 

Trying 210.154.87.18 ... 
Connected to mail.ash.jp Escape character is '^]'. 
220 mail.ash.jp ESMTP Sendmail 8.9.1/3.7W; Tue, 29 Sep 1998 12:30: 13 +0900 (JST) 

HELO foo.or.jp 
250 mail.ash.jp Hello pc20.lo.ash.jp [10.0.1.20], 
pleased to meet you 

MAIL FROM: user@foo.or.jp 
250 user@foo.or.jp... Sender ok 

RCPT TO: joe@ash.jp 
250 joe@ash.jp... Recipient ok 

DATA 
354 Enter mail, end with "." on a line by itself 
From: user@foo.or.jp 
Subjet: test 
Hello world. 
. 
250 NAA02891 Message accepted for delivery 

QUIT 
221 mail.ash.jp closing connection

```