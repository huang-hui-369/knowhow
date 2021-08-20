```
explain
 SELECT  *
 from t_sendmail_history
 where  match(mail_text, title) against('テスト３' in boolean mode);
```