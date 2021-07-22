# xiunobbs分析

> 此分析建立在4.0.4版本
>
> NOTE:所有的请求入口均在`index.php`，管理入口在`admin/index.php`



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

### <span id="route_index.php">route_index.php</span>

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

### <span id="route_thread.php">route_thread.php</span>

点击主题时进入此代码

1. 获取动作`$action`

分两部分

#### 发表主题帖

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

#### 查看帖子详情

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



### <span id="route_forum.php">route_forum.php</span>

点击版块列表中的版块进入此代码

1. 通过解析请求参数获取版块id`$fid`
2. 通过解析请求参数获取版块要展示的第几页`$page`
3. 获取排序方式`$orderby`
4. 给插件预留数组`$extra`
5. 如果`$orderby`没有指定，则默认以`lastpid`排序
6. 通过`$fid`获取出版块信息`$forum`
7. 根据版块id`$fid`和组id`$gid`判断用户是否有读权限
8. 如果当前`$page`页是第1页，则读取出置顶的主题列表，否则为空，来填充`$toplist`
9. 从默认的地方读取主题列表，并对列表进行分页，读取的主题列表存放在`$threadlist`中
10. 编译并加载`view/htm/forum.htm`



### <span id="route_user.php">route_user.php</span>

#### 查看其它用户信息时进入

1. 通过参数获取指定用户id`$_uid`，如果为空，则将当前用户id`$uid`赋值给`$_uid`
2. 通过`$_uid`获取指定用户信息`$_user`
3. 编译并加载`view/htm/user.htm`



#### 查看其它用户的帖子时进入

1. 通过参数获取指定用户id`$_uid`，如果为空，则将当前用户id`$uid`赋值给`$_uid`
2. 通过`$_uid`获取指定用户信息`$_user`
3. 读取`$_user`中的数据给`$header`赋值
4. 从参数中获取要展示的第几页`$page`
5. 页大小`$pagesize`赋值为20
6. 从`$_user`中获取出主题的总数量给`$totalnum`
7. 对主题进行分页`$pagination`
8. 通过`$_uid`、`$page`、`$pagesize`读取出要展示的主题列表`$threadlist`
9. 通过`$gid`对`$threadlist`进行访问过滤
10. 编译并加载`view/htm/user_thread.htm`



#### 登录时进入

1. 点击登录按钮
2. 此时的`$method`为`GET`
3. 编译并加载`view/htm/user_login.htm`
4. 填写用户名/邮箱和密码并点击登录，再次进入此逻辑
5. 此时的`$method`为`POST`
6. 通过参数获取`$email`，它可能是用户名和邮箱，支持使用这两种方式登录
7. 通过参数获取md5之后的密码`$password`
8. 如果`$email`是邮箱的话，则通过邮箱获取出用户信息`$_user`
9. 如果`$email`是用户名的话，则通过用户名获取出用户信息`$_user`
10. 判断密码`$password`是否是md5结果的长度
11. 通过对`$password`加`$_user['salt']`md5结果和`$_user`中的密码对比
12. 对比成功后，则更新登录时间和次数
13. 更新当前`$uid`并设置用户`token`



#### 注册时进入

1. 从全局配置`$conf`中读取是否允许用户注册
2. 此时的`$method`为`GET`
3. 编译并加载`view/htm/user_create.htm`
4. 此时的`$method`为`POST`
5. 通过参数获取`$email`
6. 通过参数获取`$username`
7. 通过参数获取md5之后的密码`$password`
8. 通过参数获取`$code`，邮箱注册验证码
9. 如果开启了注册邮箱验证，则判断`$email`和`$code`是否正确
10. 判断`$email`格式，通过`$email`读取用户数据，如有数据，则说明用户已存在
11. 判断`$username`格式，通过`$username`读取用户数据，如有数据，则说明用户已存在
12. 判断`$password`格式
13. 随机生成一个加密密码的salt`$salt`
14. 对`$password`加`$salt`md5处理并赋值给`$pwd`
15. 用户组`$gid`默认为101
16. 用相关数据拼一个数组`$user`，通过这个数组创建新用户，并返回`$uid`
17. 再通过`$uid`读取出完整的用户信息`$user`
18. 更新session
19. 更新用户`token`



