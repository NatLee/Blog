
---
title: 使用Python Notion API來匯入筆記
categories: blog
tags:
  - blog
  - notion
  - python
  - api
date: 2022-04-06 09:58:00
---


## 前言
----------

最近發現有些東西還是得線上搞會比較方便

所以從Obsidian轉到Notion

但我又不想一篇篇手動匯入，於是使用了Notion的API

WIP

<!--more-->

## 內容
----------

```python
from notion_client import Client

# https://github.com/ramnes/notion-sdk-py/discussions/121

# Initialize the client
notion = Client(auth=NOTION_TOKEN)


def insert_to_notion(date: str, content: str, title="Work note", tag="work"):
    notion.pages.create(
        **{
            "parent": {"database_id": DATABASE_ID},
            "properties": {
                "title": {"title": [{"type": "text", "text": {"content": title}}]},
                "Tags": {"type": "multi_select", "multi_select": [{"name": tag}]},
                "Created": {"date": {"start": date}},
            },
            "children": [
                {
                    "object": "block",
                    "type": "paragraph",
                    "paragraph": {
                        "rich_text": [{"type": "text", "text": {"content": content}}]
                    },
                }
            ],
        }
    )

```



## Reference
----------