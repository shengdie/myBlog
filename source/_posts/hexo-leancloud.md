---
title: Hexo增加阅读量统计
date: 2016-10-18 00:28:12
categories: 网站
tags: Hexo
toc: ture
---
### 简单的“不蒜子”
[不蒜子](http://busuanzi.ibruce.info/ "busuanzi")来自昵称为“不如”的“码农”，其强大之处是使用非常简单，只需要给自己的网页增加几行代码就开可以实现阅读量统计和访客统计，可以说是建站小白们的救星，比如博主。而对于Hexo的用户们，很多主题实际上都已经添加了不蒜子统计功能，这样大家做的就更简单了，只需要在主题配置里打开相应的设置就行了，如果没有，加入也是非常的简单，具体我就不说了，不如已经说的非常详细，可以[点这儿](http://ibruce.info/2015/04/04/busuanzi/ "ibruce")。

### Leancloud平台
不蒜子虽然简单易用，但是也有缺点，最大的缺点便是它只能统计具体的某一个页面的访问数，但是却不能在缩略列表里显示列表内每一个页面的统计量，比如在主页，一般是N篇博文的缩略内容，不蒜子就不能显示各博文的访问量。
所以呢，要自己动手搞其他的了。[Leancloud](https://leancloud.cn/ "leancloud")就是不错的平台，目前注册用户都有一定的免费额度，对于个人小博客足够用了。而且已经有人写好代码了，出处在[夏末的博客](https://notes.wanghao.work/2015-10-21-为NexT主题添加文章阅读量统计功能.html "xiamo")，根据其给的教程，博主给自己的博客加统计颇费了一番周折，因为其博文目的是指导Next主题的用户，然而目前Next主题已经集成了代码，所以Next主题用户们只需在主题设置里打开，非常方便。而其他主题用户就要自己动手了。其实主要步骤夏末的博文里已经讲的非常清楚，我不在此赘述，我只说一些原作者没说明白的。

#### 主要思路
1. 首先在Leancloud创建一个App，用来存储数据。
2. 然后在网页加一个js脚本，用来实现读取页面信息，并将其push到Leancloud的App，并且每刷新一次页面就将计数+1.
3. 通过页面信息同样可以从Leancloud pull出计数，显示在任何一个地方。

#### 脚本代码
核心的脚本代码在夏末博文里没有更新，实际上夏末后面有更新代码，可以从其博客网页源码里看到，博主在此贴出，并且附一些注释：
{% codeblock %}
script.
  function showTime(Counter) {
    var query = new AV.Query(Counter);
    var entries = [];
    var $visitors = $(".leancloud_visitors"); //$visitors里是所有class为leancloud_visitors的elements
    $visitors.each(function () {
      entries.push( $(this).attr("id").trim() );
    });  //将每个element的id推送的云端，作为计数的主要标志，这个id推荐取页面的path，比如本博文的就是"/websites/2016-10-18-hexo-leancloud.html"
    query.containedIn('url', entries);
    query.find()
      .done(function (results) {
        var COUNT_CONTAINER_REF = '.leancloud-visitors-count';
        if (results.length === 0) {
          $visitors.find(COUNT_CONTAINER_REF).text(0); //没有访问量显示0
          return;
        }
        for (var i = 0; i < results.length; i++) {
          var item = results[i];
          var url = item.get('url');
          var time = item.get('time');
          var element = document.getElementById(url); //根据id找到element
          $(element).find(COUNT_CONTAINER_REF).text(time); //将element下class为leancloud-visitors-count的child的内容置为访问次数
        }
        for(var i = 0; i < entries.length; i++) {
          var url = entries[i];
          var element = document.getElementById(url);
          var countSpan = $(element).find(COUNT_CONTAINER_REF);
          if( countSpan.text() == '') {
            countSpan.text(0);
          }
        }
      })
      .fail(function (object, error) {
        console.log("Error: " + error.code + " " + error.message);
      });
  }
  function addCount(Counter) { //基本和showTime一样，不过是用在详情页面，刷新一次+1
    var $visitors = $(".leancloud_visitors");
    var url = $visitors.attr('id').trim();
    var title = $visitors.attr('data-flag-title').trim();
    var query = new AV.Query(Counter);
    query.equalTo("url", url);
    query.find({
      success: function(results) {
        if (results.length > 0) {
          var counter = results[0];
          counter.fetchWhenSave(true);
          counter.increment("time");
          counter.save(null, {
            success: function(counter) {
              var $element = $(document.getElementById(url));
              $element.find('.leancloud-visitors-count').text(counter.get('time'));
            },
            error: function(counter, error) {
              console.log('Failed to save Visitor num, with error message: ' + error.message);
            }
          });
        } else {
          var newcounter = new Counter();
          /* Set ACL */
          var acl = new AV.ACL();
          acl.setPublicReadAccess(true);
          acl.setPublicWriteAccess(true);
          newcounter.setACL(acl);
          /* End Set ACL */
          newcounter.set("title", title);
          newcounter.set("url", url);
          newcounter.set("time", 1);
          newcounter.save(null, {
            success: function(newcounter) {
              var $element = $(document.getElementById(url));
              $element.find('.leancloud-visitors-count').text(newcounter.get('time'));
            },
            error: function(newcounter, error) {
              console.log('Failed to create');
            }
          });
        }
      },
      error: function(error) {
        console.log('Error:' + error.code + " " + error.message);
      }
    });
  }
  $(function() { //可以认为是main函数
    var Counter = AV.Object.extend("Counter");
    if ($('.leancloud_visitors').length == 1) {
      addCount(Counter); //如果在详情页，执行addCount
    } else if ($('.post-title').length > 1) { // 注意post-title是标题的class，不同的主题可能不一样，博主因为这浪费了一天时间
      showTime(Counter); //如果在列表页，执行showTime
    }
  });
{% endcodeblock %}

说明一下，博主主题用的语言是```jade```，所以一开始的```script.```是```script element```的关键词，转成```html```就是```<script>...</script>```，所以脚本主体是```script.```以下的部分。
这个脚本，和之前的Leancloud提供的```av-core-mini-0.6.1.js```和```AV.initialize(App_id,App_key)```放在你需要显示计数的页面里。然后在需要显示数量的地方加入：
{% codeblock jade代码 %}
span.leancloud_visitors(id=url_for(page.path), data-flag-title=page.title)
  span.leancloud-visitors-count
{% endcodeblock %}
{% codeblock html代码 %}
<span id="path of page" data-flag-title="title of page" class="leancloud_visitors">
<span class="leancloud-visitors-count"></span>
</span>
{% endcodeblock %}
然后就应该可以显示了。
