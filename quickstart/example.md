Example
=========

和其他开源的调度和编排平台不同的一点在于，Eru 致力于提供离线在线一致性，因此通过 cli lambda 子命令我们可以用来做一些离线的计算任务。

比如下面视频中，我们把一个大文本抽成了 100 个随机文件，每个包含若干个单词，使用 Lambda 对每一个文件统计其单词总数。这里我们起了 100 个容器，每个容器 1% 的 CPU 占用。

<script type="text/javascript" src="https://asciinema.org/a/142690.js" id="asciicast-142690" async></script>

我们也可以在装好 core 之后使用 cli 做 agent 的部署，可以参考下面视频：

<script type="text/javascript" src="https://asciinema.org/a/142614.js" id="asciicast-142614" async></script>