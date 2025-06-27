# Деплой React + Strapi на сервер с Nginx, PM2 и HTTPS

Полный гайд по развёртыванию React-приложения и Strapi-бэкенда на сервере с использованием:

- **PM2** — для управления процессами Strapi
- **Nginx** — как реверс-прокси и сервер статики
- **Certbot** — для SSL-сертификатов Let's Encrypt
- **Vite** + React на фронте

---

## 📁 Структура проекта на сервере

```bash
/var/www/
├── lesnaya-zastava-frontend     # React (build)
└── lesnaya-zastava-backend      # Strapi (PM2)
```

## Шаг 0: Настройка

Нужно перейти в директорию frontend и открыть .env файл -

```bash
cd /lesnaya-zastava-frontend/
sudo nano .env
```

там будет примерно такое содержание -

```bash
VITE_API_BASE_URL=https://leznaya-zastava.ydns.eu
```

нужно будет поменять значение этой строки на то, на котором будет лежать ваш бэкенд strapi.

и затем установить зависимости командой -

::: warning

```bash
npm i --force
```

:::

Далее перейдем в директорию backend -

```bash
cd /lesnaya-zastava-backend/config/
sudo nano middlewares.ts
```

внутри будет примерно такое содержимое -

```bash
export default [
  "strapi::logger",
  "strapi::errors",
  "strapi::security",
  {
    name: "strapi::cors",
    config: {
      origin: ["https://leznaya-zastava.ydns.eu", "http://localhost:5174"],
      methods: ["GET", "POST", "PUT", "DELETE", "PATCH"],
      headers: "*",
    },
  },
  "strapi::poweredBy",
  "strapi::query",
  "strapi::body",
  "strapi::session",
  "strapi::favicon",
  "strapi::public",
];
```

здесь нужно менять строку origin, добавить ссылку на ваш frontend, он позволит совершать запросы с данного домена

вот как можно загрузить файл на удаленный сервер по ssh

```bash
rsync -avz -e "ssh -i ~/.ssh/ssh_keys/myserver-key" ./lesnaya-zastava-backend/ root@94.232.40.253:/var/www/lesnaya-zastava-backend/

rsync -avz -e "ssh -i ~/.ssh/ssh_keys/myserver-key" ./lesnaya-zastava-frontend/dist/ root@94.232.40.253:/var/www/lesnaya-zastava-frontend/

```

## 🛠️ Шаг 1: Установка зависимостей

```bash
sudo apt update && sudo apt install -y
  nginx curl nodejs npm git
  certbot python3-certbot-nginx
```

## 🛠️ ⚙️ Шаг 2: Установка и запуск Strapi

```bash
cd /var/www/lesnaya-zastava-backend/
npm i
npm run build

sudo npm install -g pm2
pm2 start npm --name lesnaya-strapi -- run start
pm2 save
pm2 startup
```

Выполни команду, которую покажет pm2 startup (например):

```bash
sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u $USER --hp /home/$USER
```

## Шаг 3: Сборка фронтенда (React + Vite)

```bash
cd /path/to/your/react-app
npm run build
```

Затем нужно скопировать билд -

```bash
sudo mkdir -p /var/www/lesnaya-zastava-frontend
sudo cp -r dist/* /var/www/lesnaya-zastava-frontend/
```

## 🌐 Шаг 4: Настройка Nginx

Нужно создать файл -

```bash
sudo nano /etc/nginx/sites-available/lesnaya-zastava
```

И вот конфигурация которая должна там быть -

```bash
server {
    listen 80;
    server_name здесь.имена.ваших.доменов;

    root /var/www/lesnaya-zastava-frontend;
    index index.html;

    location /uploads/ {
        alias /var/www/lesnaya-zastava-backend/dist/build/uploads/;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:1337/admin/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location ~ ^/(content-|users-|api/|upload/|i18n/|email/|admin/) {
        proxy_pass http://127.0.0.1:1337;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Теперь активируем nginx конфиг -

```bash
sudo ln -s /etc/nginx/sites-available/lesnaya-zastava /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## 🔐 Шаг 5: Установка HTTPS через Certbot

```bash
sudo certbot --nginx -d leznaya-zastava.ydns.eu
```

Certbot:

- сам добавит HTTPS блок

- установит SSL сертификат от Let's Encrypt

## 🔁 Шаг 6: Автоматическое продление SSL

Нужно проверить cron-задачи -

```bash
sudo crontab -l
```

Если нет задачи `certbot renew`, добавь:

```bash
sudo crontab -e
```

И нужно вставить -

```bash
0 3 * * * certbot renew --quiet --nginx && systemctl reload nginx
```