#### 退出时进入

1. 清空用户`token`



#### 重置密码的第1步进入

1. 从全局配置`$conf`中读取是否开启了密码找回功能
2. 此时的`$method`为`GET`
3. 编译并加载`view/htm/user_resetpw.htm`
4. 用户输入邮箱找回
5. 从参数中获取`$email`
6. 从参数中获取`$code`
7. 验证`$email`和`$code`



#### 重置密码的第3步进入

1. 通过`$email`读取出`$_user`
2. 此时`$method`为`GET`
3. 编译并加载`view/htm/user_resetpw_complete.htm`
4. 此时`$method`为`POST`
5. 从参数中获取`$password`
6. 使用`$user`的`salt`对密码md5，并更新密码



#### 发送验证码进入



#### 同步登录进入







### <span id="route_my.php">route_my.php</span>

点击自己进入

1. 从参数中获取`$action`
2. 通过`$uid`读取出用户信息`$user`
3. 如果`$action`是数字的话，则将它置空

#### `$action`为空

默认进入，展示用户基本信息

1. 编译并加载`view/htm/my.htm`



#### `$action`为`password`

点击密码进入修改密码界面

1. 此时的`$method`为`GET`
2. 编译并加载`view/htm/my_password.htm`
3. 填写新旧密码后
4. 此时的`$method`为`POST`
5. 通过参数获取旧密码`$password_old`
6. 通过参数获取新密码`$password_new`
7. 通过参数获取重复的新密码`$password_new_repeat`
8. 判断两次的新密码是否相同
9. 判断`$password_old`是否是原来的密码
10. 更新新密码



#### `$action`为`thread`

点击`论坛帖子`进入

1. 参过参数获取要展示第几页`$page`
2. 默认页大小`$pagesize`为20
3. 从`$user`中获取用户的总发帖数`$totalnum`
4. 进行分页
5. 从数据库中读取出`$page`要展示的帖子列表`$threadlist`
6. 编译并加载`view/htm/my_thread.htm`



#### `$action`为`avatar`

点击`头像`进入

1. 此时的`$method`为`GET`
2. 编译并加载`view/htm/my_avatar.htm`
3. 从参数中获取头像宽度`$width`
4. 从参数中获取头像高度`$height`
5. 从参数中获取头像数据`$data`
6. 对`$data`进行`base64`编码
7. 判断头像大小



### <span id="route_attach.php">route_attach.php</span>







### <span id="route_post.php">route_post.php</span>

用户发帖进入

1. 通过参数获取`$action`
2. 检查用户是否登录

#### create

跟帖有两种模式：

- 直接跟帖[1-21]
- 高级回复[22-25]

1. 通过参数获取主题id`$tid`
2. 通过参数获取`$quick`
3. 通过参数获取引用id`quotepid`
4. 通过`$tid`获取主题信息`$thread`
5. 通过`$thread`获取版块id`$fid`
6. 通过`$fid`获取版块信息`$forum`
7. 通过`$fid`和`$gid`判断用户是否有权限在此版块回复跟帖
8. 判断此主题是否已经被关闭
9. 通过参数获取跟帖内容`$message`
10. 通过参数获取`$doctype`
11. 如果此主题是置顶主题，则清除置顶缓存
12. 通过参数获取引用帖子id`$quotepid`
13. 通过`$quotepid`找到这个跟帖信息`$quotepost`
14. 构造一个跟帖数组`$post`
15. 创建跟帖并返回跟帖id`$pid`
16. 通过`$pid`获取到跟帖信息`$post`
17. 修改`$post`最下层，其实就是跟帖窗口位置
18. 判断用户是否有权跟帖`$allowpost`
19. 判断用户是否有权修改`$allowupdate`
20. 判断用户是否有权删除`$allowdelete`
21. 编译并加载`view/htm/post_list.inc.htm`
22. 重复1-8步骤
23. 此时`$method`为`GET`
24. 编译并加载`view/htm/post.htm`页面进入高级回复
25. 重复9-20步骤



