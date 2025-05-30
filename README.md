# Tutorial 11 Pemrograman Lanjut: Kubernetes
### Madeline Clairine Gultom - 2306207846 - ADPRO A

#### Hello Minikube
> 1. Compare the application logs before and after you exposed it as a Service. Try to open the app several times while the proxy into the Service is running. What do you see in the logs? Does the number of logs increase each time you open the app?

Sebelum aplikasi diexpose:
![alt text](image/img1.png)
Pesan yang ditampilkan hanya berupa `Started HTTP server on port 8080` dan `Started UDP server on port  8081` yang  berarti aplikasi berhasil memulai server HTTP di port 8080 dan juga memulai server UDP di port 8081. Namun, tidak terdapat catatan log lainnya mengenai `request` yang masuk.

Sesudah aplikasi diexpose:
![alt text](image/img2.png)
Setelah menjalankan perintah `kubectl expose deployment hello-node --type=LoadBalancer --port=8080`, Kubernetes membuat sebuah Service yang membuka akses ke port 8080. Di Minikube, `LoadBalancer` disimulasikan sebagai `NodePort`. Kemudian, menjalankan perintah `minikube service hello-node` akan membuka URL lokal. Setiap kali URL tersebut diakses, aplikasi menerima permintaan `GET /`, dan log akan mencatat setiap request yang masuk. Jumlah log akan terus bertambah sesuai dengan banyaknya permintaan yang diterima.


#### Rolling Update & Kubernetes Manifest File
> 1. What is the difference between Rolling Update and Recreate deployment strategy?

Terdapat dua strategi deployment, yaitu Rolling Update dan Recreate untuk memperbarui aplikasi yang sedang berjalan. Perbedaan utama dari kedua strategi ini adalah dari sisi **downtime**-nya, yaitu kondisi ketika aplikasi tidak dapat diakses untuk sementara waktu, biasa terjadi ketika semua pod dalam keadaan mati. 

`Rolling Update` bekerja dengan membuat pod baru satu per satu dan pod lama akan dihapus secara bertahap dan tidak ada downtime (aplikasi tetap berjalan). Sedangkan `Recreate` bekerja dengan menghapus seluruh pod lama dan membuat pod baru sekaligus, ada downtime yang membuat aplikasi tidak dapat diakses sementara.

> 2. Try deploying the Spring Petclinic REST using Recreate deployment strategy and document your attempt.

Langkah-langkah melakukan *deploy* menggunakan strategi deployment `Recreate`.
- Membuat deployment kembali untuk menggunakan strategi deployment dan menyimpan konfigurasinya pada file `spring-petclinic-recreate.yaml` dengan menjalankan 
```
kubectl create deployment spring-petclinic-recreate \
  --image=springcommunity/spring-petclinic-rest \
  --dry-run=client -o yaml > spring-petclinic-recreate.yaml
  ```
- Memasukkan tipe strategi `Recreate` pada file .yaml tersebut
```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: spring-petclinic-recreate
  name: spring-petclinic-recreate
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-petclinic-recreate
  strategy:
    type: Recreate      # Tambahkan pada baris berikut
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: spring-petclinic-recreate
    spec:
      containers:
      - image: springcommunity/spring-petclinic-rest
        name: spring-petclinic-rest
        resources: {}
status: {}
```
- Menerapkan deployment tersebut ke cluster dengan menjalankan perintah `kubectl apply -f spring-petclinic-recreate.yaml`
- Melakukan expose pada deployment dengan menajalankan perintah `kubectl expose deployment spring-petclinic-recreate --type=LoadBalancer --port=9966`
- Memeriksa status dengan menjalankan perintah `kubectl get pods`
Output:
```
NAME                                         READY   STATUS    RESTARTS   AGE
spring-petclinic-recreate-546bd65c8c-xd7xg   1/1     Running   0          3m41s
spring-petclinic-rest-6db6bdbb69-6t2dd       1/1     Running   0          35m
spring-petclinic-rest-6db6bdbb69-9mlmz       1/1     Running   0          35m
spring-petclinic-rest-6db6bdbb69-qjzsp       1/1     Running   0          35m
spring-petclinic-rest-6db6bdbb69-r4dnj       1/1     Running   0          35m
```
- Menyimpan konfigurasi service dengan menjalankan perintah `kubectl get services/spring-petclinic-recreate -o yaml > spring-petclinic-recreate-service.yaml`

> 3. Prepare different manifest files for executing Recreate deployment strategy

Saya telah membuat dua file, yaitu file konfigurasi deployment dan service pada tahap sebelumnya yang telah menggunakan strategi deployment `Recreate`.

> 4. What do you think are the benefits of using Kubernetes manifest files? Recall your experience in deploying the app manually and compare it to your experience when deploying the same app by applying the manifest files (i.e., invoking `kubectl apply -f` command) to the cluster.

Dengan menggunakan manifest file, proses deployment tidak perlu lagi menjalankan berbagai perintah manual. Lalu, karena seluruh konfigurasi telah disimpan dalam bentuk file, maka akan lebih konsisten dan dapat digunakan kembali kapan saja dengan menjalankan perintah `kubectl apply -f <nama_file>>.yaml`. Dengan manifest file, akan lebih terbuka dan leluasa juga jika ingin mengubah konfigurasi, semisal mengubah image, jumlah `replicas`, dan lainnya yaitu dengan mengubah file .yaml tersebut. Selain itu, penggunaan manifest file juga memungkinkan kolaborasi antar anggota tim, karena file konfigurasi dapat disimpan di repositori dan dibagikan dengan mudah. Dari sisi pengelolaan, manifest file membuat struktur deployment menjadi lebih teratur, rapi, dan mudah dikelola.