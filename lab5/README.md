# Лабораторная работа 5
## Артефакты выполнения
1. Установил систему мониторинга <img width="1512" height="817" alt="image" src="https://github.com/user-attachments/assets/3c63a2ae-eb10-4417-a48a-107b7a2d4c18" />
2. Пробросил порты <img width="1512" height="283" alt="image" src="https://github.com/user-attachments/assets/717c1f96-cd64-48f9-abc1-f3b6897b4a2a" />
3. Запрашиваем пароль в терминале (kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode; echo) и заходим на локалхост <img width="1512" height="864" alt="image" src="https://github.com/user-attachments/assets/e0fbbb06-14b8-4424-a6ad-ad205fca8266" />
Данные для входа

| Имя | Пароль |
| :--- | :--- |
| admin | password |

5. Графики работы системы
<img width="1512" height="816" alt="image" src="https://github.com/user-attachments/assets/04595dfc-04fc-4150-be75-e1392d419ce1" />
<img width="1512" height="747" alt="image" src="https://github.com/user-attachments/assets/cc4d9df3-6992-47a1-93e2-e1504b406fcb" />

## Описание кода
### лабораторная работа 5: мониторинг сервиса в kubernetes

порядок выполнения шагов::

```bash
# добавление репозиторя с мониторингом и обновление списка
helm repo add prometheus-community [https://prometheus-community.github.io/helm-charts](https://prometheus-community.github.io/helm-charts)
helm repo update

# установка prometheus и grafana в отдельный неймспейс
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace

# проверка, что все поды мониторинга запустились
kubectl get pods -n monitoring

# пароль администратора для входа в grafana
kubectl get secret --namespace monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

# проброска портов для доступа к веб интерфейсу grafana
kubectl port-forward svc/prometheus-grafana 8080:80 -n monitoring
