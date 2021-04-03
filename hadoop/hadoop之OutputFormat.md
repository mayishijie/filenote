# 1. OutputFormat接口实现类
> outputFormat是MapReduce输出的基类，所有实现MR输出都实现了outputformat接口
## 1. TextputFormat:文本输出（默认格式）
> 它把每条记录写为文本行，它的键和值可以是任意类型，因为它调用toString()把他们转为字符串
## 2. SequenceFileOutputFormat
> 它的格式紧凑，容易被压缩
## 3. 自定义outputFormat
### 使用场景
   > 为了控制最终文件的输出路径以及输出格式，我们往往就自定义OutputFormat
### 自定义步骤
   > 1. 自定义类继承FileOutputFormat
   > 2. 改写RecorderWriter,具体改写输出数据的write()方法。
实例

自定义OutputFormat类
```java
public class FilterOutputFormat extends FileOutputFormat<Text, NullWritable>{

	@Override
	public RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext job)			throws IOException, InterruptedException {

		// 创建一个RecordWriter
		return new FilterRecordWriter(job);
	}
}
```

编写RecordWriter类
```java
public class FilterRecordWriter extends RecordWriter<Text, NullWritable> {
	FSDataOutputStream mayiOut = null;
	FSDataOutputStream otherOut = null;
	public FilterRecordWriter(TaskAttemptContext job) {
		// 1 获取文件系统
		FileSystem fs;
		try {
			fs = FileSystem.get(job.getConfiguration());
			// 2 创建输出文件路径
			Path mayiOutPath = new Path("e:/mayiOut.log");
			Path otherPath = new Path("e:/other.log");
			// 3 创建输出流
			mayiOut = fs.create(mayiOutPath);
			otherOut = fs.create(otherPath);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	@Override
	public void write(Text key, NullWritable value) throws IOException, InterruptedException {

		// 判断是否包含“mayi”输出到不同文件
		if (key.toString().contains("mayi")) {
			mayiOut.write(key.toString().getBytes());
		} else {
			otherOut.write(key.toString().getBytes());
		}
	}

	@Override
	public void close(TaskAttemptContext context) throws IOException, InterruptedException {
		// 关闭资源
                IOUtils.closeStream(mayiOut);
		IOUtils.closeStream(otherOut);	
         }
}
```