#### update

修改帖子内容

1. 通过参数获取跟帖id`$pid`
2. 通过`$pid`获取跟帖信息`$post`
3. 通过`$post`获取主题id`$tid`
4. 通过`$tid`获取主题信息`$thread`
5. 通过主题信息`$thread`获取版块id`$fid`
6. 通过`$fid`获取版块信息`$forum`
7. 跟帖是否是第一个跟帖`$isfirst`
8. 判断用户是否有权跟帖
9. 判断用户是否有权修改跟帖`$allowupdate`
10. 如果无权修改并且帖子信息中也有属性不允许修改或主题已经关闭，就不能修改了
11. 此时`$method`为`GET`
12. 对所有的版块列表进行过滤`$forumlist_allowthread`
13. 对`$forumlist_allowthread`中的`fid`和`name`转成数组
14. 如果当前用户不是跟帖的所有者，会对跟帖的内容进行安全处理
15. 如果在跟帖信息中发现跟帖中有附件，则通过`$pid`查找出`$attachlist`、`$imagelist`、`$filelist`
16. 编译并加载`view/htm/post.htm`，进入修改界面
17. 此时`$method`为`POST`
18. 通过参数获取跟帖主题`$subject`
19. 通过参数获取跟帖内容`$message`
20. 通过参数获取`$doctype`
21. 如果跟帖是第一条，那说明就是改的主题内容
22. 那就有可能会修改版块了，通过参数获取新的版块id`$newfid`
23. 通过`$newfid`读取出版块信息`$forum`
24. 如果修改了版块的话，就需要检查用户是否可以在此版块发帖
25. 如果修改了标题的话，就需要更新主题标题
26. 向数据库更新标题及版块变动
27. 向数据库更新帖子内容



#### delete

删除帖子

1. 通过参数获取跟帖id`$pid`
2. 通过`$pid`获取跟帖信息`$post`
3. 通过`$post`获取主题id`$tid`
4. 通过`$tid`获取主题信息`$thread`
5. 通过`$thread`获取版块id`$fid`
6. 通过`$fid`获取版块信息`$forum`
7. 是否是第一个跟帖`$isfirst`
8. 判断用户是否有权发帖
9. 判断用户是否有权删帖
10. 如果是第一个跟帖，则删除整个帖子
11. 如果不是第一个跟帖，则只删除这个跟帖



### <span id="route_mod.php">route_mod.php</span>

管理员对帖子进行`删除`、`移动`、`置顶`、`关闭`等操作时进入

1. 获取管理操作`$action`

#### top置顶

1. 此时`$method`为`GET`
2. 编译并加载`view/htm/mod_top.htm`，展示置顶选项
3. 此时`$method`为`POST`
4. 获取置顶级别`$top`
5. 获取要置顶的主题id数组`$tidarr`
6. 通过`$tidarr`获取主题列表`$threadlist`
7. 遍历操作`$threadlist`中的子集`$thread`
8. 获取版块id`$fid`
9. 获取主题id`$tid`
10. 判断用户级别
11. 判断用户组是否有权限置顶
12. 执行置顶操作
13. 记录操作日志



#### close关闭

1. 此时`$method`为`GET`
2. 编译并加载`view/htm/mod_close.htm`，展示关闭选项
3. 此时`$method`为`POST`
4. 获取关闭选项`$close`
5. 获取要关闭的主题id数组`$tidarr`
6. 通过`$tidarr`获取主题列表`$threadlist`
7. 遍历操作`$threadlist`中的子集`$thread`
8. 获取版块id`$fid`
9. 获取主题id`$tid`
10. 判断用户级别
11. 执行置顶操作
12. 记录操作日志



