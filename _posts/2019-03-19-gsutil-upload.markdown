---
layout: post
title:  "gsutil 上传文件到 google cloud storage"
date:   2019-03-19 11:27:02 +0800
comments: true
tags:
- gsutil
- google cloud storage
---

### install gsutil
- env:mac os
- refer:https://cloud.google.com/storage/docs/gsutil_install?refresh=1&hl=zh-cn

在命令提示符处输入以下内容：

`curl https://sdk.cloud.google.com | bash`

重启 shell：

`exec -l $SHELL`

授权google sdk:

`gcloud auth login`

会选择授权的用户，用于上传文件

### gsutil upload command
- refer: https://cloud.google.com/storage/docs/gsutil/commands/cp?refresh=1&hl=zh-cn

```
gsutil cp /data/test_node/1.txt gs://test_all/test_node/
```

其中test_all是bucket， 而test_node则是目标文件夹。/data/test_node/1.txt则是源文件的绝对路径，如果要上传的是文件夹， 用cp -r参数。
### 关于boto文件的说明
- refer: https://cloud.google.com/storage/docs/boto-gsutil?refresh=1&hl=zh-cn

```
创建一个可供所有员工读取的中央 boto 配置文件。

如果 gsutil 是随 Google Cloud SDK 一同安装的，您可以使用 gcloud init 来完成此操作。

例如，boto 配置文件可能包含以下内容：

[Boto]
proxy = yourproxy.com
proxy_port = 8080

[GSUtil]
parallel_composite_upload_threshold = 150M
```

