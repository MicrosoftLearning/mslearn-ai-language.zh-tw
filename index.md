---
title: Azure AI 語言練習
permalink: index.html
layout: home
---

# Azure AI 語言練習

下列練習旨在支援 Microsoft Learn 上[開發自然語言解決方案](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/)的課程模組。


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
