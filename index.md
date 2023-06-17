---
title: オンラインでホスティングされている手順
permalink: index.html
layout: home
---

# AI エンジニア演習

これらのハンズオン ラボ演習は、Microsoft のコース「[AI-102 Microsoft Azure AI ソリューションの設計と実装](https://docs.microsoft.com/learn/certifications/courses/ai-102t00)」、および同等の [Microsoft Learn のマイペースで進められるモジュール](https://aka.ms/AzureLearn_AIEngineer)をサポートしています。 演習は学習教材に沿って設計されており、教材で説明されているテクノロジを使って学習できます。

この演習を完了するには、Microsoft Azure サブスクリプションが必要です。 [https://azure.microsoft.com](https://azure.microsoft.com) で無料試用版にサインアップできます。

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| 演習 |
| --- |
{% for activity in labs %}| [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

