---
title: "FastAPI 常見問題 - SQLalchemy session"
description: "FastAPI 和 SQLAlchemy 是官方主要提到的 ORM tool，本文將會簡單說明兩個工具整合的常見問題，會再帶一點 sqlalchemy scoped_session 的介紹"
excerpt: "FastAPI 和 SQLAlchemy 是官方主要提到的 ORM tool，本文將會簡單說明兩個工具整合的常見問題，會再帶一點 sqlalchemy scoped_session 的介紹"
date: 2023-03-08T15:48:04Z
lastmod: 2023-03-08T15:48:04Z
draft: false
weight: 50
images: []
categories: ["Python"]
tags: ["Python","FastAPI"]
contributors: []
pinned: false
homepage: false
---

最近在壓測系統時，有觀察到 database connection 異常暴漲的問題，原本用 singleton pattern 封裝 database class 解決問題，確保單台 application server 只會有單個 db instance，限制 db connection 不超過 connection pool 的上限，但後續遇到這個錯誤:

調查之後發現，是 FastAPI 專案使用到 sqlalchemy scoped_session 的方式，封裝 session factory，導致錯誤產生。

所以本文將會簡單說明 FastAPI and sqlalchemy 整合的常見問題，會再帶一點 sqlalchemy scoped_session 的介紹

期許讀完這篇文之後，可以了解彼此方法的差異

### 正文開始

一般而言，使用 FastAPI 宣告 database session 的方式有兩種

1. official: 官方建議，透過 FastAPI 支援的 dependency injection 方式，定義產生 session 的 generator，並 depends 給所有 path operation (aka api endpoint) 使用
2. middleware: 在 middleware 定義 session create and session close，session 會跟著 request 傳入 path operation (aka api endpoint) 使用

這兩種方式的想要達成的效果相同，都是想讓 db session 的使用週期可以綁定 request 一起開始和結束，簡單來說就是 request 近來會先建立 db session，當 server 產生 response 之後，可以結束 db session

*兩種方法的差異在於使用範圍，middleware 是全部 api endpoint，DI 則是可以指定範圍甚至可以獨立使用，所以差異大的部分就是使用彈性*

但如果單純參考 Sqlalchemy official doc 會發現有個 contextual session which is a thread local session，這種 session 的使用效果和上述兩種方法的目的相同，接下來就會有個疑問，既然三種方式目的都相同，那彼此差異在哪? 哪個方式比較適合使用?