# Heroku 上使用 Elasticsearch 从入门到放弃

Heroku 不直接提供 elasticsearch，而是通过第三方提供，使用此功能，需要安装组件

目前，heroku 有 3 个组件提供 elasticsearch 服务，分别是：

- [SearchBox](https://elements.heroku.com/addons/searchbox)，**Starting at $0/mo.**
- [Bonsai](https://elements.heroku.com/addons/bonsai)，**Starting at $0/mo.**
- [Elasticsearch](https://elements.heroku.com/addons/foundelasticsearch)，**Starting at $67/mo.**

此次部署并不用于生产环境，且 [Bonsai](https://elements.heroku.com/addons/bonsai) 能提供更大的容量，所以选择免费的 Bonsai 组件

但目前 Bonsai 不支持 elasticsearch7 -> [Which Versions Bonsai Supports - bonsai](https://docs.bonsai.io/article/139-which-versions-bonsai-supports)



鉴于目前需求的数据量太小而 elasticsearch 相对来说太重, 且在 heroku 上使用比较麻烦, 如何可以,建议还是在 VPS 上使用

放弃在 heroku 上部署 elasticsearch



## 坑

1. 命令错误

```
Error: Missing required flag: -a, --app APP  app to run command against
```

完整命令

```
heroku addons:create bonsai:[plan] [-a APP_NAME] [--version=X.Y]
```



2. 找不到组件服务或计划

```
Couldn't find either the add-on service or the add-on plan of "bonsai:Sandbox".
```

bonsai 提供的免费计划为 sandbox-6 (区分大小写)，该计划名在文档中没有体现，在 [Bonsai Elasticsearch - Add-ons - Heroku Elements](https://elements.heroku.com/addons/bonsai) 链接的最底部才能发现

所以，应使用如下命令：

```
heroku addons:create bonsai:sandbox-6 -a APP_NAME --version=X.Y
```



3. 账户信息不完整，需添加信用卡信息

```
Please verify your account to install this add-on plan (please enter a
 ▸    credit card) For more information, see
 ▸    https://devcenter.heroku.com/categories/billing Verify now at
 ▸    https://heroku.com/verify
```





## refs

- 有助于理解 heroku 如何提供 elastictsearch 服务
  - [Flask 教程 第十八章：Heroku上的部署_天降攻城狮 - jishuwen(技术文)](https://www.jishuwen.com/d/2URg)
  - [Heroku第三方服务接入指南（一） - lizeyang的专栏 - CSDN博客](https://blog.csdn.net/lizeyang/article/details/40429979)
  - [Heroku第三方服务接入指南（二） - lizeyang的专栏 - CSDN博客](https://blog.csdn.net/lizeyang/article/details/40510271)
  - [Heroku第三方服务接入指南（三） - lizeyang的专栏 - CSDN博客](https://blog.csdn.net/lizeyang/article/details/40510353)
  - [Saleor 33: 集成 - Elasticsearch 搜索 | Jeremy's blog](https://jeremyyin.com/saleor/integrations/33_elasticsearch/)
- 文档
  - [Bonsai Elasticsearch | Heroku Dev Center](https://devcenter.heroku.com/articles/bonsai)