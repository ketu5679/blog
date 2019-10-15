# es 离线迁移

ref:[reference](https://www.jianshu.com/p/50ef4c9090f0)

##### 方案:
- elasticsearch-dump
- reindex

### elasticsearch-dump
#### 适用场景
适合数据量不大，迁移索引个数不多的场景

#### 使用方式
elasticsearch-dump是一款开源的ES数据迁移工具  
github地址: https://github.com/taskrabbit/elasticsearch-dump

1. 安装elasticsearch-dump
> npm install elasticdump -g

2. 主要参数说明
> --input: 源地址，可为ES集群URL、文件或stdin,可指定索引，格式为：{protocol}://{host}:{port}/{index}  
> --input-index: 源ES集群中的索引  
> --output: 目标地址，可为ES集群地址URL、文件或stdout，可指定索引，格式为：{protocol}://{host}:{port}/{index}  
> --output-index: 目标ES集群的索引  
> --type: 迁移类型，默认为data,表明只迁移数据，可选settings, analyzer, data, mapping, alias  

3. 迁移单个索引
> 以下操作通过elasticdump命令将集群172.16.0.39中的companydatabase索引迁移至集群172.16.0.20。
注意第一条命令先将索引的settings先迁移，如果直接迁移mapping或者data将失去原有集群中索引的配置信息如分片数量和副本数量等，
当然也可以直接在目标集群中将索引创建完毕后再同步mapping与data
```bash
elasticdump --input=http://172.16.0.39:9200/companydatabase --output=http://172.16.0.20:9200/companydatabase --type=settings
elasticdump --input=http://172.16.0.39:9200/companydatabase --output=http://172.16.0.20:9200/companydatabase --type=mapping
elasticdump --input=http://172.16.0.39:9200/companydatabase --output=http://172.16.0.20:9200/companydatabase --type=data
```

4. 迁移所有索引
> 以下操作通过elasticdump命令将将集群172.16.0.39中的所有索引迁移至集群172.16.0.20。 
注意此操作并不能迁移索引的配置如分片数量和副本数量，必须对每个索引单独进行配置的迁移，或者直接在目标集群中将索引创建完毕后再迁移数据
```bash
    elasticdump --input=http://172.16.0.39:9200 --output=http://172.16.0.20:9200
    
```

#### 使用样例
```$xslt
Examples:

# Copy an index from production to staging with mappings:
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=http://staging.es.com:9200/my_index \
  --type=mapping
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=http://staging.es.com:9200/my_index \
  --type=data

# Backup index data to a file:
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=/data/my_index_mapping.json \
  --type=mapping
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=/data/my_index.json \
  --type=data

# Backup and index to a gzip using stdout:
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=$ \
  | gzip > /data/my_index.json.gz

# Backup the results of a query to a file
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=query.json \
  --searchBody '{"query":{"term":{"username": "admin"}}}'
  

```

### reindex
1. 配置reindex.remote.whitelist参数
需要在目标ES集群中配置该参数，指明能够reindex的远程集群的白名单
2. 调用reindex api
以下操作表示从源ES集群中查询名为test1的索引，查询条件为title字段为elasticsearch，将结果写入当前集群的test2索引
```json
 {
     "source": {
         "remote": {
             "host": "http://172.16.0.39:9200"
         },
         "index": "test1",
         "query": {
             "match": {
                 "title": "elasticsearch"
             }
         }
     },
     "dest": {
         "index": "test2"
     }
 }
```
```bash
curl -X POST \
  http://es.ketu.zjh:9200/_reindex \
  -H 'Content-Type: application/json' \
  -d '{
  "source": {
    "index": "test",
    "size": 5000
  },
  "dest": {
    "index": "test2",
    "routing": "=cat"
  }
}'
```