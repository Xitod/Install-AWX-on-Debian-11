# Install AWX on Debian 11 using k3s
Here is an easy setup to configure and deploy AWX Operator on a single node using k3s. This node will be use as an ansible master to monitore your infrastructure.

*This setup can be used in production environment.*
## Sources
- [awx-on-k3s from kurokobo](https://github.com/kurokobo/awx-on-k3s) (special thanks)
- [awx-operator](https://github.com/ansible/awx-operator)
- [k3s](https://k3s.io/)
- [Computing for geeks](https://computingforgeeks.com/how-to-install-ansible-awx-on-ubuntu-linux/)
## Environment
Minimum configuration required that I am used to. 
 - **Operating System** : Debian 11
 - **CPU** : 2 cpu cores
 - **RAM** : 4 Gb
 - **Disk space** : 30Gb (enought to run script ansible) 
## Preparation
### Space provisioning
I advise to provide /var directory more space than /home following this configuration :
- / : 8.5 Gb (boot)
- /var : 12Gb
- /tmp : 1Gb
- /home : 5Gb
- swap space : 1 Gb
### Required packages
Install required packages to deploy k3s and AWX.
```bash
apt install sudo make git curl gnupg jq
```
### Corporate proxy
In a corporate environment, you may be behind a proxy. I will try to give details to avoid a lost of time !
Use this command to setup your proxy environment.
```bash
export {http_proxy,https_proxy}=http://proxy.your.domain.local:PORT
```
Modify your file /etc/apt/apt.conf and add this lines.
```bash
Acquire::http::Proxy "http://proxy.your.domain.local:PORT";
Acquire::https::Proxy "https://proxy.your.domain.local:PORT";
```
*Of course, you can personalize the command following your configuration.*
## Installation
### Install Ansible
You can follow the [official documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) to install Ansible.
For Debian 11.
```bash
sudo echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu focal main" >> /etc/apt/source.list
```
Add the key to recognize the package and install Ansible.
```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
sudo apt update
sudo apt install ansible
```
### Install k3s
Install k3s. 
```bash
curl -sfL https://get.k3s.io | sudo bash - --write-kubeconfig-mode 644
```
*I advise you to add `--write-kubeconfig-mode 644` to make the k3s config file `/etc/rancher/k3s/k3s.yaml` readable by non-root user.*

Now you can verify is it working by checking the version.
```bash
k3s --version
```
### Deploy AWX Operator
We will create a namespace called "awx" for AWX instance.
```bash
export NAMESPACE=awx
kubectl create ns ${NAMESPACE}
```
Then, modify the default namespace with our new one.
```bash
kubectl config set-context --current --namespace=$NAMESPACE
```
*Be carefull, if you shutdown your machine, you have to re set the default namespace.* 

Deploy AWX operator 0.16.1 inside our k3s node. The AWX operator is usefull to run our future AWX instance.
```bash
cd ~
git clone https://github.com/ansible/awx-operator.git
cd awx-operator/
git checkout 0.16.1
make deploy
```
*If the version is not the latest of the [official repo](https://github.com/ansible/awx-operator), you can modify the command.*

Now, you should see pods running.
```bash
$ kubectl get pods
NAME                                               READY   STATUS    RESTARTS   AGE
awx-operator-controller-manager-8d479fd7dd-okt4k   2/2     Running   0          24s
```
### Prepare config file
Clone this repository .
```bash
cd ~
https://github.com/Xitod/awx-Debian-11.git
cd awx-Debian-11
```
Modify the hostname by yours in `config/deploy.yml`
```yaml
---
spec:
  ...
  ingress_type: ingress
  ##### To Modify #####
  hostname: your.domain.local
  ##### ~~~~~~~~~~ #####
  service_type: ClusterIP
  ...
```
Modify the passwords in `config/kustomization.yml`
```yaml
---
...
secretGenerator:
  # Secret for database
  - name: awx-prod-postgres-secret
  type: Opaque
  literals:
    - host=awx-postgres
    - port=5432
    - database=awx
    - username=awx
    - type=managed
    ##### TO Modify #####
    - password=DatabasePassword
    ##### ~~~~~~~~~~ #####
  # Secret for web interface
  - name: awx-prod-admin-secret
  type: Opaque
  literals:
    ##### To Modify #####
    - password=AdminPassword
    ##### ~~~~~~~~~~ #####
...
```
*Kustomization is a very usefull tool to customize kubernetes. It permit to simplify deployment on different environments (dev, pre-prod, prod) or modify variable and secrets without touching configuration yaml files.
You can visit the [official documentation](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/) to learn more about it.*

Create a directory for the persistence of datas, like writed in `/config/create-pv.yml`.
```bash
sudp mkdir -p /volume/projects
```
*Modify the user:group of the directory if needed.*

### Deploy AWX Instance
Deploy AWX using the option `-k` to specify kustomization deployment.
```bash
kubectl -k config
```
You can check live that the pods are running.
```bash
watch kubectl get pods
```
When everything has been deployed, you should see something like this output.
```bash
NAME                                               READY   STATUS    RESTARTS   AGE
awx-operator-controller-manager-8d479fd7dd-okt4k   2/2     Running   0          14m41
awx-postgres-0                                     1/1     Running   0          5m23
awx-7c83der4c1-zn782                               4/4     Running   0          4m
```
Now, you can access to your AWX instance using the hostname you specified.
`http://your.domain.local`
