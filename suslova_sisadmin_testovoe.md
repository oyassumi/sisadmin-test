1. awk '{if ($10 ~ /^[0-9]+$/) a[$1] += $10} END {for (ip in a) print a[ip] " bytes for " ip}' access.log

2. lsof -i TCP -s TCP:ESTABLISHED

3. Сервис стал недоступен из-за исчерпания лимита подключений к MySQL (метрика max_connections). База перегрузилась из-за большого количества одновременно выполняющихся запросов. Время недоступности: с 13:00 до 13:20 и дольше, так как на графиках нет момента восстановления сервиса.

4. Ответ:
- Тег latest лучше избегать в production, так как он не дает понимания, какая именно версия развернута. Вместо него лучше использовать конкретную версию.
- Инструкция MAINTAINER устарела, вместо неё следует использовать LABEL.
- Можно добавить очистку кэша apt (`rm -rf /var/lib/apt/lists/*`) и флаг `--no-install-recommends` для уменьшения размера образа.
- RUN можно объединить, чтобы не создавать лишние слои.
- Можно использовать `.dockerignore`, чтобы исключить из сборки лишние файлы.
- Копировать только нужное (например, `COPY ./static /var/www/html`).
- Инструкцию COPY можно разместить после установки пакетов, чтобы изменения в файлах не сбрасывали кэш слоя с apt.
- Не заявлен порт, можем указать EXPOSE 80.

5. Ответ:
- Проверю доступность — curl -I http://<сервер> с разных точек, чтобы убедиться в проблеме.
- Проверю статус Nginx — systemctl status nginx. Если он не работает, перезагружу  systemctl restart nginx.
- Посмотрю логи tail -n 50 /var/log/nginx/error.log и journalctl -u nginx --since "5 min ago".
- Проверю конфигурацию nginx -t, если ошибка исправлю или откачу последние изменения.
- Проверю, что порт слушается ss -tlnp | grep :80, проверю firewall iptables -L.
- Оценю загруженность ресурсов: CPU, памяти, диска top, df -h, освобожу место при необходимости.
- Если не помогло, то переключу трафик на резервный узел. После этого углублюсь в диагностику (атаки, бэкенды, лимиты).

6. Создадим три стандартных манифеста: Deployment, Service и Ingress.

Шаги по развертыванию: 
- Подготовим образ, проверим, что он загружен в registry (например, Docker Hub или Selectel Container Registry)
- Подготовим манифесты.
- Применим манифесты в кластере Kubernetes.
- Проверим статус.

Манифесты для развертывания

**deployment.yaml**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app-image
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

```

**service.yaml**

```
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80

```

**ingress.yaml**

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
  - host: my-app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

7. В первом сообщении обязательно поздороваюсь и проявлю эмпатию. Сообщу, что вопрос взят в работу, чтобы заказчик не переживал.

> Здравствуйте!
>
> Понимаем ваше огорчение, ситуация не из приятных.
>
> Мы все детально проверим и вернемся к вам с ответом в ближайшее время.

Заказчик пишет «Опять», значит это произошло не в первый раз. Проверю историю обращений заказчика и его активные проекты, чтобы понимать о чем идет речь. Если ситуация не прояснится, уточню детали:
- Какой именно проект не работает?
- Запрошу скриншот ошибки.

Далее, чтобы не терять время, сразу приступлю к технической проверке:
- Проверю доступность сервиса (ping, порты, веб-интерфейс).
- Посмотрю свежие записи в логах на предмет критических ошибок.
- Проверю состояние ресурсов (место на диске, память, загрузка CPU).

