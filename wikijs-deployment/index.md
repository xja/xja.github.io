# Wiki.js 本地部署


## 安装 NodeJS

```ShellSession
root@ubuntu:~# adduser node
root@ubuntu:~# su - node
node@ubuntu:~$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
node@ubuntu:~$ nvm install 16.15.1
node@ubuntu:~$ which node
/home/node/.nvm/versions/node/v16.15.1/bin/node
node@ubuntu:~$ exit
```

## 安装 PostgreSQL

```ShellSession
root@ubuntu:~# apt install postgresql postgresql-contrib
root@ubuntu:~# systemctl start postgresql.service
root@ubuntu:~# su - postgres
postgres@ubuntu:~$ createuser --interactive
Enter name of role to add: wikijs
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) y
Shall the new role be allowed to create more new roles? (y/n) n
postgres@ubuntu:~$ psql
psql (14.3 (Ubuntu 14.3-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# \password wikijs
Enter new password for user "wikijs":
Enter it again:
postgres=# CREATE DATABASE wikijs;
postgres=# GRANT ALL PRIVILEGES ON DATABASE wikijs TO wikijs;
postgres=# \q
postgres@ubuntu:~$ exit
```

## 安装 Wiki.js

### 下载源码

```ShellSession
root@ubuntu:~# adduser wikijs
root@ubuntu:~# usermod -aG node wikijs
root@ubuntu:~# mkdir -p /var/www/wikijs
root@ubuntu:~# chown wikijs:wikijs /var/www/wikijs
root@ubuntu:~# su - wikijs
wikijs@ubuntu:~$ cd /var/www/wikijs
wikijs@ubuntu:/var/www/wikijs$ wget https://github.com/Requarks/wiki/releases/latest/download/wiki-js.tar.gz
wikijs@ubuntu:/var/www/wikijs$ tar xzf wiki-js.tar.gz
```

### 配置 Wiki.js

```ShellSession
wikijs@ubuntu:/var/www/wikijs$ cp config.sample.yml config.yml
wikijs@ubuntu:/var/www/wikijs$ nano config.yml
```

根据实际情况修改配置文件

```text
# PostgreSQL / MySQL / MariaDB / MS SQL Server only:
host: localhost
port: 5432
user: wikijs
pass: wikijsrocks
db: wikijs
# IP address the server should listen to
bindIP: 127.0.0.1
# Log Level
logLevel: warn
```

完成后即可尝试前台启动服务

```ShellSession
wikijs@ubuntu:/var/www/wikijs$ /home/node/.nvm/versions/node/v16.15.1/bin/node server/
```

## 创建后台服务

```ShellSession
wikijs@ubuntu:/var/www/wikijs$ nano wikijs.service
```

粘贴下面内容

```text
[Unit]
Description=Knowledge Base'd on Wiki.js
After=network.target

[Service]
Type=simple
ExecStart=/home/node/.nvm/versions/node/v16.15.1/bin/node /var/www/wikijs/server/
Restart=always

User=wikijs
Environment=NODE_ENV=production
WorkingDirectory=/var/www/wikijs

[Install]
WantedBy=multi-user.target
```

添加自启并启动

```ShellSession
root@ubuntu:~# systemctl enable --now /var/www/wikijs/wikijs.service
```

## 配置 Nginx

```ShellSession
root@ubuntu:~# nano /var/www/wikijs/nginx.conf
```

粘贴下列内容

```text
server {
    listen 80;
    #listen [::]:80;
    server_name wiki.example.com;

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl http2;
    #listen [::]:443 ssl http2;
    server_name wiki.example.com;

    charset UTF-8;
    access_log      /var/log/nginx/wikijs_access.log;
    error_log       /var/log/nginx/wikijs_error.log;

    location / {
        proxy_pass          http://127.0.0.1:3000;
        proxy_http_version  1.1;
        proxy_set_header    Upgrade $http_upgrade;
        proxy_set_header    Connection "upgrade";
        proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto $scheme;
    }

    ssl_certificate /root/.acme.sh/wiki.example.com/wiki.example.com.cer;
    ssl_certificate_key /root/.acme.sh/wiki.example.com/wiki.example.com.key;
}
```

