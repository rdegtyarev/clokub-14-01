# Домашнее задание к занятию "14.1 Создание и использование секретов"

## Задача 1: Работа с секретами через утилиту kubectl в установленном minikube

Выполните приведённые ниже команды в консоли, получите вывод команд. Сохраните
задачу 1 как справочный материал.

### Как создать секрет?

```
openssl genrsa -out cert.key 4096
openssl req -x509 -new -key cert.key -days 3650 -out cert.crt \
-subj '/C=RU/ST=Moscow/L=Moscow/CN=server.local'
kubectl create secret tls domain-cert --cert=certs/cert.crt --key=certs/cert.key
```
#### Решение
```
openssl genrsa -out cert.key 4096

openssl req -x509 -new -key cert.key -days 3650 -out cert.crt \
-subj '/C=RU/ST=Moscow/L=Moscow/CN=server.local'

kubectl create secret tls domain-cert --cert=./cert.crt --key=./cert.key
secret/domain-cert created
```

### Как просмотреть список секретов?

```
kubectl get secrets
kubectl get secret
```

#### Решение
```
kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-hn7nb   kubernetes.io/service-account-token   3      9m19s
domain-cert           kubernetes.io/tls                     2      84s

kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-hn7nb   kubernetes.io/service-account-token   3      11m
domain-cert           kubernetes.io/tls                     2      3m21s
```

### Как просмотреть секрет?

```
kubectl get secret domain-cert
kubectl describe secret domain-cert
```

#### Решение
```
kubectl get secret domain-cert
NAME          TYPE                DATA   AGE
domain-cert   kubernetes.io/tls   2      3m37s

kubectl describe secret domain-cert
Name:         domain-cert
Namespace:    clokub-14-01
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/tls

Data
====
tls.key:  3268 bytes
tls.crt:  1944 bytes
```

### Как получить информацию в формате YAML и/или JSON?

```
kubectl get secret domain-cert -o yaml
kubectl get secret domain-cert -o json
```

#### Решение
```
kubectl get secret domain-cert -o yaml
apiVersion: v1
data:
  tls.crt: <<<многобукв>>>
  tls.key: <<<многобукв>>>
kind: Secret
metadata:
  creationTimestamp: "2022-06-11T09:04:34Z"
  name: domain-cert
  namespace: clokub-14-01
  resourceVersion: "403869"
  uid: 93140037-eb96-44fc-b078-b24c6b7ddc86
type: kubernetes.io/tls

kubectl get secret domain-cert -o json
{
    "apiVersion": "v1",
    "data": {
        "tls.crt": "<<<многобукв>>>",
        "tls.key": "<<<многобукв>>>"
    },
    "kind": "Secret",
    "metadata": {
        "creationTimestamp": "2022-06-11T09:04:34Z",
        "name": "domain-cert",
        "namespace": "clokub-14-01",
        "resourceVersion": "403869",
        "uid": "93140037-eb96-44fc-b078-b24c6b7ddc86"
    },
    "type": "kubernetes.io/tls"
}
```

### Как выгрузить секрет и сохранить его в файл?

```
kubectl get secrets -o json > secrets.json
kubectl get secret domain-cert -o yaml > domain-cert.yml
```

#### Решение
При выполнении команд создаются соотвествующие файлы


### Как удалить секрет?

```
kubectl delete secret domain-cert
```

#### Решение
```
kubectl delete secret domain-cert
secret "domain-cert" deleted

kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-hn7nb   kubernetes.io/service-account-token   3      19m
```

### Как загрузить секрет из файла?

```
kubectl apply -f domain-cert.yml
```

#### Решение
```
kubectl apply -f domain-cert.yml
secret/domain-cert created

kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-hn7nb   kubernetes.io/service-account-token   3      19m
domain-cert           kubernetes.io/tls                     2      2s
```

---

## Задача 2 (*): Работа с секретами внутри модуля

Выберите любимый образ контейнера, подключите секреты и проверьте их доступность
как в виде переменных окружения, так и в виде примонтированного тома.

### Решение


Создаем deployment, указываем монтирование сертификатов из предыдущего задания:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
  namespace: clokub-14-01
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx
          imagePullPolicy: Always
          name: nginx
          resources:
            limits:
              cpu: 200m
              memory: 256Mi
            requests:
              cpu: 100m
              memory: 128Mi
          volumeMounts:
            - name: certs
              mountPath: "/etc/nginx/ssl"
              readOnly: true
          ports:
            - containerPort: 80
      volumes:
        - name: certs
          secret:
            secretName: domain-cert
```
  
Применяем:
```
kubectl apply -f app/
deployment.apps/nginx created

kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-756bb998cd-bl2dc   1/1     Running   0          2m31s
```
  
Проверяем что наши сертификаты примонтировались в /etc/nginx/ssl/
```
kubectl exec -it nginx-756bb998cd-bl2dc -- bash
root@nginx-756bb998cd-bl2dc:/# cd /etc/nginx/ssl/
root@nginx-756bb998cd-bl2dc:/etc/nginx/ssl# ls
tls.crt  tls.key
```
---
