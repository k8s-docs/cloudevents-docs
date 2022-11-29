# CloudEvents 适配器

由于并不是所有事件生产者在本地都以 CloudEvents 的格式生产事件，因此需要一些"适配器"将这些非 CloudEvents 格式的事件转换为 CloudEvents。要完成这个转换，通常需要将非 CloudEvents 格式事件的元数据抽取出来，用作 CloudEvents 的属性。为了能更好地提升适配器间不同实现的互操作性，以下文件列出了推荐使用的算法：

- [AWS S3](./aws-s3.md)
- [GitHub](./github.md)
- [GitLab](./gitlab.md)
