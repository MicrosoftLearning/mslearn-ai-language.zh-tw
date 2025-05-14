---
title: Azure AI 語言練習
permalink: index.html
layout: home
---

# Azure AI 語言練習

下列練習旨在為您提供實際動手學習體驗，您將探索開發人員在 Azure 上建立自然語言解決方案時所執行的一般工作。 

> **請注意**：若要完成練習，您需要有 Azure 訂用帳戶。 如果您還沒有 Azure 帳戶，[可以註冊一個 Azure 帳戶](https://azure.microsoft.com/free)。 新用戶有免費試用選項，其中包含前 30 天的點數。

## 練習

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

<hr>

> **注意**：雖然您可以自行完成這些練習，但它們的設計目的是要補充Microsoft Learn[ 上的課程模組](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/);您可以在其中深入了解這些練習所依據的一些基礎概念。 
