# xiunobbs分析

> NOTE:所有的请求入口均在`index.php`



## index.php

1. `$conf`变量赋值
2. 编译并加载`model.inc.php`和`index.inc.php`

## index.inc.php

1. 获取用户组列表`$grouplist`
2. 获取用户id`$uid`
3. 通过`$uid`获取出`$user`，再通过`$user`拿到用户所属的用户组id`$gid`
4. 通过`$gid`在`$grouplist`中取出用户组`$group`
5. 默认的当前版块`$fid`为0
6. 获取所有的版块列表`$forumlist`
7. 通过`$gid`和`$forumlist`取出有权限查看的板块列表`$forumlist_show`
8. 从`$forumlist_show`中把`fid`和`name`拼成数组到`$forumarr`
9. 获取路由`$route`，默认是`index`
10. 通过一个`switch`结构判断`$route`，然后加载不同的路由页
   1. [route_index.php](#route_index.php)
   2. [route_thread.php](#route_thread.php)
   3. [route_forum.php](#route_forum.php)
   4. [route_user.php](#route_user.php)
   5. [route_my.php](#route_my.php)
   6. [route_attach.php](#route_attach.php)
   7. [route_post.php](#route_post.php)
   8. [route_mod.php](#route_mod.php)
   9. [route_browser.php](#route_browser.php)

## <span id="route_index.php">route_index.php</span>

点击首页或默认时，进入此代码

1. 获取当前访问的页数`$page`
2. 从`$conf`中取出排序方式`$order`
3. 从`$conf`中取出一页显示多少主题`$pagesize`
4. 从`$forumlist_show`可查看的版块中取出版块`fid`数组`$fids`
5. 从`$forumlist_show`中计算出主题的数量`$threads`
6. 通过主题总数`$threads`、当前页数`$page`和每页显示主题数`$pagesize`，计算出分页`$pagination`
7. 通过版块id数组`$fid`、当前页`$page`、页大小`$pagesize`、排序方式`$order`、总主题数`$threads`，查找出当前页要显示的主题列表`$threadlist`
8. 如果是第一页，需要需要查找出置顶主题`$toplist3`，并加入到`$threadlist`中去
9. 再通过`$gid`过滤一下没有权限访问的主题`$threadlist`
10. 编译并加载`view/htm/index.htm`

## <span id="route_thread.php">route_thread.php</span>

点击主题时进入此代码

1. 获取动作`$action`

分两部分

### 发表主题帖

1. 检查用户是否已经登录

2. 通过`$method`获取请求方式并判断(`$mothed`是xiunophp的全局变量)

   - GET请求

     1. 获取版块id`$fid`

     2. 通过`$fid`获取对应的版块信息`$forum`

     3. 通过所有版块列表`$forumlist`和用户组`$gid`过滤出可以发帖的版块列表`$forumlist_allowthread`

     4. 把`$forumlist_allowthread`中的`fid`和`name`做成数组`$forumarr`

     5. 通过判断`$forumlist_allowthread`是否为空判断用户是否有发帖权限

     6. 编译并加载`view/htm/post.htm`
     7. 此时加载出发表主题的页面
     8. 填写主题和内容后，点发表主题的按键是一个`POST`请求

   - POST请求

     1. 获取版块id`$fid`
     2. 通过`$fid`获取对应的版块信息`$forum`
     3. 通过`$fid`和`$gid`判断用户是否有权限在此版块发帖
     4. 获取帖子的主题`$subject`
     5. 获取帖子的内容`$message`
     6. 获取文档类型`$doctype`
     7. 创建主题并返回主题`$tid`
     8. 发帖成功并返回版块页面

### 查看帖子详情

1. 获取主题id`$tid`
2. 获取当前页面`$page`(这里的当前页是指帖子内跟帖的分页)
3. 获取关键字`$keyword`
4. 获取跟帖显示数`$pagesize`
5. 通过`$tid`读取出主题信息`$thread`
6. 通过`$thread`获取出版块id`$fid`
7. 通过`$fid`获取出版块信息`$forum`
8. 通过`$tid`、`$page`、`$pagesize`读取出跟帖信息`$postlist`
9. 如果当前`$page`是第一页的话，那第一条发帖肯定是发帖者发的，需要把它指定为`$first`，并将`$postlist`中的此条信息删除
10. 如果不是第一页的话，那`$first`就是从`$postlist`的第一条开始，因为反正翻页了，怎么展示也无关了
11. 判断`$keyword`中是否有内容，如有有内容，高亮主题中的关键字
12. 判断当前用户是否可以跟帖`$allowpost`
13. 判断当前用户是否可以修改帖子内容`$allowupdate`
14. 判断当前用户是否可以删除`$allowdelete`
15. 过滤不可查看帖子的用户
16. 编译并加载`view/htm/thead.htm`



## <span id="route_forum.php">route_forum.php</span>



## <span id="route_user.php">route_user.php</span>



## <span id="route_my.php">route_my.php</span>



## <span id="route_attach.php">route_attach.php</span>



## <span id="route_post.php">route_post.php</span>



## <span id="route_mod.php">route_mod.php</span>



## <span id="route_browser.php">route_browser.php</span>



