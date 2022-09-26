- [1. Integrasi Ansible dalam CI/CD](#1-integrasi-ansible-dalam-cicd)
  - [1.1. Instalasi Server](#11-instalasi-server)
  - [1.2. Integrasi Ansible dengan Docker](#12-integrasi-ansible-dengan-docker)
    - [1.2.1. Konfigurasi pada Docker Server](#121-konfigurasi-pada-docker-server)
    - [1.2.2. Konfigurasi pada Ansible Server](#122-konfigurasi-pada-ansible-server)
  - [1.3. Integrasi Ansible dan Jenkins](#13-integrasi-ansible-dan-jenkins)
    - [1.3.1. Konfigurasi SSH Server](#131-konfigurasi-ssh-server)
    - [1.3.2. Membuat Jenkins Job Baru](#132-membuat-jenkins-job-baru)
  - [1.4. Build dan Create Container di Server Ansible](#14-build-dan-create-container-di-server-ansible)
    - [1.4.1. Install Docker di ``ansible-server``](#141-install-docker-di-ansible-server)
    - [1.4.2. Build Docker Image](#142-build-docker-image)
    - [1.4.3. Membuat Ansible Playbook](#143-membuat-ansible-playbook)
    - [1.4.4. Copy Image ke Dockerhub](#144-copy-image-ke-dockerhub)
  - [1.5. Jenkins Job untuk Build Image ke Ansible](#15-jenkins-job-untuk-build-image-ke-ansible)
  - [1.6. Menggunakan Ansible untuk Membuat Container](#16-menggunakan-ansible-untuk-membuat-container)
  - [1.7. Deploy Ansible playbook di Jenkins](#17-deploy-ansible-playbook-di-jenkins)


# 1. Integrasi Ansible dalam CI/CD

## 1.1. Instalasi Server

1. Gunakan VM yang telah disiapkan.
2. Tambahkan user ``ansadmin`` ke dalam ansible-server.
    ```
    $ sudo su -
    $ useradd ansadmin
    $ passwd ansadmin
    $ mkdir /home/ansadmin
    $ chown -R ansadmin:ansadmin /home/ansadmin/
    $ chsh -s /bin/bash ansadmin
    ```
3. Tambahkan user ke dalam ``sudoers``.
    ```
    $ visudo
    ```
    Tambahkan user ``ansadmin`` sebagai sudoers.
    ```
    # User privilege specification
    root    ALL=(ALL:ALL) ALL
    ansadmin    ALL=(ALL:ALL)   NOPASSWD: ALL
    ```
4. Login ke ``ansible-server`` menggunakan user ``ansadmin``.
5. Kemudian buat key baru dengan menggunakan akun ``ansadmin``.
    ```
    $ ssh-keygen
    ```
    Dalam pembuatan key, kita tidak perlu memasukkan passphrase.  
    File ``/home/ansadmin/.ssh/id_rsa.pub`` akan kita copy ke server tujuan yang di-manage oleh ansible nantinya.
6. Install ansible dengan menggunakan login ``root``.
    ```
    $ sudo su -
    $ apt install ansible
    ```
    Tes ansible dan Python yang berhasil diinstal.
    ```
    $ python3 --version
    $ ansible --version
    ```

## 1.2. Integrasi Ansible dengan Docker

### 1.2.1. Konfigurasi pada Docker Server

1. Login ke ``docker-server``.
2. Buat user baru dengan nama ``ansadmin``.
    ```
    $ sudo su -
    $ useradd ansadmin
    $ passwd ansadmin
    $ mkdir /home/ansadmin
    $ chown -R ansadmin:ansadmin /home/ansadmin
    $ chsh -s /bin/bash ansadmin
    ```
3. Tambahkan user ``ansadmin`` ke dalam sudoers.
    ```
    $ visudo
    ```
    Tambahkan user ``ansadmin`` sebagai sudoers.
    ```
    # User privilege specification
    root    ALL=(ALL:ALL) ALL
    ansadmin    ALL=(ALL:ALL)   NOPASSWD: ALL
    ```

### 1.2.2. Konfigurasi pada Ansible Server

1. Edit file ``/etc/ansible/hosts``.  
    Tambahkan IP address ``docker-server`` ke dalamnya.
2. Login sebagai user ``ansadmin``.
    ```
    $ sudo su - ansadmin
    ```
3. Copy-kan key yang telah di-generate ke ``docker-server``.
    ```
    $ ssh-copy-id <IP docker-server>
    ```
4. Cek key yang telah di-copy di ``docker-server`` dengan cara ssh ke ``docker-server``.
5. Tes ping dari ``ansible-server`` ke ``docker-server``.
    ```
    $ ansible all -m ping
    ``` 

## 1.3. Integrasi Ansible dan Jenkins

### 1.3.1. Konfigurasi SSH Server

1. Akses web Jenkins dan pilih menu ``Manage Jenkins --> Configure Systems``.
2. Pda bagian ``Publish over SSH``, tambahkan SSH Server baru ansible.
    - Name: ansible-server
    - Hostname: IP ansible server
    - username: ansadmin
    - password: 1234

### 1.3.2. Membuat Jenkins Job Baru

1. Buat job baru dengan nama ``CopyArtifactsToAnsible``.  
    Lakukan Copy from ``BuildAndDeployOnContainer``.
2. Disable ``Poll SCM`` (hilangkan centangnya).
3. Ubah ``SSH server`` ke ``ansible-server``.
4. Hapus bagian ``Exec command``.
5. Biarkan isi ``Remote directory`` tetap ``//opt//docker``.  
    Buat folder ``/opt/docker`` di dalam ``ansible-server``.
    ```
    $ sudo su - ansadmin
    $ sudo mkdir /opt/docker
    $ sudo chown ansadmin:ansadmin /opt/docker
    ```
6. Jalankan job.  
    Cek hasil job di console output dan folder /opt/docker di ``ansible-server``.

## 1.4. Build dan Create Container di Server Ansible

### 1.4.1. Install Docker di ``ansible-server``

1. Install docker di ``ansible-server``.
2. Tambahkan user ``docker`` ke dalam grup ``ansadmin``.
    ```
    $ usermod -aG docker ansadmin
    ```

### 1.4.2. Build Docker Image

1. Buat file ``/opt/docker/Dockerfile`` baru di ``ansible-server``
2. Copy konten ``Dockerfile`` dari ``docker-server`` ke file ``/opt/docker/Dockerfile`` di ``ansible-server``.
3. Lakukan build.
    ```
    $ cd /opt/docker
    $ docker build -t regapp:v1 .
    ```
4. Jalankan container baru.
    ```
    $ docker run -it --name regapp-server -p 8081:8080 regapp:v1
    ```
5. Tes aplikasi dengan browser.

### 1.4.3. Membuat Ansible Playbook

1. Edit file ``/etc/ansible/hosts/``.
    ```
    [dockerhost]
    192.168.0.160

    [ansible]
    192.168.0.166
    ```
2. Copy ssh-key ke server ansible sendiri.
    ```
    $ ssh-copy-id localhost
    ```
    Tes menggunakan perintah ``ansible all -a uptime`` dan pastikan semua server terdeteksi UP.
3. Buat file baru ``/opt/docker/regapp.yml``.
4. Tes dengan menggunakan perintah berikut
    ```
    $ ansible-playbook regapp.yml --check
    ```
    Pastikan tidak ada error.
5. Jalankan playbook dengan menggunakan perintah berikut.
    ```
    $ ansible-playbook regapp.yml
    ```
    Cek image yang berhasil dibuat dengan perintah ```docker images```.

### 1.4.4. Copy Image ke Dockerhub

1. Register ke [hub.docker.com](https://hub.docker.com/)
2. Setting username dan password Dockerhub di ``ansible-server``.  
    Pastikan menjalankan perintah berikut sebagai user ``ansadmin``.
    ```
    $ docker login
    ```
    Masukkan username dan password akun kita.
3.  Update docker image name.
    ```
    $ docker images
    $ docker tag <IMAGE ID> <USERNAME>/regapp:latest
    $ docker push <USERNAME>/regapp:latest
    ```
4. Cek Dockerhub untuk melihat apakah image kita sudah berhasil terupload.

## 1.5. Jenkins Job untuk Build Image ke Ansible

1. Edit file ``/opt/docker/regapp.yml`` di ``ansible-server``.
    ```
    ---
    - hosts: ansible
    
    tasks: 
    - name: create docker image
        command: docker build -t regapp:latest .
        args:
        chdir: /opt/docker
    
    - name: create tag to push image onto dockerhub
        command: docker tag regapp:latest munif/regapp:latest
    
    - name: push docker image
        command: docker push munif/regapp:latest
    ```
2. Buka web Jenkins dan update job ``CopyArtifacToAnsible``.
    - Exec command
        ```
        ansible-playbook /opt/docker/regapp.yml
        ```
3. Aktifkan kembali ``Poll SCM`` dengan schedule ``* * * * *``.  
    Kemudian klik ``Apply`` dan ``Save``.
4. Edit project dan commit untuk mentrigger Jenkins.  
    Cek dockerhub untuk memastikan bahwa image yang baru berhasil di-push.

## 1.6. Menggunakan Ansible untuk Membuat Container

1. Buat file baru ``/opt/docker/deploy_regapp.yml``.
2. Cek deployment Ansible.
    ```
    $ ansible-playbook deploy_regapp.yml --check
    ```
3. Login ke ``docker-server``. Hapus semua Docker container dan Docker images.
    ```
    $ sudo su -
    $ docker ps -a
    $ docker rm -f <CONTAINER ID>
    $ docker image prune
    $ docker rmi <IMAGE ID>
    ```
4. Ubah permission file berikut.
    ```
    $ chmod 777 /var/run/docker.sock
    ```
5. Kembali lagi ke ``ansible-server``. Jalankan ansible playbook.
    ```
    $ ansible-playbook deploy_regapp.yml
    ```
    Tunggu sampai proses selesai.
6. Cek hasil pembuatan container di ``docker-server``.
7. Buka website aplikasi yang telah berhasil di-deploy ``http://<IP docker server>:8082/webapp``.

## 1.7. Deploy Ansible playbook di Jenkins

1. Edit file ``deploy_regapp.yml`` (contoh: ada di file ``deploy_regapp2.yml``).
2. Tes playbook dengan menggunakan perintah ``ansible-playbook deploy_regapp.yml --check``.
3. Apabila tidak ada error, jalanan playbook dengan perintah ``ansible-playbook deploy_regapp.yml``.
4. Buka Jenkins Dashboard dan edit job ``CopyArtifactToDocker``.
5. Edit ``Exec command``.
    ```
    ansible-playbook /opt/docker/regapp.yml;
    sleep 10;
    ansible-playbook /opt/docker/deploy_regapp.yml;
    ```
6. Cek hasil deployment dan website hasilnya.