# 4d-component-sendmail
Easily send mail without using Internet Commands.

About
---
This is a [libcurl](http://curl.haxx.se) based 4D component (v14+) that allows you to send email in one line.

Consider it a [SMTP_QuickSend](http://doc.4d.com/4Dv15/4D-Internet-Commands/15/SMTP-QuickSend.301-2397900.ja.html) replacement.

**Why not Internet Commands?**

There is a bug in the plugin (or, perhaps it is standard behaviour) where some characters in an ISO-2022-JP encoded emails using the CP932 (Windows-31J) character set (a.k.a. JIS X 0208 of 1990) are lost in transmission. It means that __you can't reliably send messages to dumbphones that do not support Unicode__ (and there are many of them, especially in the business scene). In addition, you can't send attachments with ```SMTP_QuickSend```.

This component does its own MIME encoding using regular string commands.

It also accepts multiple attachments specified with ```ARRAY OBJECT```.

It also exposes a set of low level MIME APIs, that take ```C_OBJECT``` as a reference.

Examples
---

Procedurally create MIME
```
  //ISO-2022-JPで拡張文字が含まれるメールを構築
$sample:="ｱｲｳｴｵ髙橋あいうえお①②③"

$message:=MIME_New ("iso-2022-jp")
  //useExtendedJISX0208: 非公式（Shift]JISではなく，Windows-31J/CP932に基づくISO-2022-JP変換を実行します。）
MIME_SET_OPTION ($message;"useExtendedJISX0208";"true")
MIME_ADD_HEADER ($message;"From";"MIYAKO <keisuke.miyako@4d.com>")
MIME_ADD_HEADER ($message;"To";"MIYAKO <keisuke.miyako@4d.com>")
MIME_ADD_HEADER ($message;"Subject";$sample*5)
MIME_SET_BODY ($message;$sample*5)
MIME_ADD_ATTACHMENT ($message;"4D.png";$sample*5+".png";"image/png")

$MIME:=MIME_Export_to_variable ($message)
  //メールソフトで開く
$path:=Temporary folder+Generate UUID+".eml"
TEXT TO DOCUMENT($path;$MIME;"us-ascii")
OPEN WEB URL($path)
```

Send non-Unicode message with attachments

```
  //Internet Commandを使用せずにメールを送信
$hostName:="smtp.gmail.com"
$msgFrom:="miyako.keisuke@gmail.com"
$msgTo:="keisuke.miyako@4d.com"
$subject:="ｱｲｳｴｵ髙橋あいうえお①②③"
$message:="ｱｲｳｴｵ髙橋あいうえお①②③"
$sessionParam:=0  //TLS
$port:=465
$userName:="miyako.keisuke"
$password:=""
$encoding:="iso-2022-jp"
  //添付ファイルはオブジェクトの配列で
ARRAY OBJECT($attachments;0)
C_OBJECT($attachment)
OB SET($attachment;"name";"添付ファイル.png")  //ファイル名は拡張文字を使用しないほうが無難でしょう
OB SET($attachment;"type";"image/png")
OB SET($attachment;"data";"4D.png")  //リソースパス, 絶対パス, またはbas64データ
APPEND TO ARRAY($attachments;$attachment)
  //curlのエラーコード
$err:=Sendmail ($hostName;$msgFrom;$msgTo;$subject;$message;$sessionParam;$port;$userName;$password;$encoding;->$attachments)
```



