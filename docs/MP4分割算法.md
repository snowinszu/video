# MP4分割算法

原文

[MP4文件格式的解析，以及MP4文件的分割算法](https://www.cnblogs.com/haibindev/archive/2011/10/17/2214518.html)



mp4应该算是一种比较复杂的媒体格式了，起源于QuickTime。以前研究的时候就花了一番的功夫，尤其是如何把它完美的融入到**视频****点播**应用中，更是费尽了心思，主要问题是处理mp4文件庞大的“媒体头”。当然，流媒体点播也可以采用flv格式来做，flv也可以封装H.264视频数据的，不过Adobe却不推荐这么做，人家说毕竟mp4才是H.264最佳的存储格式嘛。

这几天整理并重构了一下mp4文件的解析程序，融合了分解与合并的程序，以前是c语言写的，应用在linux上运行的服务器程序上，现在改成c++，方便我在其他项目中使用它，至于用不用移植一份c#的，暂时用不到，等有必要了再说吧。这篇文章先简单介绍一下mp4文件的大体结构，以及它的分割算法，之后再写文章介绍如何把mp4完美应用在点播项目中。

 

**一、MP4格式分析**                  

　　**MP4**(MPEG-4 Part 14)是一种常见的多媒体容器格式，它是在“ISO/IEC 14496-14”标准文件中定义的，属于MPEG-4的一部分，是“ISO/IEC 14496-12(MPEG-4 Part 12 ISO base media file format)”标准中所定义的媒体格式的一种实现，后者定义了一种通用的媒体文件结构标准。MP4是一种描述较为全面的容器格式，被认为可以在其中嵌入任何形式的数据，各种编码的视频、音频等都不在话下，不过我们常见的大部分的MP4文件存放的**AVC(H.264)**或**MPEG-4(Part 2)**编码的视频和**AAC**编码的音频。MP4格式的官方文件后缀名是“.mp4”，还有其他的以mp4为基础进行的扩展或者是缩水版本的格式，包括：**M4V**,  **3GP**, **F4V**等。

　　mp4是由一个个“box”组成的，大box中存放小box，一级嵌套一级来存放媒体信息。box的基本结构是：

 ![img](https://pic002.cnblogs.com/images/2011/254714/2011101702102220.jpg)

　　其中，size指明了整个box所占用的大小，包括header部分。如果box很大(例如存放具体视频数据的mdat box)，超过了uint32的最大数值，size就被设置为1，并用接下来的8位uint64来存放大小。

个mp4文件有可能包含非常多的box，在很大程度上增加了解析的复杂性，这个网页上<http://mp4ra.org/atoms.html>记录了一些当前注册过的box类型。看到这么多box，如果要全部支持，一个个解析，怕是头都要爆了。还好，大部分mp4文件没有那么多的box类型，下图就是一个简化了的，常见的mp4文件结构：

![img](https://pic002.cnblogs.com/images/2011/254714/2011101702485271.jpg)

一般来说，解析媒体文件，最关心的部分是视频文件的宽高、时长、码率、编码格式、帧列表、关键帧列表，以及所对应的时戳和在文件中的位置，这些信息，在mp4中，是以特定的算法分开存放在stbl box下属的几个box中的，需要解析stbl下面所有的box，来还原媒体信息。下表是对于以上几个重要的box存放信息的说明：

![img](https://pic002.cnblogs.com/images/2011/254714/2011101710133455.jpg)

看吧，要获取到mp4文件的帧列表，还挺不容易的，需要一层层解析，然后综合stts stsc stsz stss stco等这几个box的信息，才能还原出帧列表，每一帧的时戳和偏移量。而且，你要照顾可能出现或者可能不出现的那些box。。。可以看的出来，mp4把帧sample进行了分组，也就是chunk，需要间接的通过chunk来描述帧，这样做的理由是可以压缩存储空间，缩小媒体信息所占用的文件大小。这里面，stsc box的解析相对来说比较复杂，它用了一种巧妙的方式来说明sample和chunk的映射关系，特别介绍一下。

![img](https://pic002.cnblogs.com/images/2011/254714/2011101710461660.jpg)

······

获取关键帧列表，进行Mp4分割

**二、MP4文件的分割算法**

所谓“分割”，就是把大文件切成小文件，要实现mp4的分割，

- 首先，需要获取到关键帧列表
- 然后，选择要分割的时间段（比如从关键帧开始）
- 接着，重新生成moov box（注意所有相关的box 以及 box size都需要改变）
- 最后，拷贝对应的数据，生成新文件











