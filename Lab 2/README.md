- [1. Integrasi Tomcat Server dan CI/CD](#1-integrasi-tomcat-server-dan-cicd)
  - [1.1. Instalasi Tomcat](#11-instalasi-tomcat)
    - [1.1.1. Menyiapkan VM](#111-menyiapkan-vm)
    - [1.1.2. Install Java](#112-install-java)
    - [1.1.3. Konfigurasi Tomcat](#113-konfigurasi-tomcat)
    - [1.1.4. Start Tomcat Server](#114-start-tomcat-server)
    - [1.1.5. Akses Web UI Port 8080](#115-akses-web-ui-port-8080)
  - [1.2. Integrasi Tomcat dengan Jenkins](#12-integrasi-tomcat-dengan-jenkins)
    - [1.2.1. Install plugin "Deploy to container"](#121-install-plugin-deploy-to-container)
    - [1.2.2. Konfigurasi Tomcat dengan Credentials](#122-konfigurasi-tomcat-dengan-credentials)
    - [1.2.3. Membuat Jenkins Job Baru](#123-membuat-jenkins-job-baru)
  - [1.3. Deploy Artifact ke Tomcat Server](#13-deploy-artifact-ke-tomcat-server)
  - [1.4. Automatic Build menggunakan Poll SCM](#14-automatic-build-menggunakan-poll-scm)

# 1. Integrasi Tomcat Server dan CI/CD
## 1.1. Instalasi Tomcat
### 1.1.1. Menyiapkan VM
Gunakan VM Ubuntu 20.04 yang telah disiapkan.


### 1.1.2. Install Java
Install Java

    ```
    $ sudo apt install openjdk-11-jre
    ```

### 1.1.3. Konfigurasi Tomcat
1. Pindah ke direktori ```/opt```.
    ```
    $ cd /opt/
    ```
2. Download [Tomcat](https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65.tar.gz)
    ```
    $ sudo wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65.tar.gz
    ```
3. Extract Tomcat.
    ```
    $ sudo tar -xvzf apache-tomcat-9.0.65.tar.gz
    $ sudo mv apache-tomcat-9.0.65.tar.gz tomcat
    ```

### 1.1.4. Start Tomcat Server
1. Untuk menjalankan tomcat 
    ```
    $ cd tomcat/bin
    $ sudo ./startup.sh
    ```
2. Untuk mematikan Tomcat 
    ```
    $ sudo ./shutdown.sh
    ```

### 1.1.5. Akses Web UI Port 8080
1. Kunjungi server Tomcat dari web browser dan pilih menu ```Manage App```.
2. Edit file ```/opt/tomcat/webapps/host-manager/META-INF/context.mxl```
 
    Tambahkan comment ```<!-- ... -->``` pada bagian yang hanya meng-_allow_ localhost.

3. Edit file ```/opt/tomcat/webapps/manager/META-INF/context.mxl```
   
   Tambahkan comment ```<!-- ... -->``` pada bagian yang hanya meng-_allow_ localhost.

4. Edit file ```/opt/tomcat/conf/tomcat-users.xml```
    ```
        <role rolename="manager-gui"/>
        <role rolename="manager-script"/>
        <role rolename="manager-jmx"/>
        <role rolename="manager-status"/>
        <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
        <user username="deployer" password="deployer" roles="manager-script"/>
        <user username="tomcat" password="s3cret" roles="manager-gui"/>
    ``` 

5. Restart Tomcat
    ```
    $ cd /opt/tomcat/bin
    $ sudo ./shutdown.sh
    $ sudo ./startup.sh
    ```
6. Cek kembali ke Tomcat Web UI. Klik menu ```Manage App``` dan masukkan user ```admin``` atau ```tomcat``` dengan password seperti di ```tomcat-users.xml```.

## 1.2. Integrasi Tomcat dengan Jenkins
### 1.2.1. Install plugin "Deploy to container"
1. Pilih ```Dashboard --> Manage Jenkins```.
2. Install plugin ```Deploy to container```.

### 1.2.2. Konfigurasi Tomcat dengan Credentials
1. Pilih menu ```Manage Jenkins --> Manage Credentials```.
2. Pilih ```Jenkins --> Global credentials (unrestricted)```.
3. Pilih menu ```Add Credentials```.
4. Gunakan username & password dari user yang memiliki role ```manager-script```.
    - username: deployer
    - password: deployer
    - ID: tomcat-deployer

### 1.2.3. Membuat Jenkins Job Baru
1. Kembali ke ```Dasboard```.
2. Pilih menu ```New Item```.
    - Name: BuildAndDeployJob
    - Type: Maven project
3. Setting konfigurasi:
    - Git: isikan URL git repository
    - branches to build: */master
    - Build --> Goals and options: clean install
    - Post-build Actions: Deploy war/ear to a container
      - WAR/EAR files: \*\*/\*.war\*\*
      - Containers: Tomcat 8.x Remote
        - Credentials: pilih ```tomcat-deployer```.
        - Tomcat URL: URL lokasi server tomcat
4. Jalankan Jenkins Job. Cek build log untuk memastikan job berhasil.
5. Browse server Tomcat. Pastikan ada project baru ```webapp``` yang ditambahkan.

## 1.3. Deploy Artifact ke Tomcat Server
1. Edit file ```webapp/src/main/index.jsp```.
2. Jalankan job ```BuildAndDeployJob```.

## 1.4. Automatic Build menggunakan Poll SCM
1. Pilih job ```BuildAndDeployJob```.
2. Pilih ```Configure```.
3. Setting ```Build Triggers```. Centang ```Poll SCM```.
    - Schedule: ```* * * * *```
4. Edit kembali source code-nya.
5. Cek job ```BuildAndDeployJob``` akan dijalankan secara otomatis.