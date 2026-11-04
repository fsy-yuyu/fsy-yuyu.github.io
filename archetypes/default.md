---
title: "{{ replace .File.ContentBaseName "-" " " | title }}"
date: {{ .Date }}
draft: false
description: ""
---

<!-- 通用的新页面模板：适合「关于」「收藏」「书单」这类独立页面。
     文章的模板在 archetypes/posts.md，用 hugo new content posts/xxx/index.md 创建。 -->

正文从这里开始。
