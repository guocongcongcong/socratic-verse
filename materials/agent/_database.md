---
notion-bases: true
schema:
  - id: order
    name: 顺序
    type: number
    visible: true
    width: 60
  - id: level
    name: 层级
    type: select
    visible: true
    width: 100
    options:
      - value: 入门
        color: "#4bb563"
      - value: 进阶
        color: "#4ea6f5"
      - value: 高级
        color: "#e8a138"
  - id: status
    name: 学习状态
    type: status
    visible: true
    width: 110
    options:
      - value: 未开始
        color: "#868686"
      - value: 学习中
        color: "#4ea6f5"
      - value: 已理解
        color: "#4bb563"
  - id: tags
    name: 标签
    type: multiselect
    visible: true
    width: 160
    options:
      - value: agent
      - value: skill
      - value: tool
      - value: mcp
      - value: context
      - value: harness
      - value: memory
      - value: multi-agent
      - value: production
      - value: safety
      - value: benchmark
      - value: beginner
      - value: intermediate
      - value: advanced
  - id: teaching_method
    name: 推荐教法
    type: select
    visible: true
    width: 120
    options:
      - value: V1-苏格拉底
        color: "#4bb563"
      - value: V2-费曼
        color: "#4ea6f5"
      - value: V5-合书复述
        color: "#e8a138"
      - value: V7-对子互教
        color: "#e25555"
      - value: V8-自适应
        color: "#9b59b6"
  - id: est_hours
    name: 预计学时
    type: number
    visible: true
    width: 80
  - id: confidence
    name: 可信度
    type: select
    visible: false
    width: 100
    options:
      - value: high
        color: "#4bb563"
      - value: medium
        color: "#e8a138"
      - value: low
        color: "#e25555"
  - id: created
    name: 创建
    type: date
    visible: false
    width: 100
  - id: updated
    name: 更新
    type: date
    visible: false
    width: 100
views:
  - id: default
    type: table
    filters: []
    sorts:
      - columnId: order
        direction: asc
    hiddenColumns:
      - confidence
      - created
      - updated
    columnWidths: {}
  - id: board-status
    name: 学习看板
    type: board
    filters: []
    sorts: []
    hiddenColumns: []
    columnWidths: {}
    groupByColumnId: status
  - id: board-level
    name: 按层级
    type: board
    filters: []
    sorts: []
    hiddenColumns: []
    columnWidths: {}
    groupByColumnId: level
---
