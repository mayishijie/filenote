# 1. 需求

统计硅谷影音视频网站的常规指标，各种TopN指标：

--统计视频观看数Top10

--统计视频类别热度Top10

--统计视频观看数Top20所属类别

--统计视频观看数Top50所关联视频的所属类别Rank

--统计每个类别中的视频热度Top10

--统计每个类别视频观看数Top10

# 2. 数据结构

## 1. 视频表结构

| 字段        | 备注                        | 详细描述               |
| ----------- | --------------------------- | ---------------------- |
| video id    | 视频唯一id（String）        | 11位字符串             |
| uploader    | 视频上传者（String）        | 上传视频的用户名String |
| age         | 视频年龄（int）             | 视频在平台上的整数天   |
| category    | 视频类别（Array<String>）   | 上传视频指定的视频分类 |
| length      | 视频长度（Int）             | 整形数字标识的视频长度 |
| views       | 观看次数（Int）             | 视频被浏览的次数       |
| rate        | 视频评分（Double）          | 满分5分                |
| Ratings     | 流量（Int）                 | 视频的流量，整型数字   |
| conments    | 评论数（Int）               | 一个视频的整数评论数   |
| related ids | 相关视频id（Array<String>） | 相关视频的id，最多20个 |

## 2. 准备工作

```sql
-- 外部表
create external table video_ori(
    videoId string,
    uploader string,
    age int,
    category array<string>,
    length int,
    views int,
    rate float,
    ratings int,
    comments int,
    relatedId array<string>
)
row format delimited fields terminated by '\t'
collection items terminated by '&'
location '/gulivideo/video/video';
-- 内部表
create table video_orc(
    videoId string,
    uploader string,
    age int,
    category array<string>,
    length int,
    views int,
    rate float,
    ratings int,
    comments int,
    relatedId array<string>
)
stored as orc
tblproperties("orc.compress"="SNAPPY");
-- 导入数据到orc
insert into table video_orc select * from video_ori;
```

## 3. 业务处理

1. 统计视频观看数Top10

   ```sql
   select videoId, `views`
   from video_orc
   order by `views` desc
   limit 10;
   ```

   

2. 统计视频类别热度top10

   ```sql
   -- 分析：视频类别，热度（类别下视频数多的就是热度高）
   select cate,
          count(videoId) cnt
   from (
            select videoId,
                   cate
            from video_orc vo
                     lateral view explode(vo.category) tbl as cate
        ) t
   group by cate
   order by cnt desc
   limit 10;
   
   
   --结果
   Music,179049
   Entertainment,127674
   Comedy,87818
   Film ,73293
    Animation,73293
   Sports,67329
    Games,59817
   Gadgets ,59817
   People ,48890
    Blogs,48890
   ```

   

3. 统计出视频观看数最高的20个视频的所属类别以及类别包含Top20视频的个数

   ```sql
   --分析
   --1. 视频观看数最高的20个视频
   --2. 所属类别
   --3. 类别包含Top20视频
   
   ----------------------------------
   -- 3. 类别下热度top20的视频
   select cate, count(videoId) cnt
   from (
            -- 2.视频id,类别
            select videoId, cate
            from (
                    -- 1.视频观看数top20
                     select videoId, `views`, category
                     from video_orc
                     order by `views` desc
                     limit 20
                 ) t1
                     lateral view explode(t1.category) tbl as cate
        ) t2
   group by cate
   order by cnt
   limit 20;
   
   --结果
    UNA ,1
   People ,2
    Blogs,2
   Music,5
   Entertainment,6
   Comedy,6
   ```

   

4. 