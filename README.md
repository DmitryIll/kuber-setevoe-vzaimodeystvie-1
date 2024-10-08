# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1» - Илларионов Дмитрий

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.

Уже был такой деплоймент, толкьо реплик было 2 изменил на 3:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "1180"
        - name: HTTPS_PORT
          value: "11443"
        ports:
        - containerPort: 1180
          name: http-port
        - containerPort: 11443
          name: https-port
```

Применил:

![alt text](image.png)

2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.

тип сервиса будет ClusterIP MultiPort.

```
apiVersion: v1
kind: Service
metadata:
  name: my-svc-multiport
spec:
  selector:
    app: nginx
  ports:
    - name: nginx
      protocol: TCP
      port: 9001
      targetPort: 80
    - name: multitool
      protocol: TCP
      port: 9002
      targetPort: 8080
```

Применил.

![alt text](image-1.png)


3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.

Под остался от прошлого ДЗ еще.
Подключаюсь:
```
kubectl exec multitool-tmp -it bash
```

4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.

```
curl my-svc-multiport:9001
```

![alt text](image-2.png)


```
curl my-svc-multiport:9002
```
Не подключается:
![alt text](image-3.png)

```
kubectl describe svc/my-svc-multiport
```
![alt text](image-4.png)

Все ок, но, причина в том что в поде был прописан другой порт:

```
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "1180"
```

Меняю код пода на порт 8080:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "8080"
        - name: HTTPS_PORT
          value: "11443"
        ports:
        - containerPort: 8080
          name: http-port
        - containerPort: 11443
          name: https-port
```

Применяю.
![alt text](image-5.png)

Теперь заработало:

![alt text](image-6.png)

5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

Предоставил выше.

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.

Создал:

```
apiVersion: v1
kind: Service
metadata:
  name: my-svc-nodeport
  namespace: default
spec:
  selector:
    app: nginx
  ports:
    - name: web
      port: 80
      protocol: TCP
      nodePort: 30080
  type: NodePort
```
Применил:
![alt text](image-7.png)

2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.

![alt text](image-8.png)

Заработало:

![alt text](image-12.png)


3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

Предоставил см. выше.

------

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

