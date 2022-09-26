- [1. Jenkins Intro](#1-jenkins-intro)
  - [1.1. Instalasi](#11-instalasi)
    - [1.1.1. Setup VM](#111-setup-vm)
    - [1.1.2. Install Java & Jenkins](#112-install-java--jenkins)
    - [1.1.3. Start Jenkins](#113-start-jenkins)
    - [1.1.4. Akses Web UI port 8080](#114-akses-web-ui-port-8080)
  - [1.2. Membuat Jenkins Jobs](#12-membuat-jenkins-jobs)
  - [1.3. Integrasi Git dengan Jenkins](#13-integrasi-git-dengan-jenkins)
    - [1.3.1. Install Git pada server Jenkins](#131-install-git-pada-server-jenkins)
    - [1.3.2. Install GitHub plugin pada Jenkins](#132-install-github-plugin-pada-jenkins)
  - [1.4. Menjalankan Jenkins untuk melakukan pull code dari GitHub](#14-menjalankan-jenkins-untuk-melakukan-pull-code-dari-github)
  - [1.5. Integrasi Maven dan Jenkins](#15-integrasi-maven-dan-jenkins)
    - [1.5.1. Setup Maven pada Jenkins Server](#151-setup-maven-pada-jenkins-server)
    - [1.5.2. Setup Environment Variables](#152-setup-environment-variables)
    - [1.5.3. Install Maven Plugin](#153-install-maven-plugin)
    - [1.5.4. Configure Maven dan Java](#154-configure-maven-dan-java)
  - [1.6. Build Java Project menggunakan Jenkins](#16-build-java-project-menggunakan-jenkins)


# 1. Jenkins Intro

## 1.1. Instalasi

### 1.1.1. Setup VM

Siapkan VM berbasis Ubuntu 20.04.

### 1.1.2. Install Java & Jenkins

Ikuti langkah-langkah instalasi pada website [Debian Jenkins Package](https://pkg.jenkins.io/debian-stable/)

### 1.1.3. Start Jenkins
1. Cek status service jenkins
    ```
    $ service jenkins status
    ```
2. Menjalankan jenkins
    ```
    $ sudo service jenkins start
    ```

### 1.1.4. Akses Web UI port 8080
1. Akses IP server Jenkins menggunakan browser http://192.168.0.159:8080/
2. Unlock Jenkins dengan menggunakan initial admin password
    ```
    $ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```
3. Masuk ke Jenkins menggunakan password initial tersebut.
4. Klik ikon user ```admin``` pada pojok kanan atas.
5. Pilih menu ```Configure```.
6. Ubah password pada bagian ```Password```.
7. Logout dan login kembali.


## 1.2. Membuat Jenkins Jobs
1. Pilih menu ```New Item```.
2. Beri nama item.
3. Beri deskripsi.
4. Pada bagian ```Build Steps -> Add build step``` pilih ```Execute shell```.
    Masukkan command berikut
    ```
    echo "Hello World"
    uptime
    ```
5. Pilih menu ```Build Now```
6. Klik icon hasil build untuk melihat console output dari proses build.

## 1.3. Integrasi Git dengan Jenkins

###  1.3.1. Install Git pada server Jenkins
Git sudah otomatis terinstal pada Ubuntu 20.04

### 1.3.2. Install GitHub plugin pada Jenkins
1. Kembali ke ```Dashboard```.
2. Pilih menu ```Manage Jenkins```.
3. Pilih menu ```Manage Plugins```.
4. Pilih tab ```Available``` dan kemudian cari ```github```.
5. Centang plugin ```GitHub```, kemudian pilih ```Install without restart```.
6. Kembali ke menu utama, pilih ```Manage Jenkins```. Kemudian pilih menu ```Global Tool Configuration```.
7. Pastikan ada menu ```Git``` pada ```Global Tool Configuration```. 

## 1.4. Menjalankan Jenkins untuk melakukan pull code dari GitHub
1. Create ```New Item```, dengan item name: ```PullCodeFromGitHub```.
2. Pada bagian ```Source Code Management```, konfigurasi repository yang akan ditarik kode programnya.
3. Pilih ```Apply``` kemudian ```Save```.
4. Jalankan job dan cek console outputnya.
    
    Kita juga bisa melihat di mana kode program didownload (```/var/lib/jenkins/workspace/PullCodeFromGitHub```).

## 1.5. Integrasi Maven dan Jenkins

### 1.5.1. Setup Maven pada Jenkins Server
1. Download Maven
    ```
    $ cd /opt
    $ sudo wget https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz
    $ sudo tar -xvzf apache-maven-3.8.6-bin.tar.gz
    $ sudo mv apache-maven-3.8.6 maven
    $ cd maven/bin
    $ ./mvn -v
    ```

### 1.5.2. Setup Environment Variables
1. Login sebagai root dengan ```sudo su -```.
2. Cek lokasi instalasi Java.
    ```
    $ find / -name jvm
    $ cd /usr/lib/jvm
    $ find / -name java-11*
    ```
2. Tambahkan environment variable pada ```~/.bash_profile```.
    ```
    M2_HOME=/opt/maven
    M2=/opt/maven/bin
    JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

    PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2

    export PATH
    ```

3. Jalankan perintah berikut untuk merefresh environment variable
    ```
    source .bash_profile
    ```

### 1.5.3. Install Maven Plugin
1. Kembali ke halaman utama/dashboard Jenkins.
2. Pilih ```Manage Plugins``` dan install ```Maven Integration```.

### 1.5.4. Configure Maven dan Java
1. Add JDK. Hilangkan centang install JDK.
    - Name: java-11
    - JAVA_HOME: /usr/lib/jvm/java-11-openjdk-amd64

2. Pilih ```Add Maven```. Hilangkan centang ```Install automatically```.
    - Name: maven-3.8.6
    - MAVEN_HOME: /opt/maven

3. Simpan konfigurasi plugin.

## 1.6. Build Java Project menggunakan Jenkins
1. Buat item baru dengan nama ```FirstMavenProject``` dengan pilihan ```Maven project```.
2. Tambahkan Git repository.
3. Pada bagian ```Build | Goal and options```, tambahkan parameter ```clean install```.
4. Jalankan job dengan cara mengeklik ```Build Now```.
5. Cek hasil build di ```Workspace```.
