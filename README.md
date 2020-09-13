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

## 在 Heroku 上使用 PostgreSQL

要在 Heroku 上使用 PostgreSQL，我们需要先安装 PostgreSQL 扩展。

```shell script
heroku addons:add heroku-postgresql:hobby-dev
```

如：

```shell script
heroku addons:add heroku-postgresql:hobby-dev
Created postgresql-crystalline-51657 as DATABASE_URL
Use heroku addons:docs heroku-postgresql to view documentation
```

安装完成之后，Heroku 将为我们生成一个唯一的数据库 URL - DATABASE_URL，我们可以通过下面命令查看 Heroku 的所有配置信息：

```shell script
heroku config
```

如：

```shell script
$ heroku config
 ›   Warning: heroku update available from 7.35.0 to 7.42.13.
=== powerful-ravine-55489 Config Vars
APP_KEY:      base64:ZOmgffT0TQi+0mVRQ583uan1yJtoX65SeeU7IjmlQzY=
DATABASE_URL: postgres://urptruhbcidduj:02cc791f7e4a0b2ba5e37169e17d364747aa1c6df1187bebcdc3240bbeb37f69@ec2-54-235-192-146.compute-1.amazonaws.com:5432/ddriccf5u5gmrv
vagrant@homestead:~/Code/weibo$
```

在本地开发中，我们使用了 MySQL 来作为数据库储存，但在 Heroku 环境上我们要改为使用 PostgreSQL 来作为数据库储存。因此在进行数据库设置时，我们需要对当前环境进行判断。如果环境为本地环境，则使用 MySQL 数据库，若为 Heroku 环境，则使用 PostgreSQL 数据库。我们可以通过为 Heroku 新增一个 IS_IN_HEROKU 配置项来判断应用是否运行在 Heroku 上。

```shell script
heroku config:set IS_IN_HEROKU=true
```

一般来说，应用的数据库都在 config/database.php 中进行配置，因此我们需要针对该配置文件，来为不同环境的数据库连接方式定义一个帮助方法，以便根据应用不同的运行环境来指定数据库配置信息，我们需要新建一个 helpers.php 文件并写入以下内容：

app/helpers.php

```shell script
<?php

function get_db_config()
{
    if (getenv('IS_IN_HEROKU')) {
        $url = parse_url(getenv("DATABASE_URL"));

        return $db_config = [
            'connection' => 'pgsql',
            'host' => $url["host"],
            'database'  => substr($url["path"], 1),
            'username'  => $url["user"],
            'password'  => $url["pass"],
        ];
    } else {
        return $db_config = [
            'connection' => env('DB_CONNECTION', 'mysql'),
            'host' => env('DB_HOST', 'localhost'),
            'database'  => env('DB_DATABASE', 'forge'),
            'username'  => env('DB_USERNAME', 'forge'),
            'password'  => env('DB_PASSWORD', ''),
        ];
    }
}
```

可以看到，我们定义了 get_db_config 方法来根据数据库的不同运行环境获取不同的配置信息。通过 Heroku 生成的 DATABASE_URL 包含了一切与数据库相关的配置信息，如主机、用户名、密码、数据库等，因此我们需要使用 parse_url 方法对其进行解析，来获取到指定的值。当运行环境为 Heroku 时，我们使用 DATABASE_URL 提供的数据库配置信息，如果为其它环境，则使用默认的数据库配置信息。

在我们新增 helpers.php 文件之后，还需要在项目根目录下 composer.json 文件中的 autoload 选项里 files 字段加入该文件：

composer.json

```shell script
{
    ...

    "autoload": {
        "psr-4": {
            "App\\": "app/"
        },
        "classmap": [
            "database/seeds",
            "database/factories"
        ],
        "files": [
            "app/helpers.php"
        ]
    }
    ...
}
```

修改保存后运行以下命令进行重新加载文件即可：

```shell script
composer dump-autoload
```

现在，让我们使用刚刚定义好的 get_db_config 方法对数据库进行配置。将数据库配置文件替换为以下内容：

config/database.php

```shell script
<?php

use Illuminate\Support\Str;

$db_config = get_db_config();

return [

    'default' => $db_config['connection'],

    'connections' => [

        'mysql' => [
            'driver' => 'mysql',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'prefix_indexes' => true,
            'strict' => true,
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            ]) : [],
        ],

        'pgsql' => [
            'driver'   => 'pgsql',
            'host'     => $db_config['host'],
            'port'     => env('DB_PORT', '5432'),
            'database' => $db_config['database'],
            'username' => $db_config['username'],
            'password' => $db_config['password'],
            'charset' => 'utf8',
            'prefix' => '',
            'prefix_indexes' => true,
            'schema' => 'public',
            'sslmode' => 'prefer',
        ],

];
```

我们可以使用 heroku run 在 Heroku 运行 Laravel 的指定命令。现在我们需要在 Heroku 上执行迁移，生成用户表，可通过下面命令来完成：

```shell script
heroku run php artisan migrate
```

若提示是否要在生产环境上运行此命令，请输入 yes 或者 y。

如果你要在 Heroku 上重置 PostgreSQL 数据库，可以使用以下命令（知晓即可，你不需要执行）：

```shell script
heroku pg:reset DATABASE
heroku run php artisan migrate
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

## 前端样式美化

```shell script
composer require laravel/ui:^1.0 --dev
```

package.json

```shell script
{
    "private": true,
    "scripts": {
        "dev": "npm run development",
        "development": "cross-env NODE_ENV=development node_modules/webpack/bin/webpack.js --progress --hide-modules --config=node_modules/laravel-mix/setup/webpack.config.js",
        "watch": "npm run development -- --watch",
        "watch-poll": "npm run watch -- --watch-poll",
        "hot": "cross-env NODE_ENV=development node_modules/webpack-dev-server/bin/webpack-dev-server.js --inline --hot --config=node_modules/laravel-mix/setup/webpack.config.js",
        "prod": "npm run production",
        "production": "cross-env NODE_ENV=production node_modules/webpack/bin/webpack.js --no-progress --hide-modules --config=node_modules/laravel-mix/setup/webpack.config.js"
    },
    "devDependencies": {
        "axios": "^0.19",
        "bootstrap": "^4.0.0",
        "cross-env": "^5.1",
        "jquery": "^3.2",
        "laravel-mix": "^4.0.7",
        "lodash": "^4.17.13",
        "popper.js": "^1.12",
        "resolve-url-loader": "^2.3.1",
        "sass": "^1.15.2",
        "sass-loader": "^7.1.0",
        "vue-template-compiler": "^2.6.10"
    }
}
```

```shell script
npm config set registry=https://registry.npm.taobao.org
yarn config set registry 'https://registry.npm.taobao.org'
```

```shell script
yarn install --no-bin-links
yarn add cross-env
```

```shell script
npm run dev
```

```shell script
npm run watch-poll
```

## 添加语言包

```shell script
composer require "overtrue/laravel-lang:~3.0"
```

安装成功后，在 config/app.php 文件中将以下这一行：

```shell script
Illuminate\Translation\TranslationServiceProvider::class,
```

替换为：

```shell script
Overtrue\LaravelLang\TranslationServiceProvider::class,
```

最后，我们还需要将项目语言设置为中文。

config/app.php

```shell script
<?php

return [
    .
    .
    .
    'locale' => 'zh-CN',
    .
    .
    .
];    
```
