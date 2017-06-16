# Полезные шаблоны конфигов для Nginx

Переведено и дополнено на основе репозитория [nginx-conf](https://github.com/lebinh/nginx-conf) от [@lebinh](https://github.com/lebinh)

## Содержание
- [Команды Nginx](#Команды-nginx)
- [Блок location на PHP](#блок-location-на-php)
- [Перенаправления](#Перенаправления)
    - [Перенаправление на www](#Перенаправление-на-www)
    - [Перенаправление на no-www](#Перенаправление-на-no-www)
    - [Перенаправление на HTTPS](#Перенаправление-на-https)
    - [Слеш в конце адресной строки](#Слеш-в-конце-адресной-строки)
    - [Перенаправление со страницы на страницу](#Перенаправление-со-страницы-на-страницу)
    - [Перенаправление с сайта на сайт](#Перенаправление-с-сайта-на-сайт)
    - [Перенаправление на определенный путь в URI](#Перенаправление-на-определенный-путь-в-uri)
- [Производительность](#Производительность)
    - [Кэширование](#Кэширование)
    - [Gzip сжатие](#gzip-сжатие)
    - [Кэш файлов](#Кэш-файлов)
    - [SSL Кэш](#ssl-кэш)
    - [Поддержка Upstream](#Поддержка-upstream)
- [Мониторинг](#Мониторинг)
- [Безопасность](#Безопасность)
    - [Активация базовой аунтификации](#Активация-базовой-аунтификации)
    - [Открыть только локальный доступ](#Открыть-только-локальный-доступ)
    - [Защита SSL настроек](#Защита-ssl-настроек)
- [Прочее](#Прочее)
    - [Подзапросы после завершения](#Подзапросы-после-завершения)
    - [Распределение ресурсов между источниками](#Распределение-ресурсов-между-источниками)
- [Источники](#Источники)

## Команды Nginx
Основные команды для выполнения базовый операций во время работы Nginx.

* `nginx -V` - проверить версию Nginx, его скомпилированные параметры конфигурации и установленные модули.
* `nginx -t` - протестировать конфигурационный файл и проверить его расположение.
* `nginx -s reload` - перезапустить конфигурационный файл без перезагрузки Nginx.

[⬆ Наверх](#Содержание)
## Блок location на PHP
Простой шаблон для быстрой и легкой установки PHP, FPM или CGI на ваш сайт.
```nginx
location ~ \.php$ {
  try_files $uri =404;
  client_max_body_size 64m;
  client_body_buffer_size 128k;
  include fastcgi_params;
  fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  fastcgi_pass unix:/path/to/php.sock;
}
```
[⬆ Наверх](#Содержание)
## Перенаправления
### Перенаправление на www
[Корректный способ](http://nginx.org/en/docs/http/converting_rewrite_rules.html) определить удаленный сервер по домену без *www* и перенаправить его c *www*:
```nginx
server {
  listen 80;
  server_name example.org;
  return 301 $scheme://www.example.org$request_uri;
}

server {
  listen 80;
  server_name www.example.org;
}
```
Это будет нормально работать для HTTPS, если используется в соответствующем блоке server{}, где идёт прослушивание 443 порта.
[⬆ Наверх](#Содержание)
### Перенаправление на no-www
Корректный способ определить удаленный сервер по домену c *www* и перенаправить его без *www*:
```nginx
server {
  listen 80;
  server_name example.org;
}

server {
  listen 80;
  server_name www.example.org;
  return 301 $scheme://example.org$request_uri;
}
```
Это будет нормально работать для HTTPS, если используется в соответствующем блоке server{}, где идёт прослушивание 443 порта.
[⬆ Наверх](#Содержание)
### Перенаправление на HTTPS
Способ для переадресации с HTTP на HTTPS:  
```nginx
server {
  listen 80;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl;

  # let the browsers know that we only accept HTTPS
  add_header Strict-Transport-Security max-age=2592000;
}
```
[⬆ Наверх](#Содержание)
### Слеш в конце адресной строки
Данная строка добавляет слэш `/` в конце каждого URL, только в случае, если в URL нет точки или параметров. То есть после *example.com/index.php* или *example.com/do?some=123* слэш не поставится.  
```nginx
rewrite ^([^.\?]*[^/])$ $1/ permanent;
```
[⬆ Наверх](#Содержание)
### Перенаправление со страницы на страницу
```nginx
location = /oldpage.html {
    return 301 http://example.org/newpage.html;
  }
```
[⬆ Наверх](#Содержание)
### Перенаправление с сайта на сайт
```nginx
server {
  server_name old-site.com;
  return 301 $scheme://new-site.com$request_uri;
}
```
[⬆ Наверх](#Содержание)
### Перенаправление на определенный путь в URI
```nginx
location /old-site {
  rewrite ^/old-site/(.*) http://example.org/new-site/$1 permanent;
}
```
[⬆ Наверх](#Содержание)
## Производительность
### Кэширование
Навсегда разрешить браузерам кэшировать статические содержимое. Nginx установит оба заголовка: Expires и Cache-Control.
```nginx
location /static {
  root /data;
  expires max;
}
```
Запретить кэширование браузерам (например для отслеживания запросов) можно следующим образом:
```nginx
location = /empty.gif {
  empty_gif;
  expires -1;
}
```
[⬆ Наверх](#Содержание)
### Gzip сжатие
```nginx
gzip  on;
gzip_buffers 16 8k;
gzip_comp_level 6;
gzip_http_version 1.1;
gzip_min_length 256;
gzip_proxied any;
gzip_vary on;
gzip_types
  text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
  text/javascript application/javascript application/x-javascript
  text/x-json application/json application/x-web-app-manifest+json
  text/css text/plain text/x-component
  font/opentype application/x-font-ttf application/vnd.ms-fontobject
  image/x-icon;
gzip_disable "msie6";
```
[⬆ Наверх](#Содержание)
### Кэш файлов
Если у вас кешируется большое количество статических файлов через Nginx, то кэширование метаданных этих файлов позволит сэкономить время задержки.
```nginx
open_file_cache max=1000 inactive=20s;
open_file_cache_valid 30s;
open_file_cache_min_uses 2;
open_file_cache_errors on;
```
[⬆ Наверх](#Содержание)
### SSL кэш
Подключение SSL кэширования позволит возобновлять SSL сессии и сократить время к следующим обращениям к SSL/TLS протоколу.
```nginx
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
```
[⬆ Наверх](#Содержание)
### Поддержка Upstream
Активация кеширования c использованием Upstream подключений:
```nginx
upstream backend {
  server 127.0.0.1:8080;
  keepalive 32;
}

server {

  location /api/ {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
  }
}
```
[⬆ Наверх](#Содержание)
### Мониторинг
По умолчанию [Stub Status](http://nginx.org/ru/docs/http/ngx_http_stub_status_module.html) модуль не собирается, его сборку необходимо разрешить с помощью конфигурационного параметра —with-http_stub_status_module и активировать с помощью:
```nginx
location /status {
  stub_status on;
  access_log off;
}
```
Данная настройка позволит вам получать статус в обычном текстовом формате по общему количеству запросов и клиентским подключениям (принятым, обработанным, активным).

Более информативный статус от Nginx можно получить с помощью [Luameter](https://luameter.com/), который несколько сложнее в установке и требует наличия Nginx Lua модуля. Это предоставит следующие метрики по различным конфигурационным группам в формате JSON:

* Общее количество запросов/ответов.
* Общее количество ответов сгруппирированных по статус кодам: 1xx, 2xx, 3xx, 4xx, 5xx.
* Общее количество байт принятых/отправленных клиенту.
* Промежуточные отрезки времени для оценки минимума, максимума, медианы, задержек и тд.
* Среднестатистическое количество запросов для простоты мониторинга и составления прогнозов по нагрузке.
* [И прочее...](https://luameter.com/metrics)

[Пример дашборда от Luameter](https://luameter.com/demo).

Также для сбора статистики отлично подходит [ngxtop](https://github.com/lebinh/ngxtop).

[⬆ Наверх](#Содержание)
## Безопасность
### Активация базовой аунтификации
Для начала вам потребуется создать пароль и сохранить его в обычной текстовом файле:
```
имя:пароль
```
Затем установить найтройки для server/location блока, который необходимо защитить:
```nginx
auth_basic "This is Protected";
auth_basic_user_file /path/to/password-file;
```
[⬆ Наверх](#Содержание)
### Открыть только локальный доступ
```nginx
location /local {
  allow 127.0.0.1;
  deny all;
}
```
[⬆ Наверх](#Содержание)
### Защита SSL настроек
* Отключить SSLv3, если он включен по умолчанию. Это предотвратит [POODLE SSL Attack](http://nginx.com/blog/nginx-poodle-ssl/).
* Шифры, которые наилучшим образом обеспечат защиту. [Mozilla Server Side TLS and Nginx](https://wiki.mozilla.org/Security/Server_Side_TLS#Nginx).
```nginx
# don’t use SSLv3 ref: POODLE CVE-2014-356 - http://nginx.com/blog/nginx-poodle-ssl/
ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;  

# Ciphers set to best allow protection from Beast, while providing forwarding secrecy, as defined by Mozilla (Intermediate Set) - https://wiki.mozilla.org/Security/Server_Side_TLS#Nginx
ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
ssl_prefer_server_ciphers  on;
```
[⬆ Наверх](#Содержание)
## Прочее
### Подзапросы после завершения
Бывают ситуации, когда вам необходимо передать запрос на другой бэкэнд **в дополнении или после его обработки**. Первый случай -  отслеживать количество завершенных загрузок путем вызова API, после того как пользователь скачал файл. Второй случай  -отслеживать запрос, к которому вы бы хотели вернуться как можно быстрее (возможно с пустым .gif) и сделать соответствующие записи в фоновом режиме.  [**post_action**](http://wiki.nginx.org/HttpCoreModule#post_action), который позволяет вам определить подзапрос и будет отклонен по окончанию текущего запроса - является [лучшим решением](http://mailman.nginx.org/pipermail/nginx/2008-April/004524.html) для обоих вариантов.
```nginx
location = /empty.gif {
  empty_gif;
  expires -1;
  post_action @track;
}

location @track {
  internal;
  proxy_pass http://tracking-backend;
}
```
[⬆ Наверх](#Содержание)
### Распределение ресурсов между источниками
Самый простой и наиболее известный способ кросс-доменного запроса на ваш сервер:
```nginx
location ~* \.(eot|ttf|woff) {
  add_header Access-Control-Allow-Origin *;
}
```
[⬆ Наверх](#Содержание)
## Источники
- [Nginx Official Guide](http://nginx.com/resources/admin-guide/)
- [HTML 5 Boilerplate's Sample Nginx Configuration](https://github.com/h5bp/server-configs-nginx)
- [Nginx Pitfalls](http://wiki.nginx.org/Pitfalls)

[⬆ Наверх](#Содержание)
