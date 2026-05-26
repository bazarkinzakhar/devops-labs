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
Для взлома сайта я выбрал специальный сайт для проверки уязвимостей
Адрес сайта https://owasp.org/www-project-juice-shop/
### Методы проерки уязвимостей
1. directory enumeration (перебор страниц)
суть: поиск скрытых директорий, которые не отображаются в интерфейсе, но доступны по прямому запросу. это позволяет найти админ-панели или файлы конфигурации
Результат
<img width="1502" height="594" alt="image" src="https://github.com/user-attachments/assets/d4056a9d-0fd4-430e-bab4-06c54c7e17ae" />
ffuf ничего не выдал, что говорит об устойчивости сайта от перебора директорий
```bash
# использую ffuf для перебора директорий
ffuf -u https://juice-shop.herokuapp.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -mc 200,301,403
```
Результат

2. path traversal (обход путей)
суть: попытка выйти за пределы корневой папки веб-сервера с помощью спецсимволов ../, чтобы получить доступ к системным файлам
```bash
# запрос к файлу passwd через директорию ftp
curl -k https://juice-shop.herokuapp.com/ftp/../../etc/passwd
```
Результат:
<img width="1502" height="742" alt="image" src="https://github.com/user-attachments/assets/9629f460-f57d-45c3-b54f-21655d46c7fa" />
уязвимость не подтверждена. сервер вернул код главной страницы (200 OK), а не содержимое системного файла. защита сработала корректно
3. SQL Injection (SQL-инъекция)
Суть: Внедрение вредоносного SQL-кода в поля ввода, чтобы манипулировать базой данных (например, для авторизации без пароля)
```bash
# отправка post-запроса с инъекцией в поле email
curl -X POST https://juice-shop.herokuapp.com/rest/user/login \
     -H "Content-Type: application/json" \
     -d '{"email": "admin@juice-sh.op'\''--", "password": "any"}'
```
Результат
<img width="1502" height="795" alt="image" src="https://github.com/user-attachments/assets/ccf07f6d-7dcf-4cfe-9511-7ca1bf26dfbd" />
уязвимость подтверждена. приложение не выполняет фильтрацию входных данных, из-за чего внедренный символ ' нарушает синтаксис SQL-запроса, вызывая критическую ошибку на стороне сервера
### Вывод
Учебный сайт для поиска уязвимостей оказался устойчив к базовым методам, в дальнейшем я попробую более продвинутые методы и другие сайты
