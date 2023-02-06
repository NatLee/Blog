---
title: 資安常識(3) - 簽驗章流程與完整憑證解析
categories: study
tags:
  - cryptograph
  - security
  - signature
  - certificate
  - encoding
  - decoding
abbrlink: 736906820
date: 2021-10-06 00:00:00
---


## 前言
----------

上次提到的加密演算法『公鑰加密，私鑰解密』達成了PKI中傳遞資料的隱密性，以及雜湊演算法的低碰撞性可以檢查傳遞資料的完整性。

那麼，身份鑑別跟不可否認性呢？我們來看看『私鑰加密，公鑰解密』，簽驗章流程與憑證是什麼。

<!--more-->

## 內容
----------

### 數位簽章
----------

#### 何謂數位簽章
----------

什麼是數位簽章(digital signature)？

我們還是先來看看[維基百科](https://zh.wikipedia.org/zh-tw/%E6%95%B8%E4%BD%8D%E7%B0%BD%E7%AB%A0)的說法：

> 數位簽章（Digital Signature），又稱公鑰數位簽章，是一種功能類似寫在紙上的普通簽名、但是使用了公鑰加密領域的技術，以用於鑑別數位訊息的方法。一套數位簽章通常會定義兩種互補的運算，一個用於簽名，另一個用於驗證。法律用語中的電子簽章與數位簽章代表之意義並不相同。電子簽章指的是依附於電子文件並與其相關連，用以辨識及確認電子文件簽署人身分、資格及電子文件真偽者；數位簽章則是以數學演算法或其他方式運算對其加密而形成的電子簽章。意即並非所有的電子簽章都是數位簽章。

維基百科的解釋落落長，我們根據上面的敘述拆出三個關鍵字來分別定義：

1. 普通簽名
	一般我們在現實生活中，常常需要簽署文件表示同意或是認可，這就是普通簽名。

2. 電子簽章
	也叫做電子簽名，但在網路世界中，我們沒辦法拿筆進行簽名，所以我們必須要依附電子文件的方式來進行簽署。在我們不能違背不可否認性的狀況下，就需要有第三方平臺來確保雙方電子簽名的公正性，如有一些提供這項服務的平臺，像是Adobe Sign、DocuSign等等。

3. 數位簽章
	跟電子簽章類似，在網路世界裡，我們需要證明自己身份的話，可以遵循公開金鑰系統來建立一對公私鑰，私鑰自行保管，公鑰盡量散佈。那麼，我們使用私鑰來對資料加密，別人可以用我們散佈的公鑰解密。這樣即可證明這資料是我認可的，也可以證明是由我發出的，以此達到確認身份，也就是身份鑑別的效果。

其實，光看這些名字跟解釋很容易被誤導以爲是同個東西，但並不盡然，它們差一個字就差千萬里。

然而，全世界有使用電子簽章或數位簽章的地方都有以政府機關爲首去定義這些名詞。

在台灣也有個法條，叫做[電子簽章法](https://law.moj.gov.tw/LawClass/LawAll.aspx?PCode=J0080037)，裏面定義的這些名詞，都是不同的東西。

我們以下所有的數位簽章都以簽章爲代稱。

我們知道什麼是簽章之後，接下來我們來看看簽章跟驗章的流程是怎麼跑的。

#### 簽驗章流程
----------

我們在瞭解簽驗章流程的時候，一定要知道我們爲何而做。

就是爲了知道對方身份以及判斷拿到的資料是否爲對方所認可的。

那麼，我們有請兩位常常出現在各大資安課本的角色 Alice 跟 Bob 出場表演。

##### 簽章

![簽章](https://i.imgur.com/0NJQNkq.png)

圖片中的順序我再用文字詳細打一次。

1. Alice對文件產生雜湊，這個文件其實就是資料。
2. Alice用Alice的**私鑰**對文件的雜湊值，也就是對摘要進行加密，產生Alice的簽名。
3. Alice將文件跟簽名傳送給Bob。

從這邊我們可以看出Alice的私鑰可以用Alice的公鑰去解密，也就是我們可以知道是Alice去簽署了這個資料。

那這邊有個問題是爲什麼我們的不是拿私鑰直接對資料加密，而是對資料產生的雜湊值加密？

仔細想想，爲什麼我們不這樣做，除了保障文件完整，我們還直接少了一個步驟，這樣不很好嗎？

這是因爲我們的文件有可能很大，非對稱加密的時間會遠超過我們預期，也就是越大的文件，我們需要加密越久。

相對的，我們對固定長度的雜湊值進行加密，這樣的做法來得比較有效率。

關於這個問題，這邊也有個答案可以參考看看，[Why is hashing (digest) employed in digital signatures?](https://security.stackexchange.com/questions/171974/why-is-hashing-digest-employed-in-digital-signatures)。

那麼，另外一個問題是萬一我的文件，也就是資料很小，小到比金鑰還短，甚至比雜湊值
短，這樣加密不就比起對雜湊值加密來得更有效率嗎？

就算資料比金鑰還短，我們在非對稱加密有說過，一樣得padding到金鑰長度才能加密。所以以RSA-1024來說也不可能比雜湊值短。

那如果我們使用能夠產生較短金鑰長度的加密算法ECC家族的ECDSA-224來說的話，這樣或許有可能資料長度比雜湊SHA-256來得短。但這種狀況多嗎？其實並不多。

但談論這些並沒有太大的意義，因爲我們一開始所說的RFC裏面就已經定義了數位簽章。

可以參照[PKCS#7](https://datatracker.ietf.org/doc/html/rfc2315)，裏面有定義簽章是對message的digest加密。


##### 驗章

![驗章](https://i.imgur.com/L6pD9F2.png)

4. Bob用Alice散佈在外的公鑰對簽名進行解密。
5. Bob對文件生成雜湊值。
	這邊會有疑問的是，要如何知道是哪一種雜湊？
	答案請參照[PKCS#7](https://datatracker.ietf.org/doc/html/rfc2315)，裏面有定義簽章的雜湊類型跟更詳細的傳送流程。
6. 比對解密後跟自己生成的雜湊值是否一致，以此確認文件是否完整。
	然後Bob能使用Alice的公鑰解密成功，其實也代表簽名是Alice的。
	其中，任何一項出錯都會導致驗章失敗。

剛剛有一個問題是，爲什麼我們不對文件加密，這邊也可以看出來最後我們比對的是雜湊值，如果我們文件很大的話，比對大文件耗的時間肯定比固定長度的雜湊值來得多。



### 憑證
----------

#### 何謂憑證
----------

剛剛提到簽章就是用自己的私鑰去對文件的雜湊進行簽署（加密），可是問題來了！

大家都可以隨意產生公私鑰對，用OpenSSL就可以做，我不就可以亂產生公鑰散佈到網路上宣稱我是別人。

這樣的話，我們要怎麼知道這個簽章到底是不是你簽出來的？

這邊就該是憑證登場的時候了，也就是我們需要第三方來證明這個公鑰到底能不能夠被信任，這樣才能達成我們一開始說到的『私鑰加密，公鑰解密』。那麼，這個第三方就是PKI內的角色CA。

我們在PKI那邊有提到憑證申請的時候，CA會打開你的CSR（申請資料及公鑰），然後抽出裏面的資料用CA自己的私鑰對它加密以證明你的身份及這份公鑰是由我這個CA擔保的。

這邊又有一個疑問，我用OpenSSL也可以自己當CA啊！那不就可以自己隨便幫別人簽憑證了嗎？

沒錯！你是可以隨便幫別人簽，但你簽的憑證能夠被信任嗎？

但這個問題就跟這些問題是一樣的。

> 爲什麼無照駕駛會被罰錢
> 爲什麼學生證能證明你是學生
> 爲什麼畢業證書
> ...

這些證明的背後，都有一個具有公信力的角色存在，所以他們頒發的證明、證書都能夠被信任。那麼，我們自己亂簽的憑證是不具有公信力的。

接下來，我們來看看剛剛所說的一張憑證的訊息通常有哪些欄位並且它是用什麼方式表示的。

#### X.509 憑證格式
----------

有關憑證格式的規範寫在[RFC 5280](https://datatracker.ietf.org/doc/html/rfc5280)。

我們先拿一張實際的憑證來看看。

```
This is a real-world X.509 certificate, specifically a root CA.
https://letsencrypt.org/certificates/
-----BEGIN CERTIFICATE-----
MIIEkjCCA3qgAwIBAgIQCgFBQgAAAVOFc2oLheynCDANBgkqhkiG9w0BAQsFADA/MSQwIgYDVQQKExtEaWdpdGFsIFNpZ25hdHVyZSBUcnVzdCBDby4xFzAVBgNVBAMTDkRTVCBSb290IENBIFgzMB4XDTE2MDMxNzE2NDA0NloXDTIxMDMxNzE2NDA0NlowSjELMAkGA1UEBhMCVVMxFjAUBgNVBAoTDUxldCdzIEVuY3J5cHQxIzAhBgNVBAMTGkxldCdzIEVuY3J5cHQgQXV0aG9yaXR5IFgzMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAnNMM8FrlLke3cl03g7NoYzDq1zUmGSXhvb418XCSL7e4S0EFq6meNQhY7LEqxGiHC6PjdeTm86dicbp5gWAf15Gan/PQeGdxyGkOlZHP/uaZ6WA8SMx+yk13EiSdRxta67nsHjcAHJyse6cF6s5K671B5TaYucv9bTyWaN8jKkKQDIZ0Z8h/pZq4UmEUEz9l6YKHy9v6Dlb2honzhT+Xhq+w3Brvaw2VFn3EK6BlspkENnWAa6xK8xuQSXgvopZPKiAlKQTGdMDQMc2PMTiVFrqoM7hD8bEfwzB/onkxEz0tNvjj/PIzark5McWvxI0NHWQWM6r6hCm21AvA2H3DkwIDAQABo4IBfTCCAXkwEgYDVR0TAQH/BAgwBgEB/wIBADAOBgNVHQ8BAf8EBAMCAYYwfwYIKwYBBQUHAQEEczBxMDIGCCsGAQUFBzABhiZodHRwOi8vaXNyZy50cnVzdGlkLm9jc3AuaWRlbnRydXN0LmNvbTA7BggrBgEFBQcwAoYvaHR0cDovL2FwcHMuaWRlbnRydXN0LmNvbS9yb290cy9kc3Ryb290Y2F4My5wN2MwHwYDVR0jBBgwFoAUxKexpHsscfrb4UuQdf/EFWCFiRAwVAYDVR0gBE0wSzAIBgZngQwBAgEwPwYLKwYBBAGC3xMBAQEwMDAuBggrBgEFBQcCARYiaHR0cDovL2Nwcy5yb290LXgxLmxldHNlbmNyeXB0Lm9yZzA8BgNVHR8ENTAzMDGgL6AthitodHRwOi8vY3JsLmlkZW50cnVzdC5jb20vRFNUUk9PVENBWDNDUkwuY3JsMB0GA1UdDgQWBBSoSmpjBH3duubRObemRWXv86jsoTANBgkqhkiG9w0BAQsFAAOCAQEA3TPXEfNjWDjdGBX7CVW+dla5cEilaUcne8IkCJLxWh9KEik3JHRRHGJouM2VcGfl96S8TihRzZvoroed6ti6WqEBmtzw3Wodatg+VyOeph4EYpr/1wXKtx8/wApIvJSwtmVi4MFU5aMqrSDE6ea73Mj2tcMyo5jMd6jmeWUHK8so/joWUoHOUgwuX4Po1QYz+3dszkDqMp4fklxBwXRsW10KXzPMTZ+sOPAveyxindmjkW8lGy+QsRlGPfZ+G6Z6h7mjem0Y+iWlkYcV4PIWL1iwBi8saCbGS5jN2p8M+X+Q7UNKEkROb3N6KOqkqm57TH2H3eDJAkSnh6/DNFu0Qg==
-----END CERTIFICATE-----
```

一開始看到應該會覺得這三小，怎麼一堆亂碼。

其實仔細看它是有格式的，我們可以注意到上面有些地方我們還是看得懂的。

像是`-----BEGIN CERTIFICATE-----`跟`-----END CERTIFICATE-----`。

這個是一種叫做PEM的格式。

##### PEM 格式
----------

PEM全名是Privacy Enhanced Mail，這種格式滿好理解的。

就是一個`BEGIN`，然後夾東西，最後放個`END`。

就像上面的憑證一樣，根據維基百科，關於PEM格式的定義在RFC 1421 - 1424。

> The PEM format was first developed in the privacy-enhanced mail series of [RFCs](https://en.wikipedia.org/wiki/Request_for_Comments "Request for Comments"): RFC 1421, RFC 1422, RFC 1423, and RFC 1424.

當初會有這個格式是因爲如其名，隱私強化郵件，也就是把內容亂碼化，讓別人沒辦法直接懂，需要經過一次解碼才能看懂，但又需要分隔符號來告知說亂碼化的區塊是什麼，這個格式就因此誕生。

那麼，這個亂碼化也只是做一次base64的編碼，所以裏面的那陀東西就是用Base64進行編碼的產生的。

##### Base64 編解碼
----------

我們常常聽到Base64，它就是由英文字母`a-z`, `A-Z`, `0-9`和 `/`, `+` 共64種在ASCII內可正常顯示的字元來進行編碼（所以裏面的bit代表的也是ASCII上實際對應的編碼），最後再加上`=`用來填充，然後編碼的結果是字串。

那這邊可能有問題是說，爲什麼需要填充？

首先，我們先看看它的原理。

Base64是每6個bit爲一單位進行編碼。

![sample](https://i.imgur.com/mVnHAZO.png)

可是一個byte是8個bit，所以如果沒辦法被8整除的話，就必須做填充，那麼填充就是添加`=`來達成。

![equal_sample](https://i.imgur.com/tbQyKiA.png)

看到這邊可能會產生一些問題。

像是這樣換算下來，它不是讓資料變得更長了嗎？不會很不方便嗎？

還有科技不是一直在進步，爲什麼現在還在用base64？

先回答第一個問題，這個格式本身的目的就是爲了以前郵件傳輸附件，而附件本身又是binary，沒辦法直接傳送，所以才挑選ASCII上的常用可視字元來做轉換。

它當然把資料變更長了，但也因此能夠傳送了，這就是所謂的有得必有失。

回答第二個問題之前，我們首先要知道ASCII其實以前是用7個bit去做編碼，現在會多一位是爲了檢查。

我們可以參考[BBC的定義](https://www.bbc.co.uk/bitesize/guides/zsnbr82/revision/5)：

> ASCII uses **8 bits** to represent a character.
> However, one of the bits is a **parity bit**. This is used to perform a parity check (a form of error checking). This uses up one bit, so ASCII represents 128 characters (the equivalent of 7 bits) with 8 bits rather than 256.

那麼，以前大多數的系統是以8個bit爲一單位，所以會在傳送的時候有機會被擦除多餘的bit，因而導致訊息傳遞有誤。

所以，Base64的做法才誕生，這樣我們就可以對任意數量的bit進行編碼。

也因此，我們現在仰賴base64來進行安全的傳送文本資料。

於是我們傳遞文本會有以下步驟：

1. 文本資料先經過 UTF-8 編碼或其他編碼成bytes
2. bytes的資料再經過base64的安全轉換成base64編碼的文本資料
3. base64編碼的文本資料再轉換成ASCII編碼的bytes資料進行傳送

那麼，接收方只要反向操作，就能夠把接收到的bytes資料轉回最一開始的文本資料了。

從這個方法來看，我們其實除了郵件的附件外，任何binary資料都可以使用base64編碼再安全的轉成ASCII的bytes進行傳送，像是圖片也可以。

這也就是爲什麼現在還在用base64的原因。

更詳細的答案可以參考[這邊](https://stackoverflow.com/questions/3538021/why-do-we-use-base64)。


- Base64 URL

另外，base64有一個派生叫base64 URL，它們只差在兩個不同的字在轉換上不一樣。

![base64_url_compare](https://i.imgur.com/IiGJnLK.png)

這兩個被替代的符號分別是：

- \+ -> -
	這是因爲網站對路徑解析的時候，加號會被當成參數，所以得換掉。

- / -> \_
	這是因爲網站解析斜線會被當成子路徑，所以也得換掉。

那麼，這個編碼方式通常被用來網路服務直接存取base64 URL文字再解碼成binary的做法。

在base64這邊最後，有個問題，就是爲什麼一定要64？

這沒什麼好說的，就單純因爲它們當初就定義這64個字元。

其實還有其他版本的編碼，像是base56，這個編碼就把一些容易混淆的字元換掉，像是`0`跟大寫的英文字母`o`，或是`1`跟大寫的英文字母`i`等等。

整個base56的集合就只剩以下這些字元：

```
23456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnpqrstuvwxyz
```

接着，我們把憑證這串base64解碼之後，會得到什麼？

##### DER 編碼
----------

我們使用base64解碼還原後，會得到以下這樣的binary資料：

```
30 82 04 92 30 82 03 7A A0 03 02 01 02 02 10 0A 01 41 42 00 00 01 53 85 73 6A 0B 85 EC A7 08 30 0D 06 09 2A 86 48 86 F7 0D 01 01 0B 05 00 30 3F 31 24 30 22 06 03 55 04 0A 13 1B 44 69 67 69 74 61 6C 20 53 69 67 6E 61 74 75 72 65 20 54 72 75 73 74 20 43 6F 2E 31 17 30 15 06 03 55 04 03 13 0E 44 53 54 20 52 6F 6F 74 20 43 41 20 58 33 30 1E 17 0D 31 36 30 33 31 37 31 36 34 30 34 36 5A 17 0D 32 31 30 33 31 37 31 36 34 30 34 36 5A 30 4A 31 0B 30 09 06 03 55 04 06 13 02 55 53 31 16 30 14 06 03 55 04 0A 13 0D 4C 65 74 27 73 20 45 6E 63 72 79 70 74 31 23 30 21 06 03 55 04 03 13 1A 4C 65 74 27 73 20 45 6E 63 72 79 70 74 20 41 75 74 68 6F 72 69 74 79 20 58 33 30 82 01 22 30 0D 06 09 2A 86 48 86 F7 0D 01 01 01 05 00 03 82 01 0F 00 30 82 01 0A 02 82 01 01 00 9C D3 0C F0 5A E5 2E 47 B7 72 5D 37 83 B3 68 63 30 EA D7 35 26 19 25 E1 BD BE 35 F1 70 92 2F B7 B8 4B 41 05 AB A9 9E 35 08 58 EC B1 2A C4 68 87 0B A3 E3 75 E4 E6 F3 A7 62 71 BA 79 81 60 1F D7 91 9A 9F F3 … skipping 160 bytes … FC F2 33 6A B9 39 31 C5 AF C4 8D 0D 1D 64 16 33 AA FA 84 29 B6 D4 0B C0 D8 7D C3 93 02 03 01 00 01 A3 82 01 7D 30 82 01 79 30 12 06 03 55 1D 13 01 01 FF 04 08 30 06 01 01 FF 02 01 00 30 0E 06 03 55 1D 0F 01 01 FF 04 04 03 02 01 86 30 7F 06 08 2B 06 01 05 05 07 01 01 04 73 30 71 30 32 06 08 2B 06 01 05 05 07 30 01 86 26 68 74 74 70 3A 2F 2F 69 73 72 67 2E 74 72 75 73 74 69 64 2E 6F 63 73 70 2E 69 64 65 6E 74 72 75 73 74 2E 63 6F 6D 30 3B 06 08 2B 06 01 05 05 07 30 02 86 2F 68 74 74 70 3A 2F 2F 61 70 70 73 2E 69 64 65 6E 74 72 75 73 74 2E 63 6F 6D 2F 72 6F 6F 74 73 2F 64 73 74 72 6F 6F 74 63 61 78 33 2E 70 37 63 30 1F 06 03 55 1D 23 04 18 30 16 80 14 C4 A7 B1 A4 7B 2C 71 FA DB E1 4B 90 75 FF C4 15 60 85 89 10 30 54 06 03 55 1D 20 04 4D 30 4B 30 08 06 06 67 81 0C 01 02 01 30 3F 06 0B 2B 06 01 04 01 82 DF 13 01 01 01 30 30 30 2E 06 08 2B 06 01 05 05 07 02 01 16 22 68 74 74 70 3A 2F 2F 63 70 73 2E 72 6F 6F 74 2D 78 31 2E 6C 65 74 73 65 6E 63 72 79 70 74 2E 6F 72 67 30 3C 06 03 55 1D 1F 04 35 30 33 30 31 A0 2F A0 2D 86 2B 68 74 74 70 3A 2F 2F 63 72 6C 2E 69 64 65 6E 74 72 75 73 74 2E 63 6F 6D 2F 44 53 54 52 4F 4F 54 43 41 58 33 43 52 4C 2E 63 72 6C 30 1D 06 03 55 1D 0E 04 16 04 14 A8 4A 6A 63 04 7D DD BA E6 D1 39 B7 A6 45 65 EF F3 A8 EC A1 30 0D 06 09 2A 86 48 86 F7 0D 01 01 0B 05 00 03 82 01 01 00 DD 33 D7 11 F3 63 58 38 DD 18 15 FB 09 55 BE 76 56 B9 70 48 A5 69 47 27 7B C2 24 08 92 F1 5A 1F 4A 12 29 37 24 74 51 1C 62 68 B8 CD 95 70 67 E5 F7 A4 BC 4E 28 51 CD 9B E8 AE 87 9D EA D8 BA 5A A1 01 9A DC F0 DD 6A 1D 6A D8 … skipping 160 bytes … 28 EA A4 AA 6E 7B 4C 7D 87 DD E0 C9 02 44 A7 87 AF C3 34 5B B4 42
```

好了，所以我說這又是三小？

一坨十六進位的數字，根本看不懂！

其實它是依循一種叫做DER的編碼格式來排列這些數字。

DER全名是Distinguished Encoding Rule，中文就是『可識別編碼規則』。

而DER又是BER的一種被閹割格式。

等等我們會討論DER跟BER的差異。

我們先看看BER是什麼，BER全名是Basic Encoding Rule。

它是一種T-L-V格式。

等等！T-L-V又是什麼新名詞？

其實就是Type(或是Tag)-Length-Value的簡稱，也就是說如下圖範例：

![T-L-V_format](https://i.imgur.com/S4cwjR3.png)

- Type
	用來表示後面資料的屬性。

- Length
	用來表示後面資料的總長度，也就是後面有幾個byte資料（一個hex資料是4 bits，一個byte是8 bits，所以是兩個hex值）。

- Value
	用來表示後面資料的RAW值。

也就是說，我們有這三樣東西就可以完美的表示一個結構化的資料。

###### Type
----------

接下來，就要看看Type有哪些。

根據[維基百科](https://en.wikipedia.org/wiki/X.690#Types)上面列的表格：

![types](https://i.imgur.com/uNle3pT.png)

最左邊的`Name`是指這個type / tag / label / class的名稱（超多種代稱）。

再來，`Primitive Construction`是指說這個class必須是哪種類型，是primitive？還是constructed？或是我全都要。

這邊會有疑問的是primitive到底是什麼？

![primitive](https://i.imgur.com/EzHgkFf.png)

還是看不懂，爲什麼扯個類別可以提到原始？

那我們看看它相對的是constructed，也就是結構。

從這邊我們可以推論出來，這邊代表的是這個類別能放的東西是**純粹的值**還是可以放**子結構資料**。換句話說，就是陣列裏面能不能有陣列的意思。

最後，Tag number是這個類別的代表編號。

我們比較表格上面的decimal跟hex，可以發現初號機是`EOC`，它是編號`0x00`。

但其實，這編號是有意義的，可以參考[維基百科](https://en.wikipedia.org/wiki/X.690#Encoding)裏面的定義。

裏面有提到這個表格：

![tag_define](https://i.imgur.com/wmYATQs.png)

剛剛我們提到編號，接下來有更細節的編碼。

這邊舉例一個tag就是`0x30`的`sequence`，以二進位表示就是`0011 0000`。

以表格的模式拆解，會得到Octet1，變成`00 1 10000`。

從這邊可以看對上一個表格中，sequence的decimal表示是`16`，也就是二進位的`10000`。

那麼，它是的P/C是Primitive，也就是`1`，所以我們可以得到`110000`。

可是前綴還有`00`，這要怎麼解釋？

維基上面還有另外兩個表格，如下：

![class_name](https://i.imgur.com/Gdm97rH.png)

這邊第一個表格定義了我們Tag Class，也就是Tag的類型，有點饒舌，但就是Tag本身還有被分類的意思。

那麼`0`就是所謂的ASN.1原生類別，寫成16進位就是`0x00`。

ASN.1我們等等會提到。

第二個表格則是再解釋了一次P/C的意思。

這樣線索都找齊了，我們可以拼出`sequence`的真正編碼`00 1 10000`。

可是剛剛說到的Octet 1後面還有一個Octet 2 onwards是什麼意思？

這時就要提到有一個東西叫Long form，也就是萬一我們與時俱進，多了很多tag，導致編號超過5個bit長度的時候，就會用到了。

那麼，我們今天編號真的超過的話，Tag Number就會全是1，也就是Octet 1的1-5位bit全都是1。

然後Octet 2的位子8就是1，後面1-7位就是實際編號，這邊要注意的是這1-7位編號不能全是0，也就是一定要有東西的意思。

也因爲Octet 2的關係，這樣我們能表示的編號就從Short form五位（最多只能到編號31）擴充到Long form七位（最多可以編號到 127）了。

###### Length
----------

剛剛說完Tag是什麼，我們來要看看Length。

根據維基百科提到這個表格：

![length_form](https://i.imgur.com/NBJmjNV.png)

我們由一開始的範例可以看到一般長度也是由兩個hex值表示，但其實是1+7個bit表示。

按表格，第一位也就是編號8的位子跟剛剛tag欄位一樣是表示Long form還是short form，其他1-7位是表示實際長度。

那麼，如果編號8是0，後面的數字代表的就是short form的長度，7個bit最多可以表示到`0000000-1111111`，也就是`0-127`。例如`0x05`就是後面資料有五個byte的長度。

如果編號8是1，後面1-7位都是0的話，就代表長度是不定的。

那如果編號8的位置是1，那就有三種可能。

1. Bits位置1-7全部都是0，那就是我的資料是不定長度，在最後資料結尾會有`00 00`來做分隔符號。
	例如：`30 80 02 01 05 00 00`，這串資料就是一個sequence的結構資料，裏面有一個不定長度的子結構，那這個子結構的type是`0x02`，我們查表可以看到是`Integer`，然後這個長度是`0x01`，也就是一個byte，資料爲`0x05`，結尾補上分隔符號`0x0000`。
2. Bits位置1-7如果是`0000001`-`1111110`，也就是decimal的`1-126`的話，就代表後面會有一個有限長度的資料，但長度太長，我沒辦法用一個bytes表示，所以跟剛剛的tag class一樣需要擴充，這個表示方式也稱爲Long form。
	這邊維基百科有個例子。
	![long_form_example](https://i.imgur.com/dTJmCc7.png)
	
	Octet 1是length，它的長度是`1 0000010`，最前面的`1`代表它是long form，長度需要用七個7bits來表示，寫成16進位，也就是`0x82`。那麼，後面七個bit是`0000010`，代表著它後面必定有兩個bytes來表示它的長度。
	Octet 2跟Octet 3就是真正的資料長度`00000001 10110011`，用decimal表示就是`435`，也就是說後面必定接**435個byte**長度的資料。
	
	這邊有個問題是說，萬一資料長度還是超過`126`要怎麼辦？
	剛剛有說到不定長度的做法，就是`0x80`，然後後面接`0x0000`。

	不過，也可以想想看，如果我長度是最大值的126的話，代表我可以用126個byte表示資料長度，那麼126 bytes最多可以表示2^126-1個byte，也就代表我後面接的資料有2^126-1個byte這麼大。我們用計算機算一下，可以得到這個數字。
	
	![2^126-1](https://i.imgur.com/LcBVNpQ.png)
	
	後面的除以1024^3是爲了把byte換算成TB，也就是說我們換算成TB還有10^28這麼多，以目前看來一般資料是不太可能超越這個數字。
	
3. Bits位置1-7是`1111111`的話，整個來看是`1111 1111`，目前是一個保留的位子。

###### Value
----------

根據剛剛的Tag，我們知道這value是什麼類別，再根據Length知道這value的長度有多長。

那麼，value本身就有可能是各種類型的資料（反正都是binary），當然也有可能是一個新的結構，但這個結構也必須要遵從T-L-V格式去建立。

到這邊，還記得我們一開始的那個範例嗎？

```
30 13 02 01 05 16 0e 41 6e 79 62 6f 64 79 20 74 68 65 72 65 3f
```

查表我們可以知道`0x02`是Integer，`0x16`是IA5String，也就是一種String。

我們就可以解析一下這個範例得到這樣的架構。

![sample](https://i.imgur.com/3mhc3Hk.png)

其中，IA5String Tag後面的value我們可以根據ASCII解碼得到一個字串`Anybody there?`。

我們看完BER的T-L-V格式了，還記得我們這邊的標題其實是DER嗎？

###### BER與DER的差異
----------

剛剛有提到DER是閹割版的BER，那麼DER到底是什麼？

它跟BER相比有四個顯著的特性，分別是：

1. 長度一定要寫出來，而且越短越好。也就是說，它不能是`0x80`的不定長度。
2. Bitstring, octetstring跟其他受限的character strings，必須要是Primitive。
3. 如果它的Type是`SET`的話，內容必須按照它的Type編號進行排序。
4. DER大多用在密碼學相關應用（像是這邊說的憑證）上，所以必須能夠被唯一表示。對於BER的Boolean來說，True的表示就是除了`0x00`外的`0x01-0xFF`。但這在DER不被允許，反而True只能是`0xFF`。

所以有些人會說DER是BER的一種子集，這樣說其實也沒錯，它們的差異就如同上面所說。

這邊會有個問題是，BER既然可以解DER編碼，那爲什麼要特別再搞出一個DER？

我在上面其實也有提到了，DER大多是用在密碼學相關應用，因此才把BER閹割去定義一個更嚴格的格式。而且，談到安全的東西總不能馬虎，固定長度也是一個很重要的特性，總不可能我跟你說我要傳金鑰給你，然後不給你長度吧！

這一節的最後，總會有一種問題，就是這些原理啥的我都知道了，可是爲什麼要用DER？

有一個答案是因爲RFC裏面定義憑證必須使用DER編碼。

這樣可能會再衍生一個新的問題，那DER是怎麼來的？

我們就可以想想看另外一個問題，以前人怎麼傳資料的？

以前電腦通訊沒這麼發達，如果我在傳資料給你的時候，先跟你說我要傳什麼類型的資料，然後我資料有多長，這樣你收資料的時候就可以知道什麼時候才能把整個資料收完。

這不就是T-L-V格式的做法嗎？

也就是因爲通訊的需要，經過各種規範，最後迭代出BER這種做法，那DER只是嚴謹一點的BER。

##### ASN.1
----------

我們在上面討論過DER編碼，也用了範例來瞭解。

那麼，舊圖重用。

![sample](https://i.imgur.com/3mhc3Hk.png)

我可以直接跟你說，這就是ASN.1。

我們可以看看[維基百科](https://zh.wikipedia.org/zh-tw/ASN.1)怎麼說明ASN.1：

> 在電信和計算機網絡領域，ASN.1（Abstract Syntax Notation One) 是一套標準，是描述數據的表示、編碼、傳輸、解碼的靈活的記法。它提供了一套正式、無歧義和精確的規則以描述獨立於特定計算機硬體的對象結構。

維基總是講得很抽象，我們可以用一句話解決它，那就是：

> 一種抽象定義資料結構的方法。

我們寫程式宣告變數的時候，最重要的就是需要知道我們這個變數是什麼型別。這也可以對比到ASN.1上面的資料類別。

![ASN.1_sample](https://i.imgur.com/edD0CJ3.png)

根據這個範例，我們可以看出來，ASN.1就純粹只是資料結構的定義。

那麼，真正的實作呢？就交給BER、DER這些格式吧！

實作的時候，以我們的範例來說，它整個是一個`SEQUENCE`，值就是裏面的兩個元素，分別是`INTEGER`跟`IA5String`，值分別是`5`跟`Anybody there?`。

##### 憑證解析
----------

有了前面的背景知識後，我們對那些binary資料已經不陌生了。

終於可以開始解析這張憑證了。

這邊推薦一個實用的網站，可以拿來線上解析ASN.1，叫做[ASN.1
 JS](https://lapo.it/asn1js)，這邊用的憑證範例就是從上面拉過來的。

我們可以把上面一開始看到的憑證整個貼過去解碼，就可以得到以ASN.1的格式表示的憑證。

![cert](https://i.imgur.com/9mtsLQs.png)

###### Object Identifier
----------

我們看圖片，如果只看Type會發現，有個東西之前沒看過，那就是OBJECT IDENTIFIER（OID）。

OID就是由國際組織定義的唯一識別碼，[維基百科](https://en.wikipedia.org/wiki/Object_identifier)有寫編碼規則。

那爲什麼要有這個唯一識別碼呢？

我們知道電腦的世界都是二進位，所以文字也得二進位表示，那最方便的就是數字。我們傳送前事先定義好這些數字代表什麼再傳送，接收方就可以按照定義去解析這些數字，也就是唯一識別碼。

###### 憑證欄位
----------
  
我們先看來看看一張健全的X.509 v3憑證裏面有什麼東西，再對應到剛剛範例中。

- version
	憑證的版本。
- serialNumber
	憑證的序號，可以是任意的值，只要是唯一即可。
- signature
	CA對這張憑證簽章的時候是使用什麼演算法。
	- algorithm
		簽章使用的演算法。
	- parameters
		簽章演算法需要的參數。
- issuer
	發起這個憑證的人或組織是誰，通常會寫組織名字（organizationName）跟一般稱呼（commonName）。
- validity
	一張憑證的有效期限。
	- notBefore
		在這之前無效。
	- notAfter
		在這之後無效。
- subject
	這張憑證是要頒給誰的，通常會寫國家（countryName）、組織名稱（organizationName）及一般稱呼（commonName）。
- subjectPublicKeyInfo
	申請人的公鑰資訊。
	- algorithm
		公鑰是適用於哪種非對稱加密演算法的。
	- subjectPublicKey
		公鑰是什麼。
- issuerUniqueID（Optional）
	憑證發起者的唯一ID。
- subjectUniqueID（Optional）
	申請人的唯一ID。
- extensions
	擴充欄位。
- signatureAlgorithm
	對憑證簽章使用的演算法。
- signatureValue
	簽章值，也就是CA對這張憑證簽出來的值。

看到上面的列表會發現，也太複雜，根本看不懂這些欄位要幹嘛！

我們接下來一一稍微詳細介紹一下欄位的用途，然後範例的code block爲了避免眼花，所以在markdown借用js渲染。

- version
	憑證的版本，總共有3版，以十六進位表示：`0x0`、`0x1`、`0x2`。
	`INTEGER 2`
- serialNumber
	憑證的序號，可以是任意的值，只要是唯一即可。RFC內定義最長只能有20 bytes。
	`INTEGER (124 bit) 13298795840390663119752826058995181320`
- signature
	CA對這張憑證簽章的時候是使用什麼演算法。
	- algorithm
		簽章使用的演算法。
	`OBJECT IDENTIFIER 1.2.840.113549.1.1.11 sha256WithRSAEncryption (PKCS #1)`
	- parameters
		簽章演算法需要的參數。大多時候都是`null`，以binary表示就是`05 00`。

- issuer
	發起這個憑證的人或組織是誰，通常會寫組織名字（organizationName）跟一般稱呼（commonName）。

	```js
	SEQUENCE (2 elem)
	  SET (1 elem)
		SEQUENCE (2 elem)
		  OBJECT IDENTIFIER 2.5.4.10 organizationName (X.520 DN component)
		  PrintableString Digital Signature Trust Co.
	  SET (1 elem)
		SEQUENCE (2 elem)
		  OBJECT IDENTIFIER 2.5.4.3 commonName (X.520 DN component)
		  PrintableString DST Root CA X3
	```

- validity
	一張憑證的有效期限。
	- notBefore
		在這之前無效。
	- notAfter
		在這之後無效。
	
	白話就是在這期間有效。

	我覺得這邊比較奇葩的就是這個，爲什麼不寫After跟Before就好，硬要在前面加個not，感覺就很有事。

	```js
	SEQUENCE (2 elem)
	  UTCTime 2016-03-17 16:40:46 UTC
	  UTCTime 2021-03-17 16:40:46 UTC
	```

	這邊範例要注意的是，上面明明就是寫2016-03-17之類的這種時間，爲什麼是UTCTime？這邊沒爲什麼，只是因爲parse成這樣比較好看。
	實際上還是一坨binary，但要注意的是UTCTime這個格式是有坑的，而且內容是要用ASCII解碼。
	
	根據[這邊](https://www.obj-sys.com/asn1tutorial/node15.html)說的：
	
	> UTCTime values take the form of either "YYMMDDhhmm\[ss\]Z" or "YYMMDDhhmm\[ss\](+|-)hhmm". The first form indicates (by the literal letter "Z") UTC time. The second form indicates a time that differs from UTC by plus or minus the hours and minutes represented by the final "hhmm".
	
	它其實只有YY，並不是YYYY。那如果我們要YYYY，就必須使用GeneralizedTime，[這邊](https://www.obj-sys.com/asn1tutorial/node14.html)有介紹：
	
	 > Type GeneralizedTime takes values of the year, month, day, hour, time, minute,second, and second fraction in any of three forms.
	> 1.  Local time only. ``YYYYMMDDHH\[MM\[SS\[.fff\]\]\]'', where the optional fff is accurate to three decimal places.
	> 2.  Universal time (UTC time) only. ``YYYYMMDDHH\[MM\[SS\[.fff\]\]\]Z''.
	> 3.  Difference between local and UTC times. ``YYYYMMDDHH\[MM\[SS\[.fff\]\]\]+-HHMM''.
	
	那有個問題是，爲什麼要分成兩種格式？
	
	這是因爲一開始設計的人沒想過人類竟然會活過西元2000年。（並沒有
	其實是因爲以前電腦記憶體很珍貴，要盡量減少資源消耗，所以才把西元年縮減成兩位，也就是UTCTime（從1950開始）。
	但是，後來過2000年發現這東西遲早會遇到Bug，就是萬一時間到2051年就會被表示成1951年，所以才出現GeneralizedTime。
	
	所以，RFC裏面也規定憑證在2050之後MUST全面使用GeneralizedTime。
		
- subject
	這張憑證是要頒給誰的，通常會寫國家（countryName）、組織名稱（organizationName）及一般稱呼（commonName）。
	```js
	SEQUENCE (3 elem)
	  SET (1 elem)
	    SEQUENCE (2 elem)
	      OBJECT IDENTIFIER 2.5.4.6 countryName (X.520 DN component)
	      PrintableString US
	  SET (1 elem)
	    SEQUENCE (2 elem)
	      OBJECT IDENTIFIER 2.5.4.10 organizationName (X.520 DN component)
	      PrintableString Let's Encrypt
	  SET (1 elem)
	    SEQUENCE (2 elem)
	      OBJECT IDENTIFIER 2.5.4.3 commonName (X.520 DN component)
	      PrintableString Let's Encrypt Authority X3	
	```

- subjectPublicKeyInfo
	申請人的公鑰資訊。
	- algorithm
		公鑰是適用於哪種非對稱加密演算法的。
	- subjectPublicKey
		公鑰是什麼。

	```js
	SEQUENCE (2 elem)
	  SEQUENCE (2 elem)
	    OBJECT IDENTIFIER 1.2.840.113549.1.1.1 rsaEncryption (PKCS #1)
	    NULL
	  BIT STRING (2160 bit) 0011000010000010000000…
	    SEQUENCE (2 elem)
	      INTEGER (2048 bit) 197972484760754376823558522464922272…
	      INTEGER 65537
	```

	這個範例我們可以看到它適用於RSA，然後底下的金鑰有2048 bit，也就是RSA-2048。另外，我們可以發現65537，就是我們上次說的RSA中公鑰(N, e)裏面`e`。
	
	問題是爲什麼是65537？
	這邊我就不詳細說明了，有興趣的可以參考[這邊](https://crypto.stackexchange.com/questions/3110/impacts-of-not-using-rsa-exponent-of-65537/3113)。
	
- issuerUniqueID（Optional）
	憑證發起者的唯一ID。

	RFC裏面有提到
	>    If present, version MUST be v2 or v3.

	這是選填欄位，然後這張憑證沒有這個欄位，

	而且這其實是版本2的憑證才有的東西，v3是爲了兼容才放上去定義。
	當初v1沒有這個欄位，所以會出現issuer或是subject完全重複的問題，才更新憑證版本去添加這個欄位。

	那什麼是重複呢？我們先看看這個情境：
		
	> 假設今天有一間公司叫A，它向某CA申請憑證，但A最後無情倒閉了。後來又有一間新公司也叫A，也向某CA申請憑證。

	這個問題會變成雖然CA可以按照憑證編號去判斷，但也只能判斷是發出的不同張憑證，卻無法判斷故事中A到底是舊的還是新的，也就是因爲這樣才需要唯一識別的方式。


- subjectUniqueID（Optional）
	申請人的唯一ID。

	RFC裏面有提到
	>    If present, version MUST be v2 or v3.

	這也是選填欄位，然而這張憑證也沒有這個欄位，

	至於爲什麼有這個欄位，在剛剛有提到過。

- extensions
	擴充欄位。

	RFC裏面有提到
	>    If present, version MUST be v3.

	當初v2是因爲要新增兩個欄位才改版，但後來發現只要新增一個擴充欄位，裏面添加再多新欄位都不用改版了，因此才出現v3。

	這個擴充欄位裏面的額外添加物可多了，這邊就不一一介紹，我們只看範例憑證裏面有的東西。
	
	- basicConstraints
		基本限制，裏面有兩個欄位，分別是：
		1. 這張憑證的Subject是否爲CA，還是EE，預設是false，也就是不是CA。
		2. (Optional) 這張憑證底下的憑證鏈允許有多少CA，也就是憑證鏈的深度，如果寫1的話，代表著底下可以有無限多個CA；如果是EE憑證的話，因爲是終端，可以不用寫。
				
		詳細的解說可以看[這邊](https://www.pkisolutions.com/basic-constraints-certificate-extension/)。
		
		```js
		SEQUENCE (3 elem)
		  OBJECT IDENTIFIER 2.5.29.19 basicConstraints (X.509 extension)
		  BOOLEAN true
		  OCTET STRING (8 byte) 30060101FF020100
			SEQUENCE (2 elem)
			  BOOLEAN true
			  INTEGER 0
		```
	
		這個範例我這邊有點疑問，就是裏面有兩個True。
		第一個True是確認是否Subject爲CA，但第二個True呢？
		而且第二個欄位爲什麼是SEQUENCE，不是INTEGER嗎？
	
		爲了確認這個問題，我去搜尋沒發現有人有這個疑問，然後我撈了其他憑證來對照發現有些明明不是發給CA的憑證，外面的BOOLEAN竟然是true。還有些外面是true，然後SEQUENCE是`30 00`，也就是長度0的SEQUENCE。這也代表它有定義basicConstraints這個欄位，但想用預設值。
		
		目前這個答案是我猜測的，我猜它外面的true應該是想表示它有這個欄位，然後欄位內容接下來以`OCTET STRING`表示。
		
		如果有人有找到或是知道精確的答案，再麻煩留言告訴我，謝謝！
		
	
	- keyUsage
		定義金鑰用途。
		欄位除了定義OID之外還有一個BIT STRING欄位用來表示這張憑證的公鑰能夠做的事情。
		```js
		id-ce-keyUsage OBJECT IDENTIFIER ::= { id-ce 15 }
		KeyUsage ::= BIT STRING {
			digitalSignature (0),
			nonRepudiation (1), -- recent editions of X.509 have -- renamed this bit to contentCommitment
			keyEncipherment (2),
			dataEncipherment (3),
			keyAgreement (4),
			keyCertSign (5),
			cRLSign (6),
			encipherOnly (7),
			decipherOnly (8)
		}
		```
				
		在[這邊](https://ldapwiki.com/wiki/KeyUsage)有更詳細的說明。
	
		```js
		SEQUENCE (3 elem)
		  OBJECT IDENTIFIER 2.5.29.15 keyUsage (X.509 extension)
		  BOOLEAN true
		  OCTET STRING (4 byte) 03020186
		    BIT STRING (7 bit) 1000011
		```
		  
		 這邊範例也有一樣的`BOOLEAN true`，跟上一項一樣我也不知道爲什麼，但推測可能是想表示它這項是有內容的。
		 		 		  
	- authorityInfoAccess
		可向頒發此憑證的CA取得的額外資訊。
		
		```js
		SEQUENCE (2 elem)
		  OBJECT IDENTIFIER 1.3.6.1.5.5.7.1.1 authorityInfoAccess (PKIX private extension)
		  OCTET STRING (115 byte) 3071303206082B060105050730018626…
			SEQUENCE (2 elem)
			  SEQUENCE (2 elem)
				OBJECT IDENTIFIER 1.3.6.1.5.5.7.48.1 ocsp (PKIX)
				[6] (38 byte) http://isrg.trustid.ocsp.identrust.com
			  SEQUENCE (2 elem)
				OBJECT IDENTIFIER 1.3.6.1.5.5.7.48.2 caIssuers (PKIX subject/authority info access descriptor)
				[6] (47 byte) http://apps.identrust.com/roots/dstrootcax3.p7c
		```
			
		從範例來看，我們可以看到這個CA提供了兩項服務，OSCP跟caIssuers。
		
		OCSP就是一種線上詢問憑證狀態的方法，比起CRL清單來得不受限更新時間，但會對被詢問方造成負擔（當然，狂問就等於DDoS了）。[維基百科](https://zh.wikipedia.org/wiki/%E5%9C%A8%E7%BA%BF%E8%AF%81%E4%B9%A6%E7%8A%B6%E6%80%81%E5%8D%8F%E8%AE%AE)也有頁面說明。然後，我們看範例就可以知道它提供的OCSP查詢位置是`http://isrg.trustid.ocsp.identrust.com`。
		
						
	- authorityKeyIdentifier(AKI)
		取自issuer CA的Public key經由SHA-1 Hash value做為Key的identifier。
		這個做法比起版本2的Issuer UID來說，還多了一項資訊，就是我目前這張憑證究竟來自哪張憑證，而不是只有唯一的一個Issuer UID。
		對應方法就是AKI會等於上一層憑證的subjectKeyIdentifier(SKI)。
	
		```js
		SEQUENCE (2 elem)
		  OBJECT IDENTIFIER 2.5.29.35 authorityKeyIdentifier (X.509 extension)
		  OCTET STRING (24 byte) 30168014C4A7B1A47B2C71FADBE14B9075FFC41560858910
		```
		
	- subjectKeyIdentifier(SKI)
		取自Subject的Public key經由經由SHA-1 Hash value做為Key的identifier。
		SKI如剛剛所說的，如果有下一層憑證的話，會變成下一層憑證的AKI。以此形成一個一層層憑證鏈的對應關係。

		```js
		SEQUENCE (2 elem)
		  OBJECT IDENTIFIER 2.5.29.14 subjectKeyIdentifier (X.509 extension)
		  OCTET STRING (22 byte) 0414A84A6A63047DDDBAE6D139B7A64565EFF3A8ECA1
		```
		
		這邊有一張範例圖，看了可能會比較瞭解：
		
		![aki_ski](https://i.imgur.com/mQjOBte.png)
		
			
	- certificatePolicies
		當初CA基於怎樣的規範發這張憑證，[這邊](https://www.ibm.com/docs/en/zos/2.2.0?topic=customization-using-certificate-policies)有提到：
		
		> This extension contains policy information, such as how your CA operates and the intended purpose of the issued certificates.
				
		```js
		SEQUENCE (2 elem)
		  OBJECT IDENTIFIER 2.5.29.32 certificatePolicies (X.509 extension)
		  OCTET STRING (77 byte) 304B3008060667810C010201303F0…
		    SEQUENCE (2 elem)
			  SEQUENCE (1 elem)
			    OBJECT IDENTIFIER 2.23.140.1.2.1
			  SEQUENCE (2 elem)
			    OBJECT IDENTIFIER 1.3.6.1.4.1.44947.1.1.1
			    SEQUENCE (1 elem)
				  SEQUENCE (2 elem)
				    OBJECT IDENTIFIER 1.3.6.1.5.5.7.2.1 cps (PKIX policy qualifier)
				    IA5String http://cps.root-x1.letsencrypt.org
		```
			
		範例如果把它parse成字串的話，會變成：
		
		```
		[1]Certificate Policy:
		    Policy Identifier=2.23.140.1.2.1
		[2]Certificate Policy:
			Policy Identifier=1.3.6.1.4.1.44947.1.1.1
			[2,1]Policy Qualifier Info:
				Policy Qualifier Id=CPS
				Qualifier:
					http://cps.letsencrypt.org
		```
			
		這邊我們可以看出有兩個OID：
		
		1. `2.23.140.1.2.1`
			這個是表示DN（domain name），我們可以在[OID info](http://oid-info.com/get/2.23.140.1.2.1)上面查到登記這個OID的相關資料。
		2. `1.3.6.1.4.1.44947.1.1.1`
			這間企業向國際組織申請的唯一編號，又被稱爲PEN（Private Enterprise Number）。
				
		然後，`Policy Qualifier Info`裏面指的CPS就是『憑證實務作業基準』。CA會根據這個基準去管理憑證，也算是一種憑證頒發者聲明。
	
		詳細可以參考這邊[Certificate Policies extension – all you should know Part1](https://www.sysadmins.lv/blog-en/certificate-policies-extension-all-you-should-know-part-1.aspx)跟[Certificate Policies extension – all you should know Part2](https://www.sysadmins.lv/blog-en/certificate-policies-extension-all-you-should-know-part-2.aspx)。
			
			
	- cRLDistributionPoints
		如其名，這個欄位表示CRL的發佈位置。
		
		```js
		SEQUENCE (2 elem)
		  OBJECT IDENTIFIER 2.5.29.31 cRLDistributionPoints (X.509 extension)
			OCTET STRING (53 byte) 30333031A02FA02D862B687474703A2F2F637…
			SEQUENCE (1 elem)
			  SEQUENCE (1 elem)
				[0] (1 elem)
				  [0] (1 elem)
					[6] (43 byte) http://crl.identrust.com/DSTROOTCAX3CRL.crl
		```

		從範例，我們可以看出來這個CA的CRL發佈位置是`http://crl.identrust.com/DSTROOTCAX3CRL.crl`。

		詳細可以參考[這邊](https://www.ibm.com/docs/en/external-auth-server/2.4.3?topic=extensions-crldistributionpoints-extension)。

- signatureAlgorithm
	對憑證簽章使用的演算法。
	`OBJECT IDENTIFIER 1.2.840.113549.1.1.11 sha256WithRSAEncryption (PKCS #1)`

	等等！剛剛不是裏面出現過一次嗎？
	
	主要是因爲簽章爲了把算法也簽進簽章值裏面，所以才會重複出現，這樣也可以達成雙重驗證的效果。
	
	這邊[Why is the Signature Algorithm listed twice in an x509 Certificate?](https://security.stackexchange.com/questions/114746/why-is-the-signature-algorithm-listed-twice-in-an-x509-certificate)有個答案是說避免一些被攻擊的風險。

- signatureValue
	簽章值，也就是CA對這張憑證簽出來的值。
	`BIT STRING (2048 bit) 110111010011001111010111000100`
	
	等等！再等等！對這張憑證簽？！所以是簽哪些？
	
	我們剛剛說的區塊裏面除了最後的`signatureAlgorithm`跟`signatureValue`之外，其他區塊合稱`tbsCertificate`，其中`tbs`是`to be sign`的縮寫，也就是需要被簽的區塊。

最後，這邊有一張從ASN.1定義到最後PEM格式的憑證流程圖可以看看，引用自[Illustrated X.509 Certificate](https://darutk.medium.com/illustrated-x-509-certificate-84aece2c5c2e)：

![app](https://miro.medium.com/max/2400/1*ShM4dle1AXnR9pvZ1gTKfA.png)

以上就是憑證的欄位解析，如果你都有看完，那我們就看完現役的一張完整憑證了，恭喜你。

## 結論
----------

這一次資訊量很大，上次有提到密碼學，這次應用密碼學中的非對稱加密來做簽驗章，以此驗證不可否認性以及完整性。

再來，因爲無法單從公鑰分辨身份，需要第三方去背書，因此有了憑證。
最後，我們直接剖析一張完整的憑證來看看憑證裏面到底裝了些什麼。

從這邊就可以知道我們習以爲常的網路安全背後有很複雜的原理，無時無刻，每天都在作用。

那麼，學會了這些有什麼用呢？

很遺憾，在一般生活中也沒什麼用。頂多跟朋友聊天的時候可以打打嘴砲說我可以用肉眼解析憑證的二進位格式之類的。

不過在工作上，如果是做資安相關的人或許會有點用吧。

最後，關於簽驗章跟憑證，更詳細的說明可以參考[這邊](http://www.tsnien.idv.tw/Security_WebBook/%E7%AC%AC%E4%B8%83%E7%AB%A0%20%E6%95%B8%E4%BD%8D%E7%B0%BD%E7%AB%A0%E8%88%87%E6%95%B8%E4%BD%8D%E6%86%91%E8%AD%89.html)，然後ASN.1、BER跟DER可以參考[A Layman's Guide to a Subset of ASN.1, BER, and DER](https://luca.ntop.org/Teaching/Appunti/asn1.html)。

## 圖片串
----------

[這次用到的簽驗章截圖imgur存放區](https://imgur.com/a/eseCwy0)
[這次用到的憑證截圖imgur存放區](https://imgur.com/a/GS7K5lO)


## 尾語
----------

這邊有個悲報就是這是這系列最後一集。

因爲我離職了，未來很高機率不會再碰到這些知識。

感謝某公司讓我花時間自學這些以前沒想過的冷門知識，也感謝前輩願意指導。

沒想到會因爲人的關係離開，我一直覺得工作上可以互相尊重是件很重要的事情。

有人喜歡堅持己見，強硬又還一副自以爲對你好的態度，說話只會冷嘲熱諷還預設立場，連溝通都沒辦法。

期許自己未來不會變成這種老害，也希望下一份工作能更好。😥

最後，這次的內容如果有問題也歡迎留言討論。

如果要轉載的話，再麻煩標註作者跟網址，謝謝！ 😊

