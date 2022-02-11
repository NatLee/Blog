---
title: 資安常識(2) - 對稱非對稱加密演算法與雜湊算法
categories: study
tags:
  - cryptograph
  - security
  - encrypt
  - decrypt
abbrlink: 1489926540
date: 2021-09-30 00:00:00
---


## 前言
----------

接續上次的PKI，我們知道有件事是資安裏面一定會遇到的事！

也就是大家的密碼絕對不會用明文呈現。

那麼，我們要如何把一段明文變成密碼呢？

<!--more-->

## 內容
----------

把一段明文變成密碼，我們要做的就是`加密`；相反則是把密文變回明文，也就是`解密`。

另外，這邊要注意的是，有些人聽過`雜湊`，但是加解密跟雜湊是完全不一樣的東西！

一段文字（或資料）經過雜湊是不能變回去原本的資料的，因爲原始內容已經經過不斷壓縮破壞才變成長度一致的雜湊值。其中，怎麼破壞要看每個雜湊函數的定義。

就跟生米煮成熟飯一樣不能把飯變回米粒一樣。

那麼，我們之前常聽到的對稱與非對稱加密演算法屬於現代密碼學的範疇。

在密碼學裏面有分爲古典密碼學跟現在密碼學，我們先分別理解一下差異。


### 古典密碼學
----------

