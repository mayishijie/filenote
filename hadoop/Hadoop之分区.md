# 1. 自定义分区步骤

1. 继承partitioner，重写getPartition()方法

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56807637c60c427c8e8c4245f69ca303~tplv-k3u1fbpfcp-watermark.image)
2. 在job驱动中，设置自定义partitioner
>job.setPartitionerClass(MyPartitioner.class);
3. 自定义partitioner之后，需要设置响应数量ReduceTask
> job.setNumReduceTasks(5);
# 2. 分区总结

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0c75804058045ffb9c8cd4489780276~tplv-k3u1fbpfcp-watermark.image)

---
# 3. WritableComparable排序
> MapTask和ReduceTask,均会对key进行排序，这是Hadoop的默认行为，任何应用程序都会排序，不管应用是否需要。且默认都是对key按照字典顺序排序，实现方式是**快排序。**

## 排序概述：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2959aa9fcfb841bf8d005f3b7fd2c6a2~tplv-k3u1fbpfcp-watermark.image)
## 排序分类

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7bea01de26248d590b1141c4b71e900~tplv-k3u1fbpfcp-watermark.image)

全排序
> 1. 设置只有一个Reduce
> 2. 自定义排序方法，key和value调换位置，

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b91e759ce5e41ef93a97f22fb29ce93~tplv-k3u1fbpfcp-watermark.image)
# 4. combiner

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9766fc0afffe4530906d9700c2ee853e~tplv-k3u1fbpfcp-watermark.image)
# 5. 自定义combiner
1. 继承Reducer，实现reduce方法
```java
public class WordcountCombiner extends Reducer<Text, IntWritable, Text,IntWritable>{

	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException {

        // 1 汇总操作
		int count = 0;
		for(IntWritable v :values){
			count += v.get();
		}

        // 2 写出
		context.write(key, new IntWritable(count));
	}
}
```
2. job设置驱动类
```java
job.setCombinerClass(WordcountCombiner.class);
```