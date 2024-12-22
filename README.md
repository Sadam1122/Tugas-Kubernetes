# Mikroservis Kubernetes dengan HPA

Proyek ini mendemonstrasikan penerapan dua mikroservis (Webserver dan Database) pada Kubernetes menggunakan Minikube dengan driver Docker. Horizontal Pod Autoscaler (HPA) dikonfigurasi untuk melakukan penskalaan otomatis pada deployment Webserver berdasarkan penggunaan CPU. Selain itu, dilakukan pengujian dengan HTTP Traffic Generator untuk memvalidasi fungsi HPA.

---

## Daftar Isi
- [Prasyarat](#prasyarat)
- [Setup Minikube](#setup-minikube)
- [Deploy Webserver](#deploy-webserver)
- [Deploy Database](#deploy-database)
- [Konfigurasi HPA untuk Webserver](#konfigurasi-hpa-untuk-webserver)
- [Buat Service untuk Webserver](#buat-service-untuk-webserver)
- [Pengujian Beban dengan HTTP Traffic Generator](#pengujian-beban-dengan-http-traffic-generator)
- [Verifikasi Penskalaan HPA](#verifikasi-penskalaan-hpa)
- [Dokumentasi dan Video](#dokumentasi-dan-video)

---

## Prasyarat

Pastikan Anda telah menginstal dan mengonfigurasi alat berikut pada sistem Anda:
1. **Minikube**: Alat untuk menjalankan Kubernetes secara lokal.
2. **Kubectl**: Alat command-line untuk Kubernetes.
3. **Docker**: Runtime container untuk driver Minikube.
4. **HTTP Traffic Generator**: Alat seperti `httperf` atau `wrk` untuk pengujian beban.

---

## Setup Minikube

Mulai Minikube dengan driver Docker:

```bash
minikube start --driver=docker
```

---

## Deploy Webserver

1. Buat file deployment `webserver-deployment.yaml`:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: webserver
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: webserver
      template:
        metadata:
          labels:
            app: webserver
        spec:
          containers:
          - name: nginx-container
            image: nginx:latest
            ports:
            - containerPort: 80
    ```

2. Terapkan deployment:

    ```bash
    kubectl apply -f webserver-deployment.yaml
    ```

---

## Deploy Database

1. Buat file deployment `database-deployment.yaml`:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: database
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: database
      template:
        metadata:
          labels:
            app: database
        spec:
          containers:
          - name: mysql-container
            image: mysql:5.7
            env:
            - name: MYSQL_ROOT_PASSWORD
              value: rootpassword
            ports:
            - containerPort: 3306
    ```

2. Terapkan deployment:

    ```bash
    kubectl apply -f database-deployment.yaml
    ```

---

## Konfigurasi HPA untuk Webserver

Aktifkan HPA untuk deployment Webserver:

```bash
kubectl autoscale deployment webserver --cpu-percent=50 --min=2 --max=5
```

Verifikasi konfigurasi HPA:

```bash
kubectl get hpa
```

---

## Buat Service untuk Webserver

1. Buat file service `webserver-service.yaml`:

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: webserver
    spec:
      selector:
        app: webserver
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
      type: NodePort
    ```

2. Terapkan service:

    ```bash
    kubectl apply -f webserver-service.yaml
    ```

3. Dapatkan URL service:

    ```bash
    minikube service webserver --url
    ```

---

## Pengujian Beban dengan HTTP Traffic Generator

Gunakan `httperf` atau `wrk` untuk menghasilkan trafik HTTP:

```bash
httperf --hog --num-conns=1000 --rate=10 --server=<webserver-ip>
```

Ganti `<webserver-ip>` dengan IP yang diperoleh dari perintah `minikube service`.

---

## Verifikasi Penskalaan HPA

Periksa penskalaan pod selama atau setelah pengujian beban:

```bash
kubectl get pods
```

Hasil yang diharapkan menunjukkan beberapa pod `webserver` berjalan:

```
NAME                         READY   STATUS    RESTARTS   AGE
webserver-xxxxx              1/1     Running   0          1m
webserver-yyyyy              1/1     Running   0          1m
database-xxxxx               1/1     Running   0          1m
```

---


## Catatan

- Pastikan CPU Minikube mencukupi untuk mendukung penskalaan HPA.
- Gunakan perintah `kubectl describe` untuk debugging jika pod tidak melakukan penskalaan.

---
