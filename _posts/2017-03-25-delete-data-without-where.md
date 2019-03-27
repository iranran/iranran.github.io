---
layout: post
title: delete真的要加where
categories:
- MySQL
- delete语句
feature_image: "https://picsum.photos/2560/600?image=294"
---

经历过两次delete删数据没加条件的事故

### 事故1

执行delete时没加任何条件，这个是前同事搞的。下班路上接到电话问把数据全删除了怎么办，还好当时是用的腾讯云，数据库每天都有备份，直接找到对应的表把sql下载下来重新执行一次。另外也得意于那张表是个量化使用的历史统计表，只有夜里执行一次，不会产生实时数据。

后来dba把sql_safe_updates加上了，sql_safe_updates参数对不带where条件的update/delete语句进行限制无法执行，这个参数设置后，可以防止误操作把整个表都更新或者删除。

### 事故2

这个是自己弄出来的，当时脑袋有点迷糊，第一次插入的时候数据有误，心想着删除就可以了，执行了下面的sql语句

```sql
delete from tb_common_config where org_id = 7017
```

执行完就想起来换了，7017不只有我插入的数据，还有之前客户的其它配置。找DBA说没有binlog，要恢复得几个小时，让自己搞实在弄不出来再进行恢复。

还好这些操作都是后台管理的数据，数据操作都有日志存储在数据库中，找到最新的操作记录，手动给插入回去问题解决，但还是吓了一大跳。

### 解决方案

针对线上的update delete都要加where，另外操作前也要用select查出来，最好写操作的条件只是id，这样可以有效防止误伤。


<script async src="https://www.googletagmanager.com/gtag/js?id=UA-135360671-1"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'UA-135360671-1');
</script>