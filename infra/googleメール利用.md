# googleメール

- 680/月
- 送信回数、送信宛先数の制限があり

## smtp サーバー

  <mail-host>smtp.gmail.com</mail-host>
  <mail-username>notification@futari-passport.metro.tokyo.lg.jp</mail-username>
  <mail-password>Grandunit123@</mail-password>

## 受信サーバードメインの設定

1. ドメインの所有権を証明

2. MX レコードを設定する

   | 名前、ホスト、エイリアス | TTL（Time to Live）* | レコードタイプ | 優先値 | 値、応答、参照先        |
   | :----------------------- | :------------------- | :------------- | :----- | :---------------------- |
   | *@ または空欄*           | 3,600                | MX             | 1      | ASPMX.L.GOOGLE.COM      |
   | *@ または空欄*           | 3,600                | MX             | 5      | ALT1.ASPMX.L.GOOGLE.COM |
   | *@ または空欄*           | 3,600                | MX             | 5      | ALT2.ASPMX.L.GOOGLE.COM |
   | *@ または空欄*           | 3,600                | MX             | 10     | ALT3.ASPMX.L.GOOGLE.COM |
   | *@ または空欄*           | 3,600                | MX             | 10     | ALT4.ASPMX.L.GOOGLE.COM |



## Gmail のセキュリティを設定する

- **SPF**: あるドメインから送信されたように見えるメールが、ドメイン所有者によって承認されたサーバーから送信されたものであることを受信サーバーで確認できます。
- **DKIM**: すべてのメールにデジタル署名が追加され、この署名によって、メールが偽装されていないことと配信中に改変されていないことを受信サーバーで確認できます。
- **DMARC**: SPF と DKIM でメールを認証し、不審な受信メールを管理できます。
- https://support.google.com/a/answer/10583557

https://support.google.com/a/answer/140034?hl=ja&ref_topic=2683820