## 上面代码

1. 登录你的 Heroku 账号

```shell script
heroku login -i
```

2. 添加 SSH Key 到 Heroku 上

```shell script
heroku keys:add
```

3. 创建一个 Heroku App

```shell script
heroku create
```

4. 配置 Procfile 文件

```shell script
echo web: vendor/bin/heroku-php-apache2 public/ > Procfile
git add -A
git commit -m "Procfile for Heroku"
git push
```

5. 声明 buildpack

Heroku 平台支持多种语言，在进行应用部署时，Heroku 会自动检查应用的代码是用什么语言写的，然后再接着执行一系列针对该语言的操作来准备好程序运行环境。Laravel 应用默认会包含 package.json 文件，但当 Heroku 检查到该文件时，它会认为此应用是用 Node.js 写的，因此我们需要对应用的 buildpack 进行声明，告诉 Heroku 说我们的应用是用 PHP 写的

```shell script
heroku buildpacks:set heroku/php
```

6. 生成 App Key

```shell script
php artisan key:generate --show
base64:ZOmgffT0TQi+0mVRQ583uan1yJtoX65SeeU7IjmlQzY=
```

将生成的 App Key （如以上 base64:ta1aE+E8kuyDFlURbUrHEtL4HY71WtoffyNgUKldMWw= ） 替换掉下面命令的 <your_app_key> 并运行命令

```shell script
heroku config:set APP_KEY=<your_app_key>
```

如：

```shell script
heroku config:set APP_KEY=base64:ZOmgffT0TQi+0mVRQ583uan1yJtoX65SeeU7IjmlQzY=
```

7. 配置基本完成，将代码推送到 Heroku 上

```shell script
git push heroku master
```

8. 使用以下命令查看 Heroku 站点地址

```shell script
heroku domains
=== powerful-ravine-55489 Heroku Domain
powerful-ravine-55489.herokuapp.com
```

注意以上的 dry-bastion-64171 为变量，你的 Heroku 会生成与我不同的地址，这是正常情况。

使用 Heroku 过程中如果出现问题，则可以使用下面命令来输出生产环境上的日志进行排错

```shell script
heroku logs
```

## 统一代码风格

`.editorconfig`

```shell script
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 4
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

[*.yml]
indent_size = 2

[*.{js,html,blade.php,css,scss}]
indent_style = space
indent_size = 2
```