使用 `nginx -t` 测试无误后链接到 nginx 配置目录下

```ShellSession
root@ubuntu:~# ln -s /var/www/wikijs/nginx.conf /etc/nginx/sites-enabled/wikijs.conf
root@ubuntu:~# nginx -s reload
```

完成后打开浏览器进行站点配置

## 中文分词

```ShellSession
root@ubuntu:~# apt install --no-install-recommends gcc make libc6-dev bzip2 apt-transport-https ca-certificates
root@ubuntu:~# apt install postgresql-server-dev-all
root@ubuntu:~# curl -sSkLf http://www.xunsearch.com/scws/down/scws-1.2.3.tar.bz2 | tar xjf - 
root@ubuntu:~# curl -sSkLf https://github.com/amutu/zhparser/archive/master.tar.gz | tar xzf -
root@ubuntu:~# cd scws-1.2.3 && ./configure && make -j$(nproc) install V=0 && cd ..
root@ubuntu:~# cd zhparser-master && make -j$(nproc) install && cd ..
root@ubuntu:~# apt-get purge -y postgresql-server-dev-all
root@ubuntu:~# apt-get autoremove --purge -y
root@ubuntu:~# rm -rf zhparser-master scws-1.2.3
root@ubuntu:~# su - postgres -c psql
postgres=# ALTER USER wikijs WITH SUPERUSER;
postgres=# \q
root@ubuntu:~# su - wikijs -c psql
wikijs=> CREATE EXTENSION pg_trgm;
wikijs=> CREATE EXTENSION zhparser;
wikijs=> CREATE TEXT SEARCH CONFIGURATION pg_catalog.chinese_zh (PARSER = zhparser);
wikijs=> ALTER TEXT SEARCH CONFIGURATION chinese_zh ADD MAPPING FOR n,v,a,i,e,l WITH simple;
wikijs=> ALTER ROLE wikijs SET zhparser.punctuation_ignore = ON;
wikijs=> ALTER ROLE wikijs SET zhparser.multi_short = ON;
wikijs=> \q
root@ubuntu:~# su - postgres -c psql
postgres=# ALTER USER wikijs WITH NOSUPERUSER;
postgres=# \q
```

编辑 `definition.yml` 文件

```ShellSession
root@ubuntu:~# nano /var/www/wikijs/server/modules/search/postgres/definition.yml
```

在 `enum` 下添加 `- chinese_zh`

```yaml
enum:
      ...
      - turkish
      - chinese_zh
```

修改 `wiki_db` 中的表 `searchEngines`，将 `dictLanguage` 的值改为 `chinese_zh`

```ShellSession
root@ubuntu:~# su - wikijs -c psql
```

```sql
UPDATE "public"."searchEngines" SET "isEnabled"='true', "config"='{"dictLanguage":"chinese_zh"}' WHERE  "key"='postgres';
```

## QA

1. 启动报错

    ```text
    error: Package subpath './public/extractFiles' is not defined by "exports" in /var/www/wikijs/node_modules/extract-files/package.json
    ```

    A: 使用低版本的 NodeJS，如本例中的 16.15.1，参见 [error: Package subpath './public/extractFiles' is not defined by "exports" in /home/bot/wiki/node_modules/extract-files/package.json · Discussion #5113 · requarks/wiki](https://github.com/requarks/wiki/discussions/5113)

## 参考

1. [Linux | Wiki.js](https://docs.requarks.io/install/linux)
2. [Install Wiki.js with Node.js, PostgreSQL, and Nginx on Ubuntu 20.04 LTS - Vultr.com](https://www.vultr.com/docs/install-wiki-js-with-node-js-postgresql-and-nginx-on-ubuntu-20-04-lts/)
3. [How To Install PostgreSQL on Ubuntu 20.04 [Quickstart] | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart)
4. [wiki.js 使用 postgres 支持中文全文检索 - 知乎](https://zhuanlan.zhihu.com/p/335359081)
5. [abcfy2/docker_zhparser: A source repo of Postgres Chinese full-test search docker image, based on zhparser. - Github](https://github.com/abcfy2/docker_zhparser)

