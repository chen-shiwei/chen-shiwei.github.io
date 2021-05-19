---
title: "Raft"
date: "2020-12-23 17:58:22"
tag: ["Raft"]
categories: ["Raft"]
draft: false
---

## Raft 
> 共识算法


## 一.相关项目
- Etcd
    - 为什么不大数据量的存储？
- Consule
    - 为什么提供3种一致性模式？
        - default
        - consistent
        - stale
- Tikv
    - 大数据量实现
      - multi raft？

## 二.基本概念

- 投票机制实现容错和共识
- raft集群的操作称为提案，每一个提案，必须>n/2 节点同意才能提交
- raft角色
  - Leader
  - Follower
  - Candidate