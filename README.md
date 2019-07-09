# Expedia Technical Exercise


## Released


* Landing: <http://exp.tecposter.cn/>
* UI: <http://exp.tecposter.cn/expui.html>
* Contact: <http://exp.tecposter.cn/contact/list>

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
