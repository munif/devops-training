- [1. Integrasi Docker ke CI/CD](#1-integrasi-docker-ke-cicd)
  - [1.1. Setup Docker Host](#11-setup-docker-host)
    - [1.1.1. Install VM](#111-install-vm)
    - [1.1.2. Install Docker](#112-install-docker)
  - [1.2. Create Tomcat container](#12-create-tomcat-container)
  - [1.3. Integrasi Docker dan Jenkins](#13-integrasi-docker-dan-jenkins)
    - [1.3.1. Install ```Publish over SSH``` plugin](#131-install-publish-over-ssh-plugin)
  - [1.4. Deploy Container](#14-deploy-container)
    - [1.4.1. Membuat Job Deploy to Container](#141-membuat-job-deploy-to-container)
    - [1.4.2. Update Dockerfile untuk melakukan deployment otomatis](#142-update-dockerfile-untuk-melakukan-deployment-otomatis)
  - [1.5. Melakukan otomasi build and deploy container.](#15-melakukan-otomasi-build-and-deploy-container)

# 1. Integrasi Docker ke CI/CD

## 1.1. Setup Docker Host
### 1.1.1. Install VM
Gunakan VM yang telah disediakan

### 1.1.2. Install Docker
1. Install Docker
    ```
    $ sudo apt-get update
    $ sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    
    $ sudo mkdir -p /etc/apt/keyrings
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    
    $ echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    
    $ sudo apt-get update
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```
2. Cek status service docker
    ```
    $ sudo service docker status
    ```
    
    Apabila belum aktif, aktifkan dengan perintah ```sudo docker start```.

    Tes Docker menjalankan command berikut:
    ```
    sudo docker run hello-world
    ```
3. Create user ```dockeradmin```

    User ini akan digunakan sebagai Jenkins untuk melakukan deployment ke server Docker ini.
    ```
    $ sudo su -
    $ useradd dockeradmin
    $ passwd dockeradmin
    $ usermod -aG docker dockeradmin
    $ mkdir /home/dockeradmin
    $ chown -R dockeradmin:dockeradmin /home/dockeradmin
    $ chsh -s /bin/bash dockeradmin
    ```

## 1.2. Create Tomcat container
1. Lakukan pull container Tomcat
    ```
    $ sudo docker pull tomcat
    ```

    Cek images yang telah di-download
    ```
    $ sudo docker images
    ```
2. Buat container baru dari image tomcat
    ```
    $ sudo docker run -d --name tomcat-container -p 8081:8080 tomcat
    ```
3. Lakukan perintah berikut agar Tomcat dapat berjalan dengan baik
    ```
    $ sudo docker exec -it tomcat-container /bin/bash

    # Bash di dalam Docker container
    $ cp -R webapps.dist/* webapps
    $ exit
    ```
4. Refresh kembali browser dan pastikan Tomcat dapat diakses.

## 1.3. Integrasi Docker dan Jenkins
### 1.3.1. Install ```Publish over SSH``` plugin
1. Login ke Jenkins dan buka Dashboard.
2. Pilih menu ```Manage Jenkins -> Manage Plugins```
3. Install plugin "Publish over SSH"
4. Pilih menu ```Manage Jenkins -> Configure System```.
5. Atur setting ```Publish over SSH -> SSH Server```
    - Name: docker-server
    - Hostname: alamat IP docker-server
    - Username: dockeradmin
    Untuk memunculkan setting password, klik ```Advanced```.
    - Password: 1234
6. Klik ```Test configuration``` untuk memastikan server Jenkins bisa SSH ke docker-server
   
## 1.4. Deploy Container
### 1.4.1. Membuat Job Deploy to Container
1. Buat job baru dengan nama ```BuildAndDeployOnContainer``` dengan cara meng-copy dari job ```BuildAndDeployJob```.
2. Pada bagian ```Post-build Actions```, hapus ```Deploy war/ear to a container``` dengan mengeklik tanda 'x'.
3. Pada ```Post build action```, pilih ```Send build artifacts over SSH```.
    - Name: docker-server
    - Source files: webapp/target/*.war
    - Remove prefix: webapp/target
4. Login ke server docker dengan menggunakan user ```dockeradmin```.
5. Cek folder ```/home/dockeradmin``` dan pastikan ada file ```webapp.war``` yang tercopy di sana.

### 1.4.2. Update Dockerfile untuk melakukan deployment otomatis
1. Login ke ```docker-server``` sebagai root (```sudo su -```).
2. Buat direktori dan ubah owner ke ```dockeradmin```.
    ```
    $ cd /opt/
    $ mkdir docker
    $ chown -R dockeradmin:dockeradmin docker
    ```
3. Buat file ```Dockerfile``` di ```/opt/docker```.
    ```
    FROM tomcat:latest
    RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
    COPY ./*.war /usr/local/tomcat/webapps
    ```   
4. Update job untuk meng-copy hasil build ke ```//opt//docker```.
    - Remote directory: ```//opt//docker```
5. Jalankan job. Pastikan job success dan terdapat file ```webapp.war``` pada folder ```/opt/docker```.
6. Ubah ownership dari file ```Dockerfile``` dan ```webapp.war``` ke user ```dockeradmin```.
7. Build image yang baru.
    ```
    $ docker build -t tomcat:v1 .
    ```
    Cek hasilnya dengan menggunakan perintah ```docker images```.

8. Jalankan container baru dengan menggunakan image yang baru.
    ```
    $ docker run -d --name tomcatv1 -p 8086:8080 tomcat:v1
    ```

9. Kunjungi website yang baru dengan menggunakan port 8086.
10. Apabila container tidak digunakan lagi, kita dapat mematikan dengan command ```docker stop <ID>``` dan ```docker container prune```.

## 1.5. Melakukan otomasi build and deploy container.
1. Konfigurasi ```Send build artifacts over SSH``` pada bagian ```Exec command```. Tambahkan script berikut untuk melakukan otomasi build images.
    ```
    cd /opt/docker;
    docker build -t regapp:v1 .;
    docker run -d --name registerapp -p 8087:8080 regapp:v1
    ```
2. Jalankan job dan cek images yang dihasilkan di ```docker-server```.
    Job ini juga akan menghasilkan 1 container dengan nama ```registerapp```. Cek dari browser http://<IP-docker-server>:8087/webapp.
3. Tes jalankan job lagi dan amati error yang terjadi.
    Job gagal melakukan deployment karena menghasilkan nama container yang sama.
4. Untuk memperbaiki error tersebut, tambahkan tahapan untuk menghentikan container dan menghapus docker container. Berikut adalah bagian ```Exec command``` selengkapnya.
    ```
    cd /opt/docker;
    docker build -t regapp:v1 .;
    docker stop registerapp;
    docker rm registerapp;
    docker run -d --name registerapp -p 8087:8080 regapp:v1;
    ```