---
date: '{{ .Date }}'
draft: true
categories:
- Category A
- Category B
tags:
- Tag A
- Tag B
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
---
