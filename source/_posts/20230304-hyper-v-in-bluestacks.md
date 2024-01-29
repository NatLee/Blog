---
title: BlueStacks 在有 Hyper-V 的 Windows 11 上的支援狀況
categories: dev
tags:
  - bluestacks
  - nox
  - ldplayer
  - windows
  - hyperv
abbrlink: aa88a654
date: 2023-03-04 00:00:00
---

![hyv-ld-bs-nox](https://i.imgur.com/cEz8vJ2.png)

## 前言

> 2023.11.10 更新：BlueStacks 5.13.220.2101 版本終於支援 Hyper-V 了，但是安裝完重啓後必須使用管理員身份啓動

首先，這篇絕對不是業配，而是親身經歷

我曾經浪費無數的夜晚在重灌Windows測試，最後才發現了解決方法

---

<!--more-->

## 內容

> 先說結論，BlueStacks 5 並不支援 Hyper-V，想要共存請去找夜神模擬器（Nox）或是雷電模擬器（推薦）

雖然開發人員不常使用Windows電腦，但還是有時候得用到

比如說，打遊戲打到凌晨三點，結果老闆突然打電話表示伺服器掛了需要緊急修復

這時，手邊只有Windows電腦的你就不得不使用Windows電腦進行本地測試及遠端偵錯了

常見的虛擬化工具有Docker或是Windows特有的Hyper-V及WSL

然而，如果你使用Windows電腦玩一些糞Game手遊，並且首選的模擬器是BlueStacks，你就會遇到一個嚴重的問題：

> Hyper-V 與 BlueStacks 相依問題

當然，官方一定會說他們支援Hyper-V，甚至在下載的版本連結都直接寫了

![bs-download](https://i.imgur.com/rreKQZm.png)

> 目前這篇文章誕生時，最新版的`BlueStacks 5.11.1.1002` 仍然不支援Hyper-V

接下來就來說說，我就是無法同時使用Hyper-V及BlueStacks的故事

因爲Windows的Docker及WSL都需要打開Windows控制台內的虛擬化設定

就是底下這張圖片的`開啓或關閉Windows功能`

![control-panel](https://i.imgur.com/VDtb7rE.png)

然後我們從裏面打開一些開關

- Hyper V
- Virtual Machine Platform
- Windows Hypervisor Platform
- Windows Sandbox
- Windows Subsystem for Linux

![functions](https://i.imgur.com/YJijk2W.png)

這邊問題就來了，因爲我一開始裝的BlueStacks是不支援Hyper-V的版本（支不支援都給官方說就好了）

所以我按照以下步驟執行重裝BlueStacks

1. 解除安裝BlueStacks
2. 重新開機
3. 在控制台設定虛擬化
4. 重新開機
5. 下載`聲稱支援Hyper-V的BlueStacks版本`並安裝
6. 重新開機
7. 噴錯無法正常打開

![error](https://i.imgur.com/p0U6ClS.png)

然後，看在他還很貼心的提示常見問題，不妨就點進去看看

- [如何在 Windows 上為 BlueStacks 5 開啟 Hyper-V](https://support.bluestacks.com/hc/articles/4412148150157-How-to-enable-Hyper-V-on-Windows-for-BlueStacks-5?locale=zh-tw)

裏面提到`如果啟用了 Hyper-V 而您無法啟動 Hyper-V 兼容版本的 BlueStacks 5，您可以右鍵點擊 BlueStacks 5 圖標並選擇“以管理員身份運行”。`

看起來貌似有用，我就抱著嘗試的心態按右鍵以管理員身份執行了，結果：

![error2](https://i.imgur.com/F6r070F.png)

哈哈...哈...我就知道

於是，只好聯繫BlueStacks官方客服，並且來來回回三十幾封信件

![bs mails](https://i.imgur.com/HOR1ARL.png)

信件往返的過程就是不斷的叫我重灌Windows，然後照著每一步SOP走

當然，如果測試有效並且找到原因，還會有這篇文章嗎？

最後測試當然都無效也沒有找到原因

有趣的是客服很盡責，從台北時間晚上六七點陪我測試到凌晨三四點（對方的時區好像是中亞地區，應該是上班時間吧我猜XD）

反覆重灌的期間還用遠端桌面連過來幫我安裝測試

不過，信件中除了一直叫我重灌外，還一直叫我去看這篇QA [如何在 Windows 上為 BlueStacks 5 開啟 Hyper-V](https://support.bluestacks.com/hc/articles/4412148150157-How-to-enable-Hyper-V-on-Windows-for-BlueStacks-5?locale=zh-tw)

![沒用的QA](https://i.imgur.com/ehf5z0c.png)

這篇就是我一開始遇到Error的時候看到爛的QA啊🤣

最後難過的是，客服也無能爲力，只能請我期待BlueStacks的新版本能夠支援

根據mail上記錄的版本，那時候是`5.5.x.x`，至今BlueStacks已經進版很多次了（已經算是給很多機會了）

所以看在那位認真客服的面子上，我現在才寫這篇文章

也就是`證明了BlueStacks官網說有支援Hyper-V，實際上卻沒有的這件事`

不過，裝新版的BlueStacks我還注意到一些事情

就是它除了包遠端遊玩的程式外，還多了一個虛擬貨幣的錢包程式

這不就代表BlueStacks看起來已經是無心幹好老本行了，那麼再要求它去支援（或是修復？）Hyper-V可能有點強人所難

所以，這篇文章對閱讀的觀衆有幫助的點就是：

> 既然我們沒辦法解決BlueStacks無法相容Hyper-V的問題，那我們就改用能支援Hyper-V的模擬器不就好了？

### 夜神模擬器（NOX Player）

解決方案其中一項就是使用BlueStacks的另一個對手`夜神模擬器（NOX Player）`

當然，我一開始在調研哪些模擬器支援Hyper-V有找到一篇文章說夜神模擬器是不是無法與Hyper-V共存

[Is NoxPlayer Hyper V compatibility a lie ? (Windows)](https://www.reddit.com/r/emulators/comments/woihq1/is_noxplayer_hyper_v_compatibility_a_lie_windows/)

但底下有人回應說目前有版本支援（雖然不是最新版）

![reply](https://i.imgur.com/CGJuwGq.png)

點進去可以看到NOX的日文頁面，可以點擊那個藍色的ダウンロード連結

[NoxとHyperV共存させるには(Win11向けバージョンbeta版)](https://support.bignox.com/ja/else/hypervonbeta)

![page](https://i.imgur.com/MeUaROw.png)

至於爲什麼是日文，我就沒去深究了 😅

> 這邊注意要先把Windows Defender的核心隔離功能關掉，因爲模擬器會需要跟虛擬機互動
> ![core-isolation](https://i.imgur.com/U7mejVs.png)

下載完後（有可能會遇到網路錯誤，多試幾次就可以），直接在已經開啓Hyper-V的環境下安裝

![nox-icon](https://i.imgur.com/GrT4tSg.png)

安裝完成後，雖然會跳出提示，但至少能用，隔壁家老牌BlueStacks可是連開都開不起來呢 🎃

![nox hyper v hint](https://i.imgur.com/oRB8NXr.png)

成功啓動後，左上角會顯示`Hyper-V ON`的標示

![nox hyv on](https://i.imgur.com/DsFYBuv.png)

另外，說一下夜神模擬器的好處是可以直接導出備份

![nox-backup](https://i.imgur.com/Hcowq2h.png)

不像BlueStacks要自己去`C:\ProgramData\BlueStacks_nxt`把Engine資料夾做備份

而且多開還會神機錯亂，不同的多開instance還會有相依性，到底？😅

結尾秀一張大集合

![my-dev](https://i.imgur.com/FymieZv.png)

Hyper-V、WSL、Docker、模擬器全部都能開，一整個就是舒服

### 雷電模擬器（LD Player）

有些人嘗試用了NOX之後，會發現NOX的速度有點遲緩

另一個解決方案就是雷電模擬器

![ld](https://i.imgur.com/yWUmLmg.png)

這個模擬器啓動速度比NOX還快，不妨也可以試試看

然後比較不麻煩的是，直接去官網下載最新版即可

[雷電官網下載區](https://www.ldplayer.tw/other/version-history-and-release-notes.html?log=9)

點進去可以看到他們有做更新日誌

![update-log](https://i.imgur.com/H1r0CIR.png)

安裝完之後，就可以直接用了

![combination](https://i.imgur.com/4L0xjJ0.png)

一樣是Docker正常，WSL正常，模擬器正常！


## 結論

---

這篇文章沒有要diss誰，我希望這篇文章能夠幫助到遇到類似問題的讀者

雖然在開發者在開發時可能會以為同樣的功能可以適用於不同的環境，但實際上可能會因為套件版本的不同或者作業系統環境的差異導致無法兼容

BlueStacks的開發者或許也是一樣，這個問題他們內部可能早就知道

但官網一直都寫支援的狀況下，無法解決也拉不下臉跟大家說明

畢竟，BlueStacks、NOX及雷電都是競爭對手，誰先早一步做完必要的功能就能搶到更多使用者

最後評比一下，在與Hyper-V共存的狀況下，穩定度`雷電(速度最快)>NOX(有時會遲緩)>BlueStacks(完全不能用)`

---

> 2024.01.29 更新
> 補充一下我的PC規格
> CPU: 12th Gen Intel(R) Core(TM) i7–12700 2.10 GHz
> MB: ASUS TUF GAMING B660M-PLUS D4
> RAM: DDR4 16x2 GB
> GPU: ASUS TUF NVIDIA RTX4080
> OS: Windows 11 Pro 23H2 22631.3007
> DISK: Crucial P5 Plus 2TB PCIe M.2 2280SS SSD

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/bluestacks-%E5%8F%8A-nox-player-%E5%9C%A8%E6%9C%89-hyper-v-%E7%9A%84-windows-11-%E4%B8%8A%E7%9A%84%E6%94%AF%E6%8F%B4%E7%8B%80%E6%B3%81-9a8c2ca8f13)