#### delete删除

1. 此时`$method`为`GET`
2. 编译并加载`view/htm/mod_delete.htm`，展示删除选项
3. 此时`$method`为`POST`
4. 获取要删除的主题id数组`$tidarr`
5. 通过`$tidarr`获取主题列表`$threadlist`
6. 遍历操作`$threadlist`中的子集`$thread`
7. 获取版块id`$fid`
8. 获取主题id`$tid`
9. 判断用户是否有权限删除
10. 执行置顶操作
11. 记录操作日志



#### move移动

1. 此时`$method`为`GET`
2. 编译并加载`view/htm/mod_move.htm`，展示关闭选项
3. 此时`$method`为`POST`
4. 获取要移动的主题id数组`$tidarr`
5. 通过`$tidarr`获取主题列表`$threadlist`
6. 遍历操作`$threadlist`中的子集`$thread`
7. 获取版块id`$fid`
8. 获取主题id`$tid`
9. 判断用户是否有权限移动
10. 执行置顶操作
11. 记录操作日志



#### deleteuser删除用户

管理人员点击用户信息，有`删除用户`按钮

1. 通过参数获取用户id`$_uid`
2. 判断是否有权限删除用户
3. 通过`$_uid`读取出用户信息`$u`
4. 如果用户组小于6 说明是管理组，不能删除
5. 删除用户





### <span id="route_browser.php">route_browser.php</span>

此论坛不支持太低的浏览器版本，低版本访问量提示此页，有下载高版本浏览器链接。



## admin/index.inc.php

基本上和普通用户的`index.inc.php`相同

