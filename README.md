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
heroku buildpacks:set heroku/php
```
