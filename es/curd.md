# es学习笔记

ref: http://ju.outofmemory.cn/entry/348964
ref: https://www.elastic.co/guide/en/elasticsearch/reference/6.8/docs-reindex.html

- ## index

1. 创建索引
    - number_of_shards 它控制组成索引的分片数（每个分片最多可以存储2^32个文档）
    - number_of_replicas 它控制每个分片的副本数量，一般建议最少设置为1
    
    ```shell
    curl -X PUT \
      http://es.ketu.zjh:9200/tt-index \
      -H 'Content-Type: application/json' \
      -d '{
      "settings": {
        "index": {
          "number_of_replicas": "1",
          "number_of_shards": "5"
        }
      },
      "mappings": {
        "test_type": {
          "properties": {
            "name": {
              "type": "text"
            },
            "age": {
              "type": "integer"
            }
          }
        }
      }
    }'
    
    ```
    res
    ```json
    {
        "acknowledged": true,
        "shards_acknowledged": true,
        "index": "tt-index"
    }
    ```
    failRes:
    ```json
    {
        "error": {
            "root_cause": [
                {
                    "type": "resource_already_exists_exception",
                    "reason": "index [tt-index/atttvohMSsmpdjIpvagDbg] already exists",
                    "index_uuid": "atttvohMSsmpdjIpvagDbg",
                    "index": "tt-index"
                }
            ],
            "type": "resource_already_exists_exception",
            "reason": "index [tt-index/atttvohMSsmpdjIpvagDbg] already exists",
            "index_uuid": "atttvohMSsmpdjIpvagDbg",
            "index": "tt-index"
        },
        "status": 400
    }
    ```
2. 删除索引

3. 打开关闭索引
