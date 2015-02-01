---
title: Read
layout: page
comments: yes
---

<p>以下是我的书单，数据来自豆瓣，欢迎<a href="http://www.douban.com/people/dechinkey/">关注</a>！</p>
<div id="douban"></div>
<script type="text/javascript" src="/media/js/jquery-1.7.1.min.js"></script>
<script type="text/javascript" src="/media/js/douban.js"></script>
<script type="text/javascript">
 var dbapi = new DoubanApi();
 $(document).ready(function(){
  dbapi.show();
 });
</script>