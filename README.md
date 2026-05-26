# Лабораторная 3
## Часть 1
### О работе
структура работы
* `app1/`, `app2/`: директории приложений.
* `nginx/conf.d/`: конфигурация виртуальных хостов.
* `nginx/ssl/`: SSL-сертификаты.
файлы конфигурации
app.py: код приложения, отвечает за ответ сервера (hello world)
Dockerfile: инструкция по сборке среды выполнения для app1, app2
nginx.conf: основной файл nginx, связывает все настройки виртуальных хостов
app1.conf / app2.conf: содержат правила для виртуальных хостов: редирект с 80 на 443 порта, проксирование до python-приложения и использование alias для отдачи файлов
docker-compose.yml: оркестратор, который запускает всю инфраструктуру одной командой
### Команды в терминале
```bash
# 1. генерируем сертификат и запускаем
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout lab3/nginx/ssl/nginx.key -out lab3/nginx/ssl/nginx.crt \
  -subj "/CN=localhost"

# 2. запускаем
cd lab3 && docker-compose up -d

# 3. прописываем домены
echo "127.0.0.1 app1.local app2.local" | sudo tee -a /etc/hosts

# 4. Проверка работы
curl -I http://app1.local
curl -k https://app1.local
curl -I http://app2.local
curl -k https://app2.local/data/
```
### Артефакты выполнения
<img width="1502" height="777" alt="image" src="https://github.com/user-attachments/assets/899dc7a9-e3e1-44e2-92fe-e46f2f3b3dde" />
Здесь видно, что все работает корректно
## Часть 2
