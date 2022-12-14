### spring-boot-kube
Sample Spring Boot Application to Deploy on Kubernetes

If you get below error while starting minikube with podman, follow steps after the error message to fix this issue -
[spokhyan@fedora ~]$ minikube start --driver=podman --container-runtime=cri-o
😄  minikube v1.26.0 on Fedora 36
✨  Using the podman driver based on user configuration

💣  Exiting due to PROVIDER_PODMAN_NOT_RUNNING: "sudo -n -k podman version --format " exit status 1: sudo: a password is required
💡  Suggestion: Add your user to the 'sudoers' file: 'user ALL=(ALL) NOPASSWD: /usr/bin/podman' , or run 'minikube config set rootless true'
📘  Documentation: https://podman.io

Note this has worked on fedora



On Fedora, it is the wheel group the user has to be added to, as this group has full admin privileges. Add a user to the group using the following command:

$ sudo usermod -aG wheel username

If adding the user to the group does not work immediately, you may have to edit the /etc/sudoers file to uncomment the line with the group name:

$ sudo visudo
...
%wheel ALL=(ALL) ALL

##### Allows people in group wheel to run all commands - This didn't work for me
%wheel  ALL=(ALL)       ALL

 
##### Same thing without a password - Uncomment below line to work - worked for me
%wheel  ALL=(ALL)       NOPASSWD: ALL

Did you get below error ?

[spokhyan@fedora ~]$ kubectl get pods
NAME                               READY   STATUS             RESTARTS   AGE
spring-boot-kube-65486bd84-kmwxd   0/1     ImagePullBackOff   0          79s
spring-boot-kube-65486bd84-r8wtf   0/1     ImagePullBackOff   0          79s

Follow these steps to resolve -

unresolved -- dropped podman due to known issue with client vs server incompatibility. Switched to Docker.

## Setup

### Docker 

If docker service is not installed, follow these steps -

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

##### set up the Docker repository

$ sudo dnf -y install dnf-plugins-core

$ sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo

##### Install Docker Engine

$ sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin

##### Start Docker

$ sudo systemctl start docker

### Kubernetes

$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

$ ls -l

$ sudo install minikube-linux-amd64 /usr/local/bin/minikube

#### Configure Minikube using standard Docker driver - docker provides standrad and rootless configuration

[Optional] set docker as default driver:

$ minikube config set driver docker

Start a cluster using the docker driver:

$ minikube start --driver=docker

##### If you get below error then follow subsequest steps to configure group and add user. 

bomb Exiting due to PROVIDER_DOCKER_NEWGRP: "docker version --format -" exit status 1: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version": dial unix /var/run/docker.sock: connect: permission denied bulb Suggestion: Add your user to the 'docker' group: 'sudo usermod -aG docker $USER && newgrp docker'

Create the docker group - 

$ sudo groupadd docker

Add your user to the docker group - 

$ sudo usermod -aG docker $USER

### mysql
Login into mysql server as root user -
$ mysql -uroot -p 

List Users in DB
mysql> SELECT User,Host FROM mysql.user;

Create a new user for this project -

mysql> CREATE USER 'sbkube'@'localhost' IDENTIFIED BY 'Kube365test!';

If you get "ERROR 1819 (HY000): Your password does not satisfy the current policy requirements" error then follow these steps to debug and correct

mysql> SHOW VARIABLES LIKE 'validate_password%';

I got below output -
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.02 sec)

There are three levels of password validation policy enforced when Validate Password plugin is enabled:

    LOW Length >= 8 characters.
    MEDIUM Length >= 8, numeric, mixed case, and special characters.
    STRONG Length >= 8, numeric, mixed case, special characters and dictionary file.
My policy is Medium length 

So, let's try with following command

mysql> CREATE USER 'sbkube'@'localhost' IDENTIFIED BY 'Kube365test!';
Query OK, 0 rows affected (0.02 sec)

This worked!

####
Create a mysql-deployment.yaml file and apply using kubectl command to create a mysql db in k8.

[spokhyan@fedora spring-boot-kube]$ kubectl apply -f mysql-deployment.yaml

$ kubectl exec -it mysql-6c88c5fccd-pn2tg /bin/bash
bash-4.2# mysql -h mysql -u root -p
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kubeDB             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

[spokhyan@fedora spring-boot-kube]$ kubectl get svc
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP          108m
mysql                       ClusterIP   None            <none>        3306/TCP         104m
springboot-kube-mysql-svc   NodePort    10.99.218.117   <none>        8080:30412/TCP   6m3s
[spokhyan@fedora spring-boot-kube]$ minikube ip
192.168.49.2


https://kubernetes.io/docs/reference/kubectl/cheatsheet/
