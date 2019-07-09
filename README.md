# Expedia Technical Exercise

Pulished:

* Landing: <http://exp.tecposter.cn/>
* UI: <http://exp.tecposter.cn/expui.html>
* Contact: <http://exp.tecposter.cn/contact/list>
* API: <http://exp.tecposter.cn/api/contact/>

Sub Project

* exp-server: <https://github.com/zhanjh/exp-server>
* exp-ui: <https://github.com/zhanjh/exp-ui>
* exp-front: <https://github.com/zhanjh/exp-front>

## Deplyment

### Database

Install MariaDB or MySQL first

Build contacts database

```
mysql -uroot -p < data/mysql.sql

> grant all privileges on Expedia.* to '{db-user}'@'localhost' identified by '{db-passwd}';
```


Add key for table `ContactDetail`

```sql
ALTER TABLE `ContactDetail` ADD KEY `UserID` (`UserID`);
```

Create view 'ContactSummary'

```sql
CREATE VIEW `ContactSummary` AS
select c.*, count(d.`UserID`) as DetailCount
from (`Contact` `c` left join `ContactDetail` `d` on(`c`.`UserID` = `d`.`UserID`)) group by `d`.`UserID`
```

### exp-server

clone 

```
git clone https://github.com/zhanjh/exp-server
```

Edit local config `config/config.local.js`

```
/* eslint-env node */
module.exports = {
  mysql: {
    user: '{db-user}',
    password: '{db-password}'
  }
};
```

Install node dependencies

```
yarn install
```

Start API server

```
yarn start
```

### exp-ui

clone

```
git clone https://github.com/zhanjh/exp-ui
```

Edit local setting `setting/node.front.local.js`

```js
/* eslint-env node */

module.exports = {
  front: {
    port: 8007,
  },
  site: {
    static: {
      host: 'exp.tecposter.cn'
    }
  }
};
```

yarn install

```
yarn install
```

Generate compressed css `site/static/ui/dist/css/main.css`

```
yarn run frontRelease
```

### exp-front

clone

```
git clone https://github.com/zhanjh/exp-front
```

Edit local setting

```
// setting/setting.local.js
export const localSetting = {
  apiURL: 'http://exp.tecposter.cn/api'
};

// setting/setting.server.local.js

/* eslint-env node */
module.exports = {
  front: {
    port: 8197
  },
  site: {
    static: {
      host: 'exp.tecposter.cn'
    }
  }
};
```

yarn install

```
yarn install
```

Generate js files

```
yarn run frontRelease
```

### Nginx

```
server {
    listen          80;
    listen          [::]:80;
    server_name exp.tecposter.cn;

    index   index.html;
    root    /home/zjh/work/project/exp/site;

    access_log    /home/zjh/work/project/exp/log/access.log.gz combined gzip;
    error_log     /home/zjh/work/project/exp/log/error.log;

    client_max_body_size 20M;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /ui {
      root    /home/zjh/work/project/exp/exp-ui/site/static;
    }

    location /front {
      root    /home/zjh/work/project/exp/exp-front/site/static;
    }

    location /api {
      proxy_buffering  off;
      proxy_pass       http://127.0.0.1:9198;
      proxy_set_header X-Forwarded-For $remote_addr;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

## Optimize

### UI

* Remove foundation components that not used
	* Foundation for Sites Template: <https://github.com/zurb/foundation-sites-template>

### Server - Nodejs

To research
* [Threads in NodeJs — Performance Optimization](https://medium.com/tech-tajawal/threading-in-nodejs-5d966a3b9858)
* [Using connection pools](https://www.npmjs.com/package/mysql2#using-connection-pools)

### Database

To research

* full text indexing
* ORDER BY Optimization
* [optimize COUNT(*) performance](https://stackoverflow.com/questions/19267507/how-to-optimize-count-performance-on-innodb-by-using-index)

## CI/CD

* Automated testing

## Bugs

* Keyword is not clear from the const object in js
	* Search -> show detail -> list -> pagination
