# lumen-api-demo

这是一个比较完整用 lumen 5.3 写的的 REST API 例子，如果你正在做相同的事，那么这个例子或许能帮助你。使用了 `dingo/api` ，jwt 实现登录，功能上很简单，登录，注册，发帖，评论，还写了单元测试。

[![StyleCI](https://styleci.io/repos/44219096/shield)](https://styleci.io/repos/44219096)
[![License](https://img.shields.io/github/license/liyu001989/lumen-api-demo.svg)](LICENSE)

lumen5.1看[这里](https://github.com/liyu001989/lumen-api-demo/tree/5.1)

lumen5.2看[这里](https://github.com/liyu001989/lumen-api-demo/tree/5.2)

升级到 5.3 后单元测试错误，应该是容器的问题，可能是 bug，慢慢解决

有需要随时联系我 

- QQ: 490554191
- email: liyu001989@gmail.com

[ENGLISH README](./EN_README.md)

## USEFUL LINK

读文档很重要，请先仔细读读 dingo/api，jwt，fractal 的文档。

- dingo/api [https://github.com/dingo/api](https://github.com/dingo/api)
- dingo api 中文文档 [dingo-api-wiki-zh](https://github.com/liyu001989/dingo-api-wiki-zh)
- json-web-token(jwt) [https://github.com/tymondesigns/jwt-auth](https://github.com/tymondesigns/jwt-auth)
- transformer [fractal](http://fractal.thephpleague.com/)
- apidoc 生成在线文档 [apidocjs](http://apidocjs.com/)
- rest api 参考规范 [jsonapi.org](http://jsonapi.org/format/)
- api 调试工具 [postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en)
- 参考文章 [http://oomusou.io/laravel/laravel-architecture](http://oomusou.io/laravel/laravel-architecture/)
- 该项目api文档 [http://lumen.lyyw.info/apidoc](https://lumen.lyyw.info/apidoc)
- php lint [phplint](https://github.com/overtrue/phplint)


## USAGE
```
先了解一下基本的[安装流程](https://laravel-china.org/docs/5.3/installation)，大致一样。

$ git clone git@github.com:liyu001989/lumen-api-demo.git
$ composer install
设置 `storage` 目录必须让服务器有写入权限。
$ cp .env.example .env
$ vim .env
        DB_*
            填写数据库相关配置 your database configuration
	    JWT_SECRET
            php artisan jwt:secret
	    APP_KEY
            lumen 取消了key:generate 所以随便找个地方生成一下吧
            md5(uniqid())，str_random(32) 之类的，或者用jwt:secret生成两个copy一下

$ php artisan migrate
$ php artisan db:seed (默认添加了10个用户，50篇帖子, 100调评论)
$ 生成文档我是这么写的 apidoc -i App/Http/Controller/Api/v1 -o public/apidoc
$ api文档在public/apidoc里面, 也可以看上面的 `在线api文档`
```
如果访问一直不对，可以进入public 目录执行 `php -S localhost:8000 -t public`，然后尝试调用几个接口，从而确定是否为web服务器的配置问题。


## REST API DESIGN

大概举个例子说明一下 rest api 吧

github 的 api 真的很有参考价值 [github-rest-api](https://developer.github.com/v3/)

        例子： 用户，帖子，评论
        get    /api/posts              	 帖子列表
        post   /api/posts              	 创建帖子
        get    /api/posts/5            	 id为5的帖子详情
        put    /api/posts/5            	 替换帖子5的全部信息
        patch  /api/posts/5            	 修改部分帖子5的信息
        delete /api/posts/5            	 删除帖子5
        get    /api/posts/5/comments     帖子5的评论列表
        post   /api/posts/5/comments     添加评论
        get    /api/posts/5/comments/8   id为5的帖子的id为8的评论详情
        put    /api/posts/5/comments/8   替换帖子评论内容
        patch  /api/posts/5/comments/8   部分更新帖子评论
        delete /api/posts/5/comments/8   删除某个评论
        get    /api/users/4/posts        id为4的用户的帖子列表
        get    /api/user/posts           当前用户的帖子列表

## 问题总结

<details>
  <summary>lumen 5.1 升级到 5.2</summary>

- fix compose.json

        "laravel/lumen-framework": "5.2.*",
        "vlucas/phpdotenv": "~2.2" // 这是个坑啊
      
        将5.2的composer.json拿过来对比一下吧

- fix bootstrap/app.php
- Illuminate\Contracts\Foundation\Application 改为了Laravel\Lumen\Application，所以修改一下app\providers\EventServiceProvider.php
- 可以从 5.2 的项目中，把 Middleware cp 过来
</details>


<details>
  <summary>jwt 使用</summary>

lumen 5.2 取消了session，没有了 auth 的实例，所以使用jwt的时候需要配置一下，注意 config/auth.php 中的配置，而且 user 的 model 需要实现 `Tymon\JWTAuth\Contracts\JWTSubject`;
</details>

<details>
  <summary>使用mail</summary>

  写了个例子，注册之后给用户发送邮件, 可以参考一下。

- composer 加 illuminate/mail 和 guzzlehttp/guzzle 这两个库
- 在 bootstrap/app.php 或者 provider 中注册 mail 服务
- 增加配置 mail 和 services, 从 laravel 项目里面 cp 过来
- 在 env 中增加 `MAIL_DRIVER`，账户，密码等配置
</details>

<details>
  <summary>transformer 的正确使用</summary>

  transformer 是个数据转换层，帮助你格式化资源。还可以帮助你处理资源之间的引用关系。

  试着体会一下以下几个url的也许就明白了

  - [http://lumen.lyyw.info/api/posts](http://lumen.lyyw.info/api/posts)  所有帖子列表
  - [http://lumen.lyyw.info/api/posts?include=user](http://lumen.lyyw.info/api/posts?include=user) 所有帖子列表及发帖用户
  - [http://lumen.lyyw.info/api/posts?include=user,comments](http://lumen.lyyw.info/api/posts?include=user,comments) 帖子列表及发帖的用户和发帖的评论
  - [http://lumen.lyyw.info/api/posts?include=user,comments:limit(1)](http://lumen.lyyw.info/api/posts?include=user,comments:limit(1)) 帖子列表及发帖的用户和发帖的1条评论
  - [http://lumen.lyyw.info/api/posts?include=user,comments.user](http://lumen.lyyw.info/api/posts?include=user,comments.user) 帖子列表及发帖的用户和发帖的评论，及评论的用户信息
  - [http://lumen.lyyw.info/api/posts?include=user,comments:limit(1),comments.user](http://lumen.lyyw.info/api/posts?include=user,comments:limit(1),comments.user)  帖子列表及发帖的用户和发帖的1条评论，及评论的用户信息，及评论的用户信息

  
  是不是很强大，我们只需要提供资源，及资源之间的引用关系，省了多少事
 
</details>

<details>
  <summary>transformer 如何自定义格式化资源</summary>

dingo/api 使用了 [Fractal](http://fractal.thephpleague.com/) 做数据转换，fractal 提供了3种基础的序列化格式，Array，DataArray，JsonApi，在这里有详细的说明 [http://fractal.thephpleague.com/serializers/](http://fractal.thephpleague.com/serializers/)。DataArray 是默认的，也就是所有资源一定有data和meta。当然也可以按下面这样自定义：

        只需要在 bootstrap/app.php 中设置 serializer 就行了。具体见 bootstrap/app.php 有注释
        $app['Dingo\Api\Transformer\Factory']->setAdapter(function ($app) {
            $fractal = new League\Fractal\Manager;
            // 自定义的和fractal提供的
            // $serializer = new League\Fractal\Serializer\JsonApiSerializer();
            $serializer = new League\Fractal\Serializer\ArraySerializer();
            // $serializer = new App\Serializers\NoDataArraySerializer();
            $fractal->setSerializer($serializer);,
            return new Dingo\Api\Transformer\Adapter\Fractal($fractal);
        });

个人认为默认的 DataArray 就很好用了，基本满足了 API 的需求
</details>

<details>
  <summary>repository 的使用</summary>

  为了不造成误解，v1 版本的 api 使用了仓库, v2 版本直接使用的 Eloquent。

  我对 repository 的理解是，它是一层对 orm 的封装，让 model 和 controller 层解耦，controller 只是关心增删该查什么数据，并不关心数据的操作是通过什么完成的，orm也好，DB也好，只要实现接口就好。而且封装了一层，我就可以对一些查询数据方便的进行缓存，而不需要调整 controller，非常方面，清晰。

  上面这些是它的好处，那么当然也有不好的地方，就是对于普通的项目来说，切换 orm，或者抛弃 orm 转为全部使用 DB，这样的场景还是很少的，或者也是很后期优化的时候才会用到。还有就是，当一开始大家对 repository 的概念不清楚的时候，尝尝把大段的业务逻辑放在里面，而原本这些个业务逻辑应该出现在 controller 和 services 中。

  所以一般的项目就直接使用 Eloquent 吧, 不要过度设计，这里只是个例子。使用 ORM 是一件很方面的事情， dingo 的 transform 这一层就是通过 Eloquent 去预加载的。

  例子中我是随便写的，`rinvex/repository` 和 `prettus/l5-repository` 这两个库都不错，大家可以试试
</details>

## TODO
- [ ] 单元测试

## 坑
- [https://github.com/dingo/api/issues/672](https://github.com/dingo/api/issues/672)  `transformer include`
- 如果 .env 的某个值中有空格会报错 log not found。env 中的值有空格需要引号包裹

## License

[MIT license](http://opensource.org/licenses/MIT)
