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


https://kubernetes.io/docs/reference/kubectl/cheatsheet/
