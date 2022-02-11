---
title: PKCS#1中RSA的加解密填充圖解（v1_5跟OAEP）
categories: study
tags:
  - cryptograph
  - security
  - encoding
  - decoding
  - RSA
  - PKCS
abbrlink: 833732900
date: 2021-10-31 00:00:00
---


## 前言
----------

這篇算是之前資安常識裏面提到RSA加密的延伸補充。

我們說的RSA加密不是單純只有加密，還必須加料。

這樣才能夠有一定的混淆性。

這邊記錄一下PKCS#1裏面定義的RSA加解密流程。

<!--more-->

## 內容
----------

下面部分內容會從PKCS#1中節錄，但這邊只討論加解密，並不會談到簽驗章流程。

### RSA Cryptography Specifications
----------

#### Key Types
----------

這邊寫在PKCS#1裏面的Sec.3。

我們知道RSA是非對稱加密，然後會產生兩把鑰匙，一把是公鑰，另一把則是私鑰。

但是，平常在加解密的時候，我們要怎麼知道到底哪個是私鑰？哪個是公鑰？

在PKCS#1裏面就有分別定義公鑰跟私鑰的長相。

- 公鑰（Public Key）

	```js
	RSAPublicKey ::= SEQUENCE {
	 modulus INTEGER, -- n
	 publicExponent INTEGER -- e
	}
	```

	這裏面有兩個參數，分別是modulus跟publicExponent。
	
	就是我們熟知的公鑰(N, e)。
	
	其中，N是兩個質數p、q相乘。
	那麼，e是根據費馬小定理去計算：
	
	> r = ɸ(N) = ɸ(p) ɸ(q) = (p-1)(q-1)
	
	然後，找一個小於 r 且互質的整數，就是e。
	
	這個e通常是`65537`，因爲安全性問題。
	
	可以看看[這邊](https://crypto.stackexchange.com/questions/3110/impacts-of-not-using-rsa-exponent-of-65537/3113)的解釋。

- 私鑰（Private Key）

	```js
	RSAPrivateKey ::= SEQUENCE {
	 version Version,
	 modulus INTEGER, -- n
	 publicExponent INTEGER, -- e
	 privateExponent INTEGER, -- d
	 prime1 INTEGER, -- p
	 prime2 INTEGER, -- q
	 exponent1 INTEGER, -- d mod (p-1)
	 exponent2 INTEGER, -- d mod (q-1)
	 coefficient INTEGER -- (inverse of q) mod p
	 otherPrimeInfos OtherPrimeInfos OPTIONAL
	}
	Version ::= INTEGER
	```

	從定義裏面，我們可以看到私鑰裏面裝得東西超級多。
	
	除了使用的版本(version)外，還有剛剛我們提到的公鑰，沒錯私鑰裏面也有公鑰(N, e)。
	
	裏面還有私鑰的d跟組成N的p、q。

	這邊除了這些東西外，還有一些神奇的定義就是exponent1、exponent2、coefficient跟otherPrimeInfos。
	
	otherPrimeInfos還好理解，就是有關質數的一些其他資訊。
	
	但其他的項目是什麼作用呢？

	就是一種建表的方式，有些東西先算好寫好，避免日後運算再花時間。


#### Data Conversion Primitives
----------

這邊的內容在PKCS#1的Sec.4。
   
主要內容是定義函數，介紹數字跟二進位文字（可以說是binary string）的互轉。

學海無涯，這邊推廣一下python：

```python
# 這樣就可以表示binary string了
b'Hello World'
```

Binary string也可以說是octet string或是byte string。

聽說會取名`octet`是因爲很容易跟`byte`搞混。

Octet string的範例 -> `af:ec:ec:8a:f4:09:52:53:69:d4:da:54`

那爲什麼要轉來轉去？

答案是因爲RSA是數學運算，那文字沒辦法直接計算加密，我們得轉成數字才能計算。

##### I2OSP

Integer-to-Octet-String primitive
數字轉文字

> **I2OSP(x, L)**
> x: nonnegative integer, but less than 256^L.
> L: octet string 的字節長度(length)，一個字節是8-bit。
>> 這邊有個範例
>> I2OSP(4095, 2) = 0F:FF

##### OS2IP

Octet-String-to-Integer primitive
文字轉數字

> **OS2IP(X)**
> X: octet string，出來的輸出就是按照位數進行二進位值的相加。
>> 這邊有個範例
>> OS2IP(0F:FF) = 4095（因爲`0x0FFF = 0000 1111 1111 1111`）

#### Cryptographic Primitives
----------

這邊定義在PKCS#1的Sec.5。

這節主要是定義加解密的函數，分別有公私鑰應用在加解密的狀況。

畢竟PKCS本身就是個大定義，它爲了後面不再敘述這些東西，這些函數就當成一個箱子，東西放進去，結果就出來了。

在密碼學上面，只要你引用了PKCS，除非你是RSA其中一位，不然這世界上幾乎沒有人可以再質疑你。

- 公鑰加密 RSAEP ((n, e), m) 
- 私鑰解密 RSADP (K, c)
- 私鑰簽章 RSASP1 (K, m)
- 公鑰驗章 RSAVP1 ((n, e), s)

其中，m就是message，c就是cipher，K是私鑰，s是signature。

不過這篇文章只會講加解密，並不會談到簽章的部分。

那麼，上面有應用到私鑰解密的部分，可以使用中國餘式定理的解法以加速運算。

   
##### 中國餘式定理加速運算

我們知道明文`m = c^d mod n`，`c`是密文，`d`是私鑰組成的其中一個元素，`n`則是兩個大質數`p`跟`q`相乘。

那麼，我們可以把這個公式依照餘數定理轉換形式得到兩式：

> c^d mod p = m1 ---(1)
> c^d mod q = m2 ---(2)

其中，(1)式中的`m1`可以使用費馬小定理簡化運算，我們把`d = d mod (p-1)`帶入：

> c^(d mod (p-1))  mod p

m2同理。

接着，我們令一常數爲`h`，(2)式可以改寫爲`c^d = m2 + hq`，代入第(1)式：

> (m2 + hq) mod p = m1
> → (m1 - m2) mod p = hq

因爲`p`跟`q`互質，我們可以將兩邊同乘`q`的倒數`q^(-1)`得到：

> ((m1 - m2 ) q^(-1))  mod p = h q q^(-1)
> -> h = ((m1 - m2 ) q^(-1))  mod p

最後，我們知道`n=pq`，所以可以推得：

> m = c^d mod n = m2 + hq

因爲這邊公式沒有用數學格式表示，可能比較難看，可以參考[這邊](http://jianiau.blogspot.com/2014/05/rsa-decrypt-with-crt.html)，會有比較好看的敘述。

程式上的實作可以參考[這邊](https://www.geeksforgeeks.org/weak-rsa-decryption-chinese-remainder-theorem/)，以下是節錄並修改成python3版本的程式碼。

```py
# Function to find the gcd of two 
# integers using Euclidean algorithm
def gcd(p, q):
    if q == 0:
        return p
    return gcd(q, p % q)
  
# Function to find the
# lcm of two integers 
def lcm(p, q):
    return p * q / gcd(p, q)
  
# Function implementing extended
# euclidean algorithm
def egcd(e, phi):
    if e == 0:
        return (phi, 0, 1)
    else:
        g, y, x = egcd(phi % e, e)
        return (g, x - (phi // e) * y, y)
  
# Function to compute the modular inverse
def modinv(e, phi):
    g, x, y = egcd(e, phi)
    return x % phi
  
# Implementation of the Chinese Remainder Theorem
def chineseremaindertheorem(dq, dp, p, q, c):
    m1 = pow(c, dp, p) # Message part 1
    m2 = pow(c, dq, q) # Message part 2
    qinv = modinv(q, p)
    h = (qinv * (m1 - m2)) % p
    m = m2 + h * q
    return m
  
# Driver Code
p = 9817
q = 9907
e = 65537
c = 36076319
d = modinv(e, lcm(p - 1, q - 1))
  
"""
pow(a, b, c) calculates a raised to power b 
modulus c, at a much faster rate than pow(a, b) % c
Furthermore, we use Chinese Remainder Theorem as it
splits the equation such that we have to calculate two
values whose equations have smaller moduli and exponent  
value, thereby reducing computing time.
"""
  
dq = pow(d, 1, q - 1)
dp = pow(d, 1, p - 1)
print(chineseremaindertheorem(dq, dp, p, q, c))
```

#### Encryption Schemes
----------

這個部分來說明Sec.7的內容。

主要是對於加解密額外過程的解說，也就是加密前的『填充（padding）』部分。

那爲什麼要padding呢？

因爲我們知道明文長度不一定與金鑰長度相同，而且區塊加密的狀況，我們必須要將明文切成一塊塊來進行加密。

但是，就算切成一塊塊還是會有長度不夠的狀況，因此我們還是需要做padding。

而padding除了能夠增加混淆性外，剛好也能夠把我們是用公鑰還是私鑰加密的資訊包含進去。

##### RSAES-PKCS1-V1_5

在PKCS1-v1_5中定義的加密流程如以下順序：

1. 原文經過EME-PKCS1-v1_5編碼產生EM（這邊就是所謂的padding了）
2. 經過OS2IP函數把EM轉成正整數以進行RSA裏面的數學運算
3. 經過RSAEP的數學運算加密EM
4. 把剛剛加密的結果經由I2OSP轉成長度爲k的密文C

很少人會跟你講PKCS#1講得重點是填充，大部分人都會說PKCS#1在說加解密。

其實也沒有錯，畢竟填充只是其中一個步驟。但個人認爲這才是精髓，因爲沒講誰會知道。

解密流程就是反過來做，如下：

1. 密文C經過OS2IP轉成正整數
2. 把剛剛的正整數經由RSADP解密
3. 上一步的結果經由I2OSP轉成長度爲k的EM
4. 根據EME-PKCS1-v1_5解碼EM得到原文M

###### EME-PKCS1-v1_5 Encoding & Decoding

這邊是講解Padding的過程，也就是EME-PKCS1-v1_5如何編碼。

解碼就不提了，因爲按照步驟倒回來就可以了。

上面有說到編碼產生`EM`，那這個`EM`（Encoded Message）到底是怎麼組成的？

PKCS裏面是這樣寫的：

> EM = 00 || BT || PS || 00 || M

裏面的`||`是分割符號，`BT`是資料塊的型別（Block Type），`PS`是填充用字串（Padding String），`M`是明文資料（Message）。

根據這個定義拼湊出EM，我們就可以得到一個具有PKCS承認合法的padding格式。

其中，BT跟PS的用途如以下：

- 資料塊格式（BT）
	用來標示我加密使用的是公鑰還是私鑰。
	Private Key：00, 01（00容易混淆，不建議使用）
	Public Key：02

- 填充用字串（PS）
	用來把整個EM填充成同金鑰長度的資料塊。
	長度是金鑰長度-明文長度-前導00的1 octet-BT的1 octet-分割用00的1 octet，	也就是`金鑰長度-明文長度-3 octets`。

	如果BT是00，則PS全部填00。（明文開頭是00就完蛋，不建議使用）
	如果BT是01，則PS全部填FF。
	如果BT是02，則PS用非0的亂數填充。

接下來又到了歡樂問答時間，這邊有三個問題。

1. 爲什麼要前導00？
	沒有爲什麼，定義說的就這樣。

2. 爲什麼使用私鑰的時候，全部填FF？
	有一種說法是因爲安全性。因爲全填FF的話，可以讓值變很大以防止透過運算的方式攻擊。也可能還有其他原因要再想想。

3. 亂數看起來比較安全，爲什麼不填充亂數就好？
	這邊可以想想看，我們公鑰加密才是填充亂數是爲什麼。主要是目的不同，因爲公鑰加密是爲了在充滿敵人的環境傳遞訊息，所以需要高度的混淆性。
	私鑰加密則是一種認可訊息的方式，別人只管驗證就好，也就不需要那麼麻煩。

最後，我們直接看個以OpenSSL實作加密填充的例子。

這邊是以RSA-2048做例子。

- 用私鑰加密前的填充

![private_key](https://i.imgur.com/XBhWSgm.png)

- 用公鑰加密前的填充

![public_key](https://i.imgur.com/SxnPfCz.png)


##### RSAES-OAEP

在RSAES-OAEP中定義的加密流程如以下順序：

1. 原文經過EME-OAEP編碼產生EM（這邊就是所謂的padding了）
2. 經過OS2IP函數把EM轉成正整數以進行RSA裏面的數學運算
3. 經過RSAEP的數學運算加密EM
4. 把剛剛加密的結果經由I2OSP轉成長度爲k的密文C

解密流程就是反過來做，如下：

1. 密文C經過OS2IP轉成正整數
2. 把剛剛的正整數經由RSADP解密
3. 上一步的結果經由I2OSP轉成長度爲k的EM
4. 根據EME-OAEP解碼EM得到原文M

基本上，步驟跟剛剛的v1_5一模一樣，只是編解碼的方法不一樣而已。

###### EME-OAEP Encoding & Decoding

基本上，OAEP的填充流程如下：

![oaep-flow](https://i.imgur.com/6SV8Vit.png)

這個圖是參考PKCS內的流程跟[這邊](https://blog.csdn.net/samsho2/article/details/84255173)畫的，但是光看圖可能還是會不太明白實際上是怎麼跑的。

這邊就以例子來演示填充的編碼流程，解碼則是按照步驟倒回來就行了。

![1](https://i.imgur.com/onAOFL3.png)

![2](https://i.imgur.com/Ui4NmUh.png)

![3](https://i.imgur.com/63qxp8P.png)

![4](https://i.imgur.com/qs5M1Jj.png)

![5](https://i.imgur.com/7zgvnbW.png)

![6](https://i.imgur.com/O2349Yt.png)

![7](https://i.imgur.com/FCFlPXi.png)

這邊可能會有兩個問題。

1. `Label`是什麼？
	Label通常是空字串（之前有說過空字串也可以雜湊），如果有需要使用的話必須再自行定義。PKCS#1的v2.2裏面有說如果用空字串以外的Label會out of scope。

2. `MGF`是什麼？
	全名是Mask generation function。透過MGF，我們可以長度不等的資料用遮罩來罩住，讓它能夠產生一個固定長度的資料以便我們後面做XOR（長度必須相同才能做嘛）。通常是使用雜湊相關的函數如SHA-1後，再做截斷來達成。

到這邊OAEP就結束了，這些也是一般我們做RSA的加解密會常用到的填充編碼。

那麼，簽驗章的部分也有在PKCS#1中有定義，這邊就不多做說明了。

詳細可以參考[這邊](https://blog.csdn.net/samsho2/article/details/84255382)，有更多的圖解，包含MGF也有流程說明。

## 總結
----------

這一次，我們提到的RSA實際上的加密並非單純數學運算。

畢竟只有數學運算的話，同樣的明文加密會變成同樣的密文，這樣不用數學，我們使用分析破解，不需要金鑰就可以進行攻擊了。

所以一開始說的RSA加密必須加料就是填充，而填充的方式就如同上面流程一樣複雜。

## 圖片串
----------

這次用到的圖片都放在Imgur的免空，[這裏](https://imgur.com/a/4IpGleY)。


## 結語
----------

原本沒想把padding的部分寫出來，後來想想還是整理一下好了。

這些資料或許可以幫助到一些想進入或剛進入這個領域的人。

真希望當初學習的時候也有這樣的資料可以看 😥

這種類型的知識討厭的地方除了知道就知道，不知道就真的不知道外，還需要各種通靈當初爲什麼這樣設計，很高機率演變成先射箭再畫靶去解釋原因。

不過通靈失敗率也很高就是了，畢竟跟程式一樣，文件不完善外，又不是最初的開發者，怎麼可能知道在寫什麼？😥

如果要轉載，再麻煩標註作者跟網址，謝謝！
