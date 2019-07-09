# Expedia Technical Exercise

## Deplyment

### Database

Install MariaDB or MySQL first

```
mysql -uroot -p < data/mysql.sql

> grant all privileges on Expedia.* to 'exp'@'localhost' identified by '{db-passwd}';
```
