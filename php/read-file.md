# read properties file

```php
// open file
$file_name = fopen("./MailSend.properties", "r") or die("Unable to open file!");

while(!feof($file_name)) {
    // read line from file
  $line=fgets($file_name);
    // 以等号为分隔符将行内容分割到数组中
    // explode('=', $line)[0]为等号左边的内容
  switch(explode('=', $line)[0]){
    case "IPアドレス":
        $strMailServer = trim(explode('=', $line)[1]);
        break;
    case "ポート番号":
        $strMailPort = trim(explode('=', $line)[1]);
        break;
  }
}
// close file
fclose($file_name);
```

