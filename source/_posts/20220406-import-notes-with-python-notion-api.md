---
title: 使用Python Notion API來自動匯入筆記
categories: blog
tags:
  - blog
  - notion
  - python
  - api
abbrlink: e55783e1
date: 2022-04-06 09:58:00
---

## 前言

---

最近發現有些東西還是得線上搞會比較方便

所以從 Obsidian 轉到 Notion

但我又不想一篇篇手動匯入，於是使用了 Notion 的 API

<!--more-->

## 內容

---

Notion 幾乎所有的功能都能通過送 request 來達成

網路上也有 javascript 的 API 版本，但我選擇使用非官方的 Python API [Notion SDK in Python](https://github.com/ramnes/notion-sdk-py)

文件方面，我參考了官方的[API 文件](https://developers.notion.com/reference/intro)

裏面寫得非常詳盡，但看得出來疏於更新跟細節不完整

所以整個使用過程不是很舒服，我還詢問 Notion 官方人員跟非官方的 SDK 開發者進行 payload 的調整

我直接在 repo 發文 [Reference](#Reference) XD

最後終於弄出來能夠自動新增 page 並帶有預設值的腳本

以我的需求來說，必須要可以自動填入標題跟內文，最好是屬性也可以帶入

大概會長這樣

![image](https://user-images.githubusercontent.com/10178964/160292037-a2d80819-1889-4e29-b98b-0180365e84c1.png)

以下是我自動匯入的 code

```python
from notion_client import Client

# Initialize the client
# 取得token的方法，可以參考官方文件這邊 https://developers.notion.com/docs/getting-started
notion = Client(auth=NOTION_TOKEN)


def insert_to_notion(date: str, content: str, title="Work note", tag="work"):
    # 這邊需要`DATABASE_ID`，可以在網頁版Notion點選page後，網址最後會有一個id，可以在這邊抓取
    # 例如 https://www.notion.so/natlee/test-4f3f55661c333e5585660c4c35e10533
    # 這邊的DATABASE_ID就是`4f3f55661c333e5585660c4c35e10533`

    notion.pages.create(
        **{
            "parent": {"database_id": DATABASE_ID},
            "properties": {
                "title": {"title": [{"type": "text", "text": {"content": title}}]}, # 標題的屬性跟標題是什麼
                "Tags": {"type": "multi_select", "multi_select": [{"name": tag}]}, # 標籤的屬性跟標籤是什麼
                "Created": {"date": {"start": date}}, # 建立時間，我的例子是使用像這樣的資料 `2022-04-21`
            },
            "children": [ # 這邊比較麻煩，要指定內容是哪種屬性，例如paragraph
                {
                    "object": "block",
                    "type": "paragraph",
                    "paragraph": {
                        "rich_text": [{"type": "text", "text": {"content": content}}] # 內文的屬性跟內容
                    },
                }
            ],
        }
    )

```

最後我們就可以用這個腳本批量匯入筆記了

整個過程包含跟官方來回，花了快五天

不過我的例子是，我有五百多篇格式化的筆記要把它們匯入到 Notion 上

手動太麻煩，這樣就直接搞定了（至少我知道手動一定會超過五天 XD）

順帶一提，怕的話可以考慮先用 curl 測試一筆看看

```zsh
#!/bin/zsh

curl --location --request POST 'https://api.notion.com/v1/pages' \
--header 'Content-Type: application/json' \
--header 'Notion-Version: 2022-02-22' \
--header 'Authorization: Bearer <YOUR_NOTION_AUTH_TOKEN>' \
--data-raw '{
    "parent": {
        "database_id": "<YOUR_DATABASE_ID>"
    },
    "properties": {
        "title": {
            "title": [
                {
                    "type": "text",
                    "text": {
                        "content": "test_title"
                    }
                }
            ]
        },
        "Tags": {"type": "multi_select", "multi_select": [{"name": "Daily"}]},
        "Created": {"date": {"start": "2022-03-28"}}
    },
    "children": [
        {
            "object": "block",
            "type": "paragraph",
            "paragraph": {
                "rich_text": [
                    {
                        "type": "text",
                        "text": {
                            "content": "Yo I am here!"
                        }
                    }
                ]
            }
        }
    ]
}'
```

使用後，大概會長這樣

![image](https://user-images.githubusercontent.com/10178964/160533403-96b10da6-99f2-4650-9426-0cf387e6bbc6.png)

## Reference

---

[How can I create new page with some contents of text](https://github.com/ramnes/notion-sdk-py/discussions/121)

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E4%BD%BF%E7%94%A8-python-notion-api-%E4%BE%86%E8%87%AA%E5%8B%95%E5%8C%AF%E5%85%A5%E7%AD%86%E8%A8%98-d25496ae7d18)
