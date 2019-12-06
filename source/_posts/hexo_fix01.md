---
title: 搭建hexo博客并简单的实现多终端同步
typora-root-url: ../../source
typora-copy-images-to: ../images/hexo_fix01.assets
---

​		为了实现多终端同步，花了一个小时，主要是卡在提交上了，现在的CSDN真的不能再看了，太水了，都是从别的地方复制粘贴过来，很难解决你的问题,所以还是需要去看官方的文档。

​		用终端运行下列指令就可以将文档push到source分支里

```shell
git add .
git commit -m 'hexo source post'
git push origin source
```

​		出现问题的就是最后一句，问题如下：

​		<img src="/images/hexo_fix01.assets/image-20191206213429776.png" alt="image-20191206213429776" style="zoom:33%;" />

​		原因是在theme主题里也存在.git文件，所以会提示你：

<img src="/images/hexo_fix01.assets/image-20191206213335461.png" alt="image-20191206213335461" style="zoom: 33%;" />

​		所以在mac里，你要在finder里在theme目录下command+shift+.显示隐藏文件，然后把主题里的.git文件删除，然后就可以push进去了；

<img src="/images/hexo_fix01.assets/image-20191206213652923.png" alt="image-20191206213652923" style="zoom: 33%;" />

​		详细的可以参考：https://righere.github.io/2016/10/10/install-hexo/

