---
layout: post
title: "Практичний досвід: Розгортання Traefik у FreeBSD (Reverse Proxy, ACME, mac_portacl та JWT)"
date: 2026-10-30 21:24:30 +0000
tags: ["freebsd", "jwt", "nginx", "security", "traefik"]
blogger_orig_link: https://lexsysko.blogspot.com/2026/10/traefik-freebsd-reverse-proxy-acme.html
---

[![](/assets/images/blog/ce32a6246bcc03c8-beb1aa145c9d4587.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgcVSzwyBVcyX6TJTBIf7RxIvWZ3rrZuKALjge3nCEVwEH3FGi29CN5QogL7q1pBbh5vPqZzlDIMsEbVER2Vdhka7s_rbobR5xzN6-LR2ashitOp6OoU28J8qs0nM_TQhshsKEe7bRIqJm-_NXG84VHcy-BbJneV-7yQp1OzKmuhQk6fp5IdzkemkjOF8o/s1376/traefik_freebsd_jwt_banner.png)

#### Чому Traefik, а не NGINX із модулем JWT?

**Попередня архітектура:**

Для контролю доступу до приватного статичного контенту (зображень) за
допомогою JWT-токена в cookies використовувався сервер NGINX із модулем
[nginx-auth-jwt](https://github.com/kjdev/nginx-auth-jwt).

Оскільки у стандартних пакетах FreeBSD цей модуль відсутній, збірка
виконувалася через кастомний порт
[FreeBSD NGINX Port with JWT Authentication Module](https://github.com/lexsysko/port_nginx_add_jwt_auth).

**Недоліки використання рішення на NGINX у FreeBSD :**

* Потребувало постійної ручної компіляції при оновленні системи.
* Вимагало блокування автоматичного оновлення пакета через
  `pkg lock nginx`.

**Нове рішення на Traefik:**

Traefik виступає як незалежний збалансований проксі-сервер.

Завдяки підтримці плагінів (зокрема
[traefik-jwt-plugin](https://github.com/traefik-plugins/traefik-jwt-plugin)), авторизацію та перевірку токенів винесено на рівень проксі, що дозволило
повернутися до стандартного бінарного NGINX на бекенді.

#### Схема проходження запиту (Request Flow)

Ось як відбувається обробка HTTP/HTTPS-запиту від моменту звернення клієнта до
віддачі файлу з NGINX:

[![](/assets/images/blog/0d495366e964f729-2dacd88e0083c8ad.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjNHOzTv5mlfrAMUcFW2czSQYrbKzs7CXl2gOjpP92BKHHkHjcjPkkIO3O9kSbzyb4KXNAsgFmewGgN6mrDgwjCsAaIUMk7KHYnz70k25lnBQV8BfYoRRiNa6Hzb2Y3QrqZlvn1KnIMFYSHiS3EfaBcpUmS0wHhfWJ9bxOuYG4LOzwU-HNAmjPRP8ExYos/s1376/traefik_request_flow_diagram.png)  
*Request Flow*

#### Структура та приклад JWT-токена

  
Для аутентифікації використовується токен, збережений у cookies
*Auth\_token*.

Він складається з трьох частин: *Header*, *Payload* та
*Signature*.

**Структура Payload:**

* `sub` (Subject) — ідентифікатор користувача або сесії.
* `scope` — область доступу (у нашому випадку вимагається
  значення `access_token`).
* `dur` (Duration) — тривалість дії токена в секундах.
* `iat` (Issued At) — час видачі токена (UNIX Timestamp).
* `exp` (Expiration Time) — час закінчення дії токена (UNIX
  Timestamp).

Приклад JSON-полів (Decoded JWT):

```
// Header
{
  "alg": "ES256",
  "typ": "JWT"
}

// Payload
{ 
  "kid": "key-2026-11",
  "sub": "user_78942",
  "scope": "access_token",
  "dur": "3600",
  "iat": 1700000000,
  "exp": 1700003600
}
```

**Приклад cookies  (Encoded JWT):** 

```
Auth_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyXzc4OTQyIiwic2NvcGUiOiJhY2Nlc3NfdG9rZW4iLCJkdXIiOiIzNjAwIiwiaWF0IjoxNzAwMDAwMDAwLCJleHAiOjE3MDAwMDM2MDB9.SIGNATURE_STRING
```

#### Встановлення з FreeBSD Ports

Спроба встановити Traefik через бінарний менеджер пакетів
`pkg` виявила, що доступна версія містила відомі вразливості.
Тому встановлення виконувалося з актуального репозиторію портів FreeBSD:

```
# Клонуємо офіційний репозиторій портів FreeBSD
git clone --depth 1 https://git.FreeBSD.org/ports.git /usr/ports

# Переходимо в директорію порту Traefik та збираємо його
cd /usr/ports/net/traefik
make install clean

# Додаємо сервіс до автозапуску у /etc/rc.conf
sysrc traefik_enable="YES"
sysrc traefik_conf="/usr/local/etc/traefik.yml"
```

#### Підготовка системного середовища та ізоляція

Створюємо робочі каталоги для непривілейованого користувача
`traefik`:

**Особливість із плагінами:**

Traefik використовує фіксований шлях `/plugins-storage` для
збереження плагінів Go. Щоб дотримуватися стандартів файлової системи
FreeBSD, створюємо каталог у `/var/lib/` та прив'язуємо його
символьним посиланням.

```
# Конфігураційний каталог
mkdir -p /usr/local/etc/traefik
chown traefik:traefik /usr/local/etc/traefik
chmod 750 /usr/local/etc/traefik

# Директорія для логів
mkdir -p /var/log/traefik
chown traefik:traefik /var/log/traefik

# Сховище SSL-сертифікатів (ACME Let's Encrypt) з суворими правами 600
touch /usr/local/etc/traefik/acme.json
chown traefik:traefik /usr/local/etc/traefik/acme.json
chmod 600 /usr/local/etc/traefik/acme.json

# Налаштування каталогу плагінів
mkdir -p /var/lib/plugins-storage
chown traefik:traefik /var/lib/plugins-storage
ln -s /var/lib/plugins-storage /plugins-storage
```

#### Чотири варіанти відкриття портів 80/443 без root

За замовчуванням у UNIX-системах звичайний користувач не може відкривати
порти, менші за 1024. Розглянемо всі можливі підходи до вирішення цієї
задачі:

### Варіант 1: Запуск від імені root (Не рекомендовано)

Вказати у `/etc/rc.conf` параметр
`traefik_user="root"`.  
*Мінуси:* Порушує концепцію найменших привілеїв. У разі вразливості в
проксі компрометується вся операційна система.

### Варіант 2: Перенаправлення через Firewall (PF)

Traefik запускається на високих портах (наприклад, 7080 для HTTP та 7443 для
HTTPS), а фаєрвол перенаправляє зовнішні порти 80 та 443.

Приклад конфігурації `/etc/pf.conf` (де `vtnet0` —
зовнішній мережевий інтерфейс):

```
ext_if = "vtnet0"
rdr pass on $ext_if inet proto tcp from any to ($ext_if) port 80 -> 127.0.0.1 port 7080
rdr pass on $ext_if inet proto tcp from any to ($ext_if) port 443 -> 127.0.0.1 port 7443
```

Активація правил PF:

```
pfctl -nf /etc/pf.conf
pfctl -ef /etc/pf.conf
sysrc pf_enable="YES"
sysrc pf_rules="/etc/pf.conf"
```

### Варіант 3: Контейнеризація (Docker / Podman / Jails)

Якщо Traefik розгортається у контейнері, усередині контейнера сервіс слухає
високі порти (7080/7443), а контейнерний рушій виконує прокидання на хост:

```
ports:
  - "80:7080"
  - "443:7443"
```

### Варіант 4: Модуль ядра mac\_portacl (Основне рішення для BareMetal)

Використання підсистеми **MAC (Mandatory Access Control)** у
FreeBSD дозволяє надати конкретному користувачу (UID) право відкривати
привілейовані порти без підвищення його прав до root.

1. Визначаємо UID користувача `traefik` (наприклад,
`475`):

```
id traefik
# uid=475(traefik) gid=475(traefik) groups=475(traefik)
```

2. Вмикаємо завантаження модуля у `/boot/loader.conf`:

```
mac_portacl_load="YES"
```

3. Прописуємо конфігурацію в `/etc/sysctl.conf`:

```
net.inet.ip.portrange.reservedhigh=0
security.mac.portacl.port_high=1023
security.mac.portacl.suser_exempt=1
security.mac.portacl.rules=uid:475:tcp:80,uid:475:tcp:443
```

4. Застосовуємо зміни без перезавантаження ОС:

```
kldload mac_portacl
sysctl net.inet.ip.portrange.reservedhigh=0
sysctl security.mac.portacl.port_high=1023
sysctl security.mac.portacl.suser_exempt=1
sysctl security.mac.portacl.rules="uid:475:tcp:80,uid:475:tcp:443"
```

#### Головна конфігурація: /usr/local/etc/traefik.yml

```
global:
  checkNewVersion: false
  sendAnonymousUsage: false

api:
  dashboard: true
  insecure: false

entryPoints:
  web:
    address: ":80"
    http:
      aliasHeadersStrategy: delete
      redirections:
        entryPoint:
          to: websecure
          scheme: https
          permanent: true

  websecure:
    address: ":443"
    http:
      aliasHeadersStrategy: delete
      tls:
        certResolver: "letsencrypt"

certificatesResolvers:
  letsencrypt:
    acme:
      email: "webmaster@example.com"
      storage: "/usr/local/etc/traefik/acme.json"
      caServer: "https://acme-v02.api.letsencrypt.org/directory"
      httpChallenge:
        entryPoint: "web"

experimental:
  plugins:
    jwt:
      moduleName: github.com/traefik-plugins/traefik-jwt-plugin
      version: v0.10.0

providers:
  file:
    filename: "/usr/local/etc/traefik/dynamic.yml"
    watch: true

log:
  level: "INFO"
  filePath: "/var/log/traefik/traefik.log"

accessLog:
  filePath: "/var/log/traefik/access.log"
```

#### Динамічна маршрутизація: /usr/local/etc/traefik/dynamic.yml

```
http:
  middlewares:
    cookie-jwt-auth:
      plugin:
        jwt:
          Required: true
          PayloadFields:
            - exp
            - iat
            - scope
            - dur
            - sub
          JwtHeaders:
            X-Subject: sub
            X-Scope: scope
          JwtSources:
            - type: cookie
              key: "Auth_token"
          Keys:
            - "https://auth.example.com/.well-known/jwks.json"
            #- |
            #  -----BEGIN PUBLIC KEY-----
            #  "YOUR_PEM_SECRET_KEY_HERE"
            #  -----END PUBLIC KEY-----
          # secret: "YOUR_SYMETRIC_SECRET_KEY_HERE"

    custom-error-pages:
      errors:
        status:
          - "401"
          - "403"
          - "404"
          - "500"
        service: "nginx-static-service"
        query: "/errors/{status}"

  routers:
    image-proteced--router:
      priority: 100
      rule: "(Host(`www.example.com`) || Host(`example.com`)) && PathPrefix(`/E/`) && !Method('OPTIONS')"
      service: "nginx-static-service"
      middlewares:
        - custom-error-pages
        - cookie-jwt-auth

    static-router:
      priority: 10
      rule: "(Host(`www.example.com`) || Host(`example.com`) || Host(`vod.example.com`))"
      service: "nginx-static-service"

  services:
    nginx-static-service:
      loadBalancer:
        servers:
          - url: "http://127.0.0.1:8080"
```

#### Конфігурація бекенду NGINX (/usr/local/etc/nginx/nginx.conf)

```
worker_processes  1;
error_log  /var/log/nginx/error.log;

log_format proxy_log '$remote_addr - $remote_user [$time_local] '
                     '"$request" $status $body_bytes_sent '
                     '"$http_referer" "$http_user_agent" "$http_origin" '
                     'jwt_sub="$http_x_subject"';
access_log  /var/log/nginx/access.log  proxy_log;
keepalive_timeout  65;
server_tokens off;
gzip  on;

map $http_x_scope $auth_status {
    default        "denied";
    "access_token" "allowed";
}

events {
    worker_connections  1024;
}

http {
  include       mime.types;
  default_type  application/octet-stream;

  server {
    listen 127.0.0.1:8080;
    server_name  www.example.com example.com;
    root   /usr/local/www/example.com;

    set_real_ip_from 127.0.0.1;
    set_real_ip_from ::1;

    real_ip_header X-Forwarded-For;
    real_ip_recursive on;

    error_page 401 403 404 500 /errors/$status;

    location = /errors/401 { access_log off; default_type text/plain; return 401 "Unauthorized\n"; }
    location = /errors/403 { access_log off; default_type text/plain; return 403 "Access Denied\n"; }
    location = /errors/404 { access_log off; default_type text/plain; return 404 "Not Found\n"; }
    location = /errors/500 { access_log off; default_type text/plain; return 500 "Server Error\n"; }

    location ~ /\. {
        deny all;
    }

    location /E {
        alias /usr/local/www/example.com/E;  
        set $check_condition "${request_method}_${auth_status}";  
        if check_condition = "GET_denied") {  
            return 403;
        }
        auth_basic off;
        autoindex off;
        expires 30d;
    }
  }
}
```

#### Налаштування ротації логів (/usr/local/etc/newsyslog.conf.d/traefik.conf)

```
/var/log/traefik/traefik.log traefik:traefik 640 7 * @T00 ZC /var/run/traefik.pid 30
/var/log/traefik/access.log traefik:traefik 640 14 * @T00 ZC /var/run/traefik.pid 30
```

#### Запуск та перевірка логів

```
service traefik start
tail -f /var/log/traefik/traefik.log
tail -f /var/log/traefik/access.log
```
