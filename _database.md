---
notion-bases: true
schema:
  - id: 分类
    name: 分类
    type: text
    visible: true
    width: 80
  - id: 状态
    name: 状态
    type: text
    visible: true
    width: 60
  - id: 日期
    name: 日期
    type: text
    visible: true
    width: 100
views:
  - id: default
    type: table
    filters: []
    sorts:
      - columnId: 分类
        direction: asc
    hiddenColumns: []
    columnWidths: {}
  - id: board-status
    name: 按状态
    type: board
    filters: []
    sorts: []
    hiddenColumns: []
    columnWidths: {}
    groupByColumnId: 状态
---

# 数据库索引

本目录使用 Notion Bases 插件展示所有文档。每个文档的 frontmatter 包含分类/状态/日期字段。
