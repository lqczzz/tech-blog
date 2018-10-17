---
title: elasticSearch学习笔记
date: 2018-10-13 22:29:32
tags:
---

# elasticSearch入门

最近在项目中遇到了需要搜索引擎的场景，对用户输入进行自动推荐和补全，elasticSearch是开源的搜索引引擎，入门使用也很简单。

怎么入门：

1. 类比入门：类比一个熟悉的知识点，知识迁移会更容易
2. 简单的事例入门：动手进行简单的一个demo感受一下流程

## 入门

### 1.安装es

前置条件：mac环境(其他环境自行google)，brew工具安装好了，java环境安装好了

步骤：
1. 安装es：
    - brew install elasticsearch
    - es服务端会被安装
2. 安装kibana：
    - brew install kibana
    - kibana可以理解为图形化的es客户端
3. 启动es：brew services start elasticsearch
4. 启动kibana：brew services start kibana

安装es成功之后访问`http://localhost:9200/`可以获得es的状态信息
安装kibana成功之后访问`http://localhost:5601/app/kibana#/dev_tools/console?_g=()`可以对es进行操作

### 2.自动补全

    // hashtag
    {
        id,
        name,
        score, // weight
    }


#### mappings & analysizer

    PUT feed_id
    {
        "mappings": {
            "hashtag": {
                "properties": {
                    "name": {
                        "type": "completion" 
                    }
                }
            }
        }
    }

#### 添加数据

    POST feed_id/hashtag
    {
        "name": {
            "input": ["hashtag name"],
            "weight": 2
        }
    }
    POST feed_id/hashtag/3
    {
        "name": {
            "input": ["hashtag name2"],
            "weight": 3
        }
    }
    POST feed_id/hashtag/4
    {
        "name": {
            "input": ["ashtag name2"],
            "weight": 4
        }
    }
    POST feed_id/hashtag/5
    {
        "name": {
            "input": ["shtag name2"],
            "weight": 5
        }
    }
    POST feed_id/hashtag/6
    {
        "name": {
            "input": ["htag name2"],
            "weight": 6
        }
    }
    POST feed_id/hashtag/7
    {
        "name": {
            "input": ["爱我中华"],
            "weight": 6
        }
    }
    POST feed_id/hashtag/8
    {
        "name": {
            "input": ["爱你中华"],
            "weight": 6
        }
    }
    POST feed_id/hashtag/9
    {
        "name": {
            "input": ["爱你"],
            "weight": 6
        }
    }
#### 查询数据

    POST feed_id/_search?pretty
    {
        "suggest": {
            "hashtag-suggest": {
                "prefix": "h",
                "completion": {
                    "field": "name"
                }
            }
        }
    }

## 基本概念

摘抄自[elasticsearch-definitive-guide-cn](ttps://github.com/looly/elasticsearch-definitive-guide-cn)



## 参考：
