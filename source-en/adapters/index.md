# CloudEvents Adapters

<!-- no verify-specs -->

并不是所有的事件生产者都会在本地生产 CloudEvents。
因此，可能需要一些“适配器”来将这些事件转换为 CloudEvents。
这通常意味着从事件中提取元数据，用作 CloudEvents 属性。
为了促进这些适配器的多个实现之间的互操作性，以下文档展示了应该使用的建议算法:

- [AWS S3](adapters/aws-s3.md)
- [GitHub](adapters/github.md)
- [GitLab](adapters/gitlab.md)