- [route/forum.php](#admin_route_forum.php)
- [route/group.php](#admin_route_group.php)
- [route/index.php](#admin_route_index.php)
- [route/other.php](#admin_route_other.php)
- [route/plugin.php](#admin_route_plugin.php)
- [route/setting.php](#admin_route_setting.php)
- [route/thread.php](#admin_route_thread.php)
- [route/user.php](#admin_route_user.php)



### <span id="admin_route_forum.php">route/forum.php</span>

点击`版块`进入

1. 通过参数获取操作类型`$action`

#### list

默认情况下为`list`操作

1. 此时的`$method`为`GET`
2. 编译并加载`view/htm/forum_list.htm`页面

不点击版块后面的`编辑`、`删除`按钮，对版块进行修改，点击确认后

1. 此时的`$method`为`POST`
2. 通过参数获取版块id数组`$fidarr`
3. 通过参数获取版块名数组`$namearr`
4. 通过参数获取版块排序`$rankarr`
5. 通过参数获取版块icon`$iconarr`
6. 遍历`$fidarr`，以其中的一个版块为例：
7. 判断版块`$k`是否在`$forumlist`中，如果不存在，说明是新添加的版块，调用`forum_create`创建版块
8. 如果不存在，则更新版块信息
9. 如果版块icon存在的话，则更新icon
10. 通过`$fidarr`和原版块列表`$forumlist`进行对比，找出不同的版块数组`$deletearr`，说明这是需要删除的版块
11. 清空已经删除版块的缓存





#### update

点击版块后面的`编辑`

1. 通过参数获取版块id`$_fid`
2. 通过`$_fid`读取出版块信息`$_forum`
3. 此时的`$method`为`GET`
4. 通过`$_fid`获取访问权限列表`$accesslist`
5. 如果访问权限列表，则通过`$grouplist`对`$accesslist`赋值

修改内容后

1. 此时的`$methode`为`POST`
2. 通过参数获取版块名`$name`
3. 通过参数获取版块排序`$rank`
4. 通过参数获取版块详情`$brief`
5. 通过参数获取版块公告`$announcement`
6. 通过参数获取版块版主`$modnames`
7. 通过参数获取版块是否开启权限`$accesson`
8. 通过`$modnames`获取版主uid`$moduids`
9. 对版块信息进行更新
10. 如果开启了版块权限，则更新版块权限





#### delete

点版块后面的`删除`

1. 通过参数获取版块id`$_fid`
2. 通过`$_fid`读取版块信息`$_forum`
3. 如果`$_fid`是系统分组，则不允许删除
4. 通过`$_fid`读取出此版块的主题列表`$threadlist`
5. 如果主题列表不为空，则不允许删除版块，需要先删除主题
6. 通过`$_fid`获取子版块`$sublist`
7. 如果`$sublist`不为空，则不允许删除，需要先删除子版块
8. 删除版块



### <span id="admin_route_group.php">route/group.php</span>

#### list

点击`用户`->`用户组`

1. 默认显示所有用户组
2. 通过参数获取`$action`，此时的`$action`为`list`
3. 此时的`$method`为`GET`
4. 获取系统用户组`$system_group`，这些用户组id为`0，1，2，3，4，5，6，7，101`
5. 获取用户组最大id`$maxgid`
6. 编译并加载`view/htm/group_list.htm`
7. 页面展示用户组信息，对用户组信息进行修改或删除用户组，点击确定
8. 此时的`$method`为`POST`
9. 通过参数获取用户组id数组`$gidarr`
10. 通过参数获取用户组名数组`$namearr`
11. 通过参数获取起始积分数组`$creditsfromarr`
12. 通过参数获取结束积分数组`$creditstoarr`
13. 遍历`$gidarr`，以其中的一项为例
14. 通过`gidarr`中的下标在其他数组中找出对应的数据
15. 如果在全局组列表`$grouplist`没有找到`$k`，则说明是新创建的组
16. 否则进行更新数据
17. 通过`$grouplist`和`$gidarr`找出不同的地方，这说明是删除的组`$deletearr`
18. 对`$deletearr`进行删除并清空删除组缓存

#### update

通过点击组后面的`编辑`进入

1. 通过参数获取组id`$_gid`
2. 通过`$_gid`获取用户组信息`$_group`
3. 此时的`$method`为`GET`
4. 编译并加载`view/htm/group_update.htm`
5. 展示用户组详细信息
6. 通过参数获取用户组名`$name`
7. 通过参数获取起始积分`$creditsfrom`
8. 通过参数获取结束积分`$creditsto`
9. 通过参数获取权限允许看帖`$allowread`
10. 通过参数获取权限发主题`$allowthread`
11. 通过参数获取权限回帖`$allowpost`
12. 通过参数获取权限上传`$allowattach`
13. 通过参数获取权限下载`$allowdown`
14. 将以上参数拼成一个数组
15. 如果用户组id是`1 -- 5`，则还需要获取以下权限
16. 通过参数获取权限置顶`$allowtop`
17. 通过参数获取权限编辑`$allowupdate`
18. 通过参数获取权限删除`$allowdelete`
19. 通过参数获取权限移动`$allowmove`
20. 通过参数获取权限禁止用户`$allowbanuser`
21. 通过参数获取权限删除用户`$allowdeleteuser`
22. 通过参数获取权限查看用户信息`$allowviewip`
23. 把这些参数也添加到数组中去
24. 使用这个数组对用户组进行更新



### <span id="admin_route_index.php">route/index.php</span>

登录时进入此代码



### <span id="admin_route_other.php">route/other.php</span>

点击`其他`

清理缓存功能。



### <span id="admin_route_plugin.php">route/plugin.php</span>

点击`插件`



### <span id="admin_route_setting.php">route/setting.php</span>

点击`设置`

1. 通过参数获取`$action`

#### base

默认进入此界面

1. 此时的`$action`为`base`
2. 此时的`$method`为`GET`
3. 从全局配置`$conf`中读取信息
4. 编译并加载`view/htm/setting_base.htm`
5. 修改基本设置内容，点击确定
6. 此时的`$method`为`POST`
7. 通过参数获取站点名称`$sitename`、站点介绍`$sitebrief`、站点访问限制`$runlevel`、开启用户注册`$user_create_on`、开启注册邮箱验证`$user_create_email_on`、开启找回密码`$user_resetpw_on`、语言`$_lang`
8. 对全局配置进行更新



#### smtp

点击`SMTP设置`进入此页面

1. 此时的`$action`为`smtp`
2. 此时的`$method`为`GET`
3. 编译并加载`view/htm/setting_smtp.htm`
4. 通过参数获取`$email`、`$host`、`$port`、`$user`、`$pass`
5. 拼成一个数组更新smtp设置



### <span id="admin_route_thread.php">route/thread.php</span>

点击`主题`

#### list

1. 默认页面为此页面，`$action`为`list`
2. 获取所有的主题数`$threads`
3. 计算出总页数`$totalpage`
4. 编译并加载`view/htm/thread_list.htm`



#### scan

1. 填写搜索条件
2. 通过`_SESSION`获取`$queueid`
3. 通过参数获取用户id`$_uid`
4. 如果`$_uid`不是数字，则把它当成用户名搜索出用户信息`$_user`和`$_uid`
5. 通过参数获取版块id`$fid`
6. 填充条件数组`$cond`
7. 通过`$fid`获取所有的主题列表`$threadlist`
8. 遍历`$threadlist`中的每一项，看是否符合条件



#### found

通过`点击查看`显示搜索结果

1. 通过`_SESSION`获取`$queueid`
2. 获取当前要显示的页`$page`
3. 通过`$queueid`获取队列中的数量`$total`
4. 对数据进行分页
5. 编译并加载`view/htm/thread_found.htm`



#### operation

通过点击下面的`关闭`、`打开`、`删除`进行操作

1. 通过`_SESSION`获取`$queueid`
2. 通过参数获取操作类型`$op`
3. 将队列中的数据一个接一个的弹出`$tid`
4. 根据`$op`的类型对`$tid`进行对应的操作



### <span id="admin_route_user.php">route/user.php</span>

点击用户，默认展示`用户列表`

1. 通过参数获取`$action`

#### list

1. 设置每页显示数量`$pagesize`
2. 通过参数获取搜索类型`$srchtype`
3. 通过参数获取关键词`$keyword`
4. 通过参数获取显示第几页`$page`
5. 判断`$srchtype`是否是允许的几种方式
6. 通过`$cond`获取出用户数量`$n`
7. 通过查找条件`$cond`查找出用户列表`$userlist`
8. 根据`$n`进行分页
9. 编译并加载`view/htm/user_list.htm`



#### create

点击`用户`->`创建用户`

1. 此时的`$method`为`GET`
2. 编译并加载`view/htm/user_create.htm`
3. 输入`email`、`用户名`、`密码`、`用户组`，并确定
4. 此时的`$method`为`POST`
5. 通过参数获取`$email`、`$username`、`$password`、`$_gid`
6. 通过`$email`和`$username`查询数据库，是否已经注册
7. 拼一个数组进行用户创建



#### update

点击用户后面的`编辑`操作

1. 通过参数获取用户id`$_uid`
2. 此时的`$method`为`GET`
3. 编译并加载`view/htm/user_update.htm`
4. 修改内容
5. 通过参数获取`$email`、`$username`、`$password`、`$_gid`
6. 通过`$_uid`获取出原始的用户数据`$old`
7. 通过更新的数据和原始数据`old`对比出变化的部分`$update`
8. 对变化部分`$update`进行更新



#### delete

1. 通过参数获取要删除用户的id`$_uid`
2. 通过`$_uid`读取出用户信息`$_user`
3. 通过`$_uid`删除用户
