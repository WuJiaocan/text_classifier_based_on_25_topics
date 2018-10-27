第一次获取数据错误的原因：【数据已删除】
由于用helper.scan获取bl2的数据，是generator格式，因此，第一次，取获取结果的id之后，拿给ner获取rel之后，想再次从generator中获取对应正负样本id对应的content时，结果为空。
就再去helper.scan获取bl2的数据，但是由于数据一直不断灌进es中，导致前后两次取得bl2数据数量对不上，后面又使用的是zip操作对应id和content，所以数据出错。



第二次获取数据错误的原因：
获取语句用的是
"query": {
    "bool": {
      "should": [
        {"match": {"title": "演讲 培训 大会"}},
        {"match": {"content": "演讲 培训 大会"}},
      ]
   }
},
错误原因：该查询语句会把包含yj,演，讲等结果的文章也查询数来，因为没有设置全词完全匹配。所以样本质量不高，后来做模型的预测结果不太好。



第三次，因为想要批量获取文本，改成用es.search(),发现这次获取的结果不是generator格式了，可以连续两次调用。
匹配必须要在文章title中包含关键词的样本。有大概观察了一下，手动剔除了有些主题关键词，如融资，培训，入选等。
后来在看模型的预测效果的时候，发现很多商业文章都被标为负样本了，感觉还是正样本多样性太少。


Difference Between helper.scan() And es.search():

①  helper.scan()获取的结果是generator格式，若后续需要多次取获得的结果，需要把结果先保存成list;
   es.search()获取的结果可以直接多次调用；
   
②  helper.scan()获取的结果中，解析需要的部分结果时，不需要加上["hits"]["hits"]：
   for item in es_result: 
        final_result.append(item["_id"])
        
   es.search()获取的结果中，解析需要的部分结果时，需要加上["hits"]["hits"]：
   for item in es_result["hits"]["hits"]: 
        final_result.append(item["_id"])
        
        
第四次获取数据，发现之前第三次获取的正样本中，有些文章的rel中的title为空，一般这种文章可以选择剔除，质量不是很好。
获取数据的标准：
    必须在title中包括主题关键字；
    25个主题关键词全部保留。
