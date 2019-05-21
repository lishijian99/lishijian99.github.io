##group中的push的使用
**伤情最是晚凉天，憔悴厮人不堪言，吆酒催肠三杯醉，寻香惊梦五更寒，
钗头凤斜倾有泪，徒迷花寥我无缘，小楼寂寞心与月，也难如钩也难圆！
大将生来胆气豪，腰横秋水雁翎刀，风吹橐鼓山河动，电闪旌旗日月高，
天上麒麟原有种，穴中蝼蚁岂能逃，太平待到归来日，朕与将军解战袍。**
>上次写了一篇对Aggregation聚合查询聚合查询的使用，有人对我提了group中的push的用法，因为之前也没有用过，通过翻看文档得到了如下的理解，不知道对不对，望共同进步
>我早查询官方文档时的到了如下的结果

![image.png](https://upload-images.jianshu.io/upload_images/13118451-1124abb0c3dddb17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>官方文档中有如下的解释：在结果文档中插入值到一个数组中。于是我猜想应该是在mongo数据库中有一个字段是一个集和，如果想要取到这个结合必然是要用到push这个方法。于是我在实体中加入了如下字段：

![image.png](https://upload-images.jianshu.io/upload_images/13118451-577cdbe580c23dea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>并且在数据库中填写了相对这个字段的数据如下：

![image.png](https://upload-images.jianshu.io/upload_images/13118451-fe5ad93703d4c899.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>写出了如下的聚合代码
```
//数据分组
    private List<Document> statisticsGroup(String dataSetId ,int year) {
        Criteria criteria = Criteria.where("dataSetId").is(dataSetId);
        criteria.and("year").is(year);
        Aggregation agg = Aggregation.newAggregation(
                Aggregation.match(criteria),
                Aggregation.group("month")
                        .first("month").as("month")
                        .count()
                        .as("monthCount")
                        .first("dataSetId").as("dataSetId")
                        .sum("clicks")
                        .as("sumClicks")
                        .sum("downloads")
                        .as("sumDownloads")
                        .first("dataSetType").as("dataSetType")
                        .push("extMetadata").as("extMetadata")
                ,
                Aggregation.sort(Sort.by(new Sort.Order(Sort.Direction.ASC, "month")))
        );
        AggregationResults<Document> result = mongoTemplate.aggregate(agg,Statistics.class, Document.class);
        return result.getMappedResults();
    }
```
>获取封装的数据代码如下：
```
 List<Map<String,Object>> extMetadata = (List<Map<String,Object>>)document.get("extMetadata");
```
>这样就把List<Map<String,Object>>中的数据取了出来
作者理解push的使用是：如果数据库中有集合类型的字段就要通过push来获取。

>想装个逼但是发现用first 和普通的document.get("extMetadata");就能获取集合，很难受白加班了。