以前的加密手法被稱爲隱寫術，根據[維基百科](https://zh.wikipedia.org/wiki/%E5%8F%A4%E5%85%B8%E5%AF%86%E7%A2%BC)的描述：

> **古典密碼**是密碼學中的其中一個類型，其大部分加密方式都是利用替換式密碼或移項式密碼，有時則是兩者的混合。其於歷史中經常使用，但現代已經很少使用，大部分的已經不再使用了。一般而言，經典密碼是基於一個拼音字母（像是 A-Z）、動手操作或是簡單的裝置。它們可能是一種簡單的密碼法，以致於不可信賴的地步，特別是有新技術被發展出來後。

我們可以瞭解到以前的密碼大多是用移形換位法來達成加密的目的。

加密使用移形換位法的話，我們只要再反向操作就能從密文推回明文了。

#### 凱撒密碼
----------

以維基百科的例子，我們來看看凱撒密碼。

凱撒密碼是一種替換式密碼，我們把明文的字母 a-z 打亂，再做一一對應，就可以達成凱撒加密了。

> **一般字母**: a b c d e f g h i j k l m n o p q r s t u v w x y z
> **密碼字母**: c i p h e r s t u v w x y z a b d f g j k l m n o q

像是這串明文`hello`，我們可以把它換成密文`texxa`。

除此之外，也有其他加密方式，幾乎都大同小異。

這種加密方式，我們很容易能夠透過統計分析密文來解析出明文。

例如，英文裏面最常出現的介系詞、助詞、冠詞都可以拿來當做高頻率單字來提供解析線索。

所以，不想被簡單破解就得使用現代手段。

### 現代密碼學
----------

現代密碼學主要是由電腦的出現，進而改變以前對密碼的認知。

以前人覺得密碼只要讓不知情的人看不懂就好，設計的人會運用自己的創意來製造加密方式。但這樣的方式容易讓有心人士察覺並破解，其實並不安全。

現代密碼強調的是『如何在充滿敵人的環境來傳遞資訊』，也就是說任何人都可以看到我的密文，但他們都沒辦法破譯。

因此，我們如果有一把共同持有的鑰匙去加密的話，你知道我知道，加解密都用同一把鑰匙。這樣加密後傳送，就算有人攔截密文也沒辦法看懂內容，這就是對稱式加密。

但是，如何傳遞這把鑰匙就成了一個安全問題。

後來則衍生出，我有一把公開的鑰匙，大家都可以取得；另一把非公開的鑰匙，只有我擁有。這樣兩把鑰匙，公開的叫公鑰，非公開的叫私鑰，兩把都可以拿來加解密，分別有不同的含義，這就是非對稱式加密的方法。

這時候就有一個問題，這樣講非對稱好像比對稱還厲害？爲什麼現在還在用對稱加密？

其實，非對稱加密有一些弊病存在，像是加密速度因爲數學運算導致相較於對稱加密來得慢，所以現在大多的加密手法都是混合對稱與非對稱來進行加密，也稱爲混合加密。

接下來，我們分別看看一些常見的對稱式與非對稱加密方法，而這些方法都是採用區塊加密。

那麼，區塊加密的意思是明文切成同金鑰長度的一塊塊資料塊對著金鑰進行加密。

其中，區塊加密方式有分成很多種不同的分塊方式，有興趣的可以參見[這邊的解說](https://notes.andywu.tw/2019/%E5%AF%86%E7%A2%BC%E7%9A%84%E5%8A%A0%E5%AF%86%E6%A8%A1%E5%BC%8Fecb-cbc-cfb-ofb-ctr/)。


#### 對稱式加密
----------

區塊對稱加密的算法有很多種，這邊提出兩個比較常見的例子。

##### DES
----------

根據[維基百科](https://zh.wikipedia.org/wiki/%E8%B3%87%E6%96%99%E5%8A%A0%E5%AF%86%E6%A8%99%E6%BA%96)的描述：

> 資料加密標準（Data Encryption Standard, DES）是一種對稱密鑰加密塊密碼演算法，1976年被美國聯邦政府的國家標準局確定為聯邦資料處理標準（FIPS），隨後在國際上廣泛流傳開來。它基於使用56位金鑰的對稱演算法。這個演算法因為包含一些機密設計元素，相對短的金鑰長度以及懷疑內含美國國家安全局（NSA）的後門而在開始時有爭議，DES因此受到了強烈的學院派式的審查，並以此推動了現代的塊密碼及其密碼分析的發展。

從以上內容，我們可以得到一些資訊，DES的金鑰是56 bits，但DES每一個加密區塊是64 bits。

問題來了，不是說好金鑰長度要跟區塊一樣長嗎？

其中，有 8 bits 是同位元檢查碼，是用來檢出錯誤，所以真正用來加密的長度只有 56 bits。

運算的時候，會把鑰匙折一半得到一對各32 bits的子鑰匙並分別對也被折半的區塊加密。

那麼，如果資料短於 64 bits 怎麼辦？

有一種手法叫做**padding**，它加密前會將資料填補`0`，讓長度剛好能夠被64整除。

詳細算法流程可以[參照這裏](http://www.tsnien.idv.tw/Security_WebBook/chap2/2-8%20DES%20%E5%AF%86%E7%A2%BC%E7%B3%BB%E7%B5%B1.html)。


##### AES
----------

這個算法又稱Rijndael演算法，是美國國家安全局NIST有鑑於DES被發現不安全且被破解，於是公開徵求加密演算法，於是產生了這個進階加密標準（Advanced Encryption Standard, AES）加密演算法。

它跟DES有根本上的不同，它把金鑰變資料塊變成矩陣來進行加密運算。

詳細算法流程可以[參照這裏](http://www.tsnien.idv.tw/Security_WebBook/chap2/2-10%20AES%20%E5%AF%86%E7%A2%BC%E6%A8%99%E6%BA%96.html)。



#### 非對稱式加密
----------

區塊非對稱加密的算法也有很多種，這邊也提出兩個比較常見的例子。

##### RSA
----------

這個加密算法可以用比較簡單的方式描述。

首先，我們要產生一對公私鑰：
1. 選擇兩個大的質數： p, q
2. 計算 N = p ꞏ q
3. 計算 r = ɸ(N) = ɸ(p) ɸ(q) = (p-1)(q-1)
4. 找一個小於 r 且互質的整數 e
5. 找 e 的mod倒數 d 使 e ꞏ d ≡ 1 (mod r)
6. 獲得公鑰：(N ,e), 私鑰：(N ,d)

ɸ(N)是尤拉函數，也就是從1到N，有多少跟N互質的數。
假設N是7，我們知道是一個質數，從1到7總共有6個數字1, 2, 3, 4, 5, 6都跟7互質，所以ɸ(7)就等於6。
由這邊可以知道當N是質數的話，ɸ(N) 就等於N-1。

那麼，我們舉一個產生鑰匙的簡單例子來看看：

> N = p \* q = 47 \* 71 = 3337
> φ(N) = 3220 *(46\*70)*
> 找一個和 3220 互質的數字，得鑰匙 (N, e) = (3337, 79)
> 有p、q跟e，得另一個對應的鑰匙 (N, d) = (3337, 1019)

接下來，我們要使用這對鑰匙來進行加解密。

先搬出公式：
> 公鑰加密 C ≡（M）^e(mod N)
> 私鑰解密 M ≡（C）^d(mod N)

其中，C是密文(cipher)，M是明文(message)。

相反的，我們也可以用私鑰加密，公鑰解密：
> 私鑰加密 C ≡（M）^d(mod N)
> 公鑰解密 M ≡（C）^e(mod N)

我們可以發現兩種不同加解密方式只有`d`跟`e`互換而已。

接著，我們舉一個簡單的範例來看看，但要注意的是實際使用並不會這麼簡單。

> 有一段明文M以16進位(hex，更細來看也可以說是binary)表示爲 **0069 0073**
> 用剛剛範例的公鑰加密，把明文分成兩塊0069跟0073分別加密
> C1 = **0069**^79 (mod 3337) = 00C1
> C2 = **0073**^79 (mod 3337) = 02DC
> 最後合併得到結果C = **00C1 02DC**
> 解密則是反向操作，拿私鑰解密，會得到原本的明文 0069 0073
> 這邊要注意的是不能比每一塊大小不能比N還大，因爲我們是取餘數，如果比較大的話，根據同餘的計算，加密會加成錯誤的結果，解密當然也會解成錯誤的明文（也就是解密失敗）。


剛剛提到padding，RSA的填充方式也有很多種（還有公私鑰的區別），有全部填FF，還有填亂數的做法。這邊要參照[PKCS#1](https://datatracker.ietf.org/doc/html/rfc3447)去瞭解它有哪些填充方式。（我覺得最麻煩的是RSA-OAEP）

另外，看到這邊就知道加解密不能直接字串去進行數學運算，因爲數學，總是要數字嘛，所以我們得把字串轉成數字。那麼，唯一的方法就是二進位格式。

最後，還是得強調加解密只是數學運算，並不會因爲無法計算導致無法加解密，而我們是要看結果正不正確才知道成功還是失敗。

##### ECC
----------

這個加密演算法加解密速度跟在短金鑰的狀況演算法安全度都勝過RSA，現在不全面改用是因爲專利問題跟一些歷史因素。
詳細算法流程可以[參照這裏](https://ithelp.ithome.com.tw/articles/10251031)。
	

#### 何謂安全
----------

剛剛提到加密演算法的時候，有提到安全問題，那麼什麼是安全？

安全有分成兩種：

1. 絕對安全
	不管使用任何手段都沒有任何辦法可以破解就是絕對安全。
2. 相對安全
	在有限的資源（包含時間）內，並設定一個範圍，使這段時間內是安全的，這樣就是相對安全。也就是說NIST能夠運用現有資源來推測當前電腦運算速度跟未來發展去判斷RSA-2048到2030年前都是安全的。

那麼，強度竟然能比較，就會有等級概念，這樣要怎麼分？

根據[NIST官方文件](https://csrc.nist.gov/glossary/term/security_strength)定義安全強度（Security strength）如下：

>  A number associated with the amount of work (that is, the number of operations) that is required to break a cryptographic algorithm or system. In this policy, security strength is specified in bits and is a specific value from the set {80, 112, 128, 192, 256}.

我們可以看到官方有個強度集合內五個元素`{80, 112, 128, 192, 256}`，NIST把各個金鑰長度的對稱與非對稱加密推算後，對應到這五個強度。

這邊會有個問題是我們知道AES可以用暴力破解的數量級來換算（像是AES-128用暴力破解所需嘗試次數爲2^128次，安全強度就是128），那麼RSA這種用餘數定理的數學運算加密要如何換算到這五個強度呢？

根據[這邊](https://crypto.stackexchange.com/questions/8687/security-strength-of-rsa-in-relation-with-the-modulus-size)有個解答：

> Those _appear_ to be based on the complexity of the [General Number Field Sieve](https://en.wikipedia.org/wiki/General_number_field_sieve), one of the fastest (if not _the_ fastest) classical factoring algorithms. I confirmed this in Mathematica.
> In the expression, the **n** is a number to factor. Evaluating the expression at **2^b** is a rough approximation of the the time needed to factor a b-bit integer. Here's a table showing the bit-length of the evaluation at 2^1024, 2^2048,…
>```
>   Strength  RSA modulus size   Complexity bit-length
>    80        1024              86.76611925028119
>    112       2048              116.8838132958159
>    128       3072              138.7362808527251
>    192       7680              203.01873594417665
>    256       5360              269.38477262128384
>```

從裏面可以看到它根據大數分解最快速的演算法GNFS推算出RSA-2048對應到112安全強度。[Implementation Guidance for FIPS 140-2 and the Cryptographic Module Validation Program](https://csrc.nist.gov/csrc/media/projects/cryptographic-module-validation-program/documents/fips140-2/fips1402ig.pdf)的第122頁內也有寫到NIST如何用RSA金鑰長度推算出安全強度的公式，裏面的參數是金鑰長度（應該比較直觀），代入後也是可以得到相近的值。


### 雜湊算法
----------

密碼學上有一個東西叫做訊息摘要(Message digest)，也就是將一串資料（有可能是字串，也可能是數字，反正都可以是binary），透過雜湊演算法計算，得到一組固定長度的字串。

那麼，這個雜湊算法會有這些特性：

- 輸入任意長度（其實還是有上限的，像是SHA-1上限是2^64位元長度），輸出爲固定長度

- 原始資料有些許不同，產生的輸出與原先輸出相比會有極大不同

- 強碰撞抵抗力(Strong collision resistance)，也就是不同資料**很難**有相同結果

- 單向函數(One-way function)，函數會破壞資料導致無法反向

雜湊的演算法有很多種，如MD5、SHA-1、SHA-2等等，但這邊就不全部列舉。

我們直接來看看SHA雜湊是怎麼做的。

#### SHA
----------

##### SHA-1
----------


根據[維基百科](https://zh.wikipedia.org/wiki/SHA-1)：

> SHA-1（Secure Hash Algorithm 1）是一種密碼雜湊函式，美國國家安全局設計，並由美國國家標準技術研究所（NIST）發布為聯邦資料處理標準（FIPS）。SHA-1可以生成一個被稱為訊息摘要的**160位**（20位元組）雜湊值，雜湊值通常的呈現形式為**40**個十六進位數。

我們可以注意到上面這段的數字，SHA-1可以產生長度爲 160 bits的摘要值是因爲定義中有講到內部有五個32-bit 暫存器。

那麼，我們看看它的算法是如何做的。

一開始有五個暫存器的初始值分別爲：

```
h0 := 0x67452301
h1 := 0xEFCDAB89
h2 := 0x98BADCFE
h3 := 0x10325476
h4 := 0xC3D2E1F0
```

這些值是固定的，當初他們定義就是這樣了，但爲什麼數字值是這些，我就沒有特別研究了。

接著，它會做一些前處理，把輸入的資料切成448-bit的chunk，再padding成512-bit，以確保是512的倍數。

```
Pre-processing:
append the bit '1' to the message
append k bits '0', where k is the minimum number >= 0 such that the resulting message length (in bits) is congruent to 448(mod 512)
append length of message (before pre-processing), in bits, as 64-bit big-endian integer
```

再來，因爲演算區塊是512-bit爲單位，所以要把訊息全部切成一塊塊512-bit去做。

```
Process the message in successive 512-bit chunks:
break message into 512-bit chunks
for each chunk
    break chunk into sixteen 32-bit big-endian words w[i], 0 ≤ i ≤ 15
	
	Extend the sixteen 32-bit words into eighty 32-bit words:
    for i from 16 to 79
        w[i] := (w[i-3] xor w[i-8] xor w[i-14] xor w[i-16]) leftrotate 1

    Initialize hash value for this chunk:
    a := h0
    b := h1
    c := h2
    d := h3
    e := h4

    Main loop:
    for i from 0 to 79
        if 0 ≤ i ≤ 19 then
            f := (b and c) or ((not b) and d)
            k := 0x5A827999
        else if 20 ≤ i ≤ 39
            f := b xor c xor d
            k := 0x6ED9EBA1
        else if 40 ≤ i ≤ 59
            f := (b and c) or (b and d) or(c and d)
            k := 0x8F1BBCDC
        else if 60 ≤ i ≤ 79
            f := b xor c xor d
            **k := 0xCA62C1D6**

        temp := (a leftrotate 5) + f + e + k + w[i]
        e := d
        d := c
        c := b leftrotate 30
        b := a
        a := temp

    Add this chunk's hash to result so far:
    h0 := h0 + a
    h1 := h1 + b
    h2 := h2 + c
    h3 := h3 + d
    h4 := h4 + e
	
```

我們可以看到中間，有個`k`的變數，如下：

> k := 0x5A827999
> k := 0x6ED9EBA1
> k := 0x8F1BBCDC
> k := 0xCA62C1D6

這些在這邊被稱爲鹽（Salt），但爲什麼是這些數字，我也沒研究。

但是，SHA-1最重要的精髓就是中間那80-round，它把資料壓縮成160-bit，並且在每次迴圈都使用破壞性相加，所以也沒辦法回推初始值是什麼。

最後再把五個暫存器出來的結果拼起來就變成最終輸出。

```
Produce the final hash value (big-endian):
digest = hash = h0 append h1 append h2 append h3 append h4
```

比較詳細的解說可以[參考這邊](http://www.tsnien.idv.tw/Security_WebBook/chap4/4-4%20SHA-1%20%E6%BC%94%E7%AE%97%E6%B3%95.html)。

這邊會有個疑問就是，我們剛剛說因爲五個32-bit暫存器，所以輸出爲160 bits的長度，可是爲什麼一開始維基百科最後一句說輸出其實是四十個十六進位值？

這個問題在[這邊](https://stackoverflow.com/questions/3704010/why-is-a-sha-1-hash-40-characters-long-if-it-is-only-160-bit)可以找到答案。

> One hex character can only represent 16 different values, i.e. 4 bits. (16 = 24)
> 40 × 4 = 160.

因爲我們用字元呈現，那麼每個字元是4-bit，我們就可以推算以字串的表示方式的長度爲 2^160 = 16^40，也就是需要四十個十六進位值來表示。

這邊也有另外一個[解答](https://stackoverflow.com/questions/26441931/sha1-encoding-to-hex-has-40-characters-and-160-bits)

> 160 bits / 8 = 20 bytes; a byte in hex is 2 characters (`00` to `FF`) and 2 \* 20 = 40 hex characters.
> The longer output is the hexadecimally encoded version of the hexadecimal encoded hash.


最後，我們看看空字串的SHA-1雜湊值，如下：

```bash
SHA-1("") = da39a3ee5e6b4b0d3255bfef95601890afd80709
```

想不到吧，空字串也可以有雜湊值。

##### SHA-2
----------

我們來看看維基百科怎麼描述SHA-2：

> SHA-2，名稱來自於安全雜湊演算法2（英語：Secure Hash Algorithm 2）的縮寫，一種密碼雜湊函式演算法標準，由美國國家安全局研發，由美國國家標準與技術研究院（NIST）在2001年發布。屬於SHA演算法之一，是SHA-1的後繼者。其下又可再分為六個不同的演算法標準，包括了：SHA-224、SHA-256、SHA-384、SHA-512、SHA-512/224、SHA-512/256。

由上面這段，我們可以知道SHA-2使用更多參數來提高混合性，進而誕生SHA-224、SHA-256等等雜湊算法。

以SHA-256爲例子來說，它的256我們可以由SHA-1猜測到32 \* 8 = 256。

也就是它擁有八個32-bit暫存器， 如下：

```
h0 := 0x6a09e667
h1 := 0xbb67ae85
h2 := 0x3c6ef372
h3 := 0xa54ff53a
h4 := 0x510e527f
h5 := 0x9b05688c
h6 := 0x1f83d9ab
h7 := 0x5be0cd19
```

它最後會產生256-bit的摘要長度資料。

算法跟SHA-1相比，也是大同小異，可以直接參照[維基百科](https://zh.wikipedia.org/wiki/SHA-2#%E6%BC%94%E7%AE%97%E6%B3%95)。

最後有個問題，不管是SHA-1還是SHA-2，我們說暫存器是32-bit，但是暫存器初始值爲什麼卻有八個十六進位值？

因爲我們用八個十六進位值去表示八個字元長度的字串，也就是一個字元是4-bit，那就是8 \* 4 = 32 bits。


## 結論
----------

我們從對稱與非對稱的加密演算法瞭解可以保障PKI中敘述的資料傳遞的隱密性，也從不同演算法瞭解安全的意義。

雜湊演算法則是因爲低碰撞性的原因，讓我們可以檢查傳遞資料的完整性。

但是PKI中的四大特性，還有身份鑑別與不可否認性還沒提到，這就要等到下次的簽驗章再來解說。


## 尾語
----------

其實自學這些內容還滿辛苦的，都是網路上拼拼湊湊的知識片段，還不見得每個都寫得一樣。

我到目前還沒找到一個寫得完整的書或是網站，包括我自己也覺得我寫得不太好。

如果有問題，歡迎留言討論。

如果是轉載的話，麻煩附上網址，謝謝 😊

