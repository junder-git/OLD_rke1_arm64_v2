# rke1-arm64 setup guide     
- Uses rke-v1.3.23 and docker 20.10...  ((latest and greatest seem to go bust))  
- Check for harvester arm releases in the coming future    
## Pre-requisites  
- Hardware: rpi-4b-8Gbram *X, sd card *X, usb *X, poe capeable router, poe-usb-c splitter *X; where X=4 in my case => (4devices * 5volts * 3amps = 20watts total which is a third of your typical filament light bulb at 60watts.  
- Network: Home router internal dns is already running with domain to subnet/vlan for all node hostnames ((my-case: 192.168.3.0/24 vlan: X)), (needs dns wildcard setup at some point as well as access from public dns)  
- Storage: Using rpi imager to flash 32gb san disk sd with raspbian-os && 32gb san disk usb-3.0 speeds type UHS-I with Ubuntu-22.04.03(64-bit) server. Both storage devices are to be configured with ssh user-pass, locale/timezone (gb) and hostnames pi-sd and pi-X respectively. Now poe power on from router settings page and check the boot priorities with ```sudo rpi-eeprom-config --edit``` if hostname on jmux connect dont match pi-X formatting.  
  
If in doubt, wipe all storage devices partitions and data to best of ability with zeros before using rpi-imager:    
``` bash
sudo gdisk /dev/sda  ==> o ==> w
```
``` bash
sudo dd if=/dev/zero of=/dev/sda bs=4M status=progress
```
Note, ive installed fibre/raid in the past too, to use with hypervisor from vmware with lvm flags.    
  
## Start kubernetes cluster intitialization with cluster wide packages    
``` bash
jmux connect pi@pi-1.jabl3s pi@pi-2.jabl3s pi@pi-3.jabl3s pi@pi-4.jabl3s
```
``` bash
sudo apt update
sudo apt upgrade
sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg  
```
DOCKER INSTALL SPECIFIC VERSION SEE:  
1)=> https://www.suse.com/suse-rke1/support-matrix/all-supported-versions/rke1-v1-26/  
2)=> https://docs.docker.com/engine/install/ubuntu/ list versions with => ```apt-cache madison docker-ce | awk '{ print $3 }'```  
In my case:  
``` bash
sudo apt install docker-ce=5:20.10.24~3-0~ubuntu-jammy docker-ce-cli=5:20.10.24~3-0~ubuntu-jammy containerd.io docker-buildx-plugin  
``` 
==> change between versions by apt removing and reinstall in use with pin file mod method listed below...        
``` bash
sudo nano /etc/apt/preferences.d/docker-pin
```
``` text
Package: docker-ce  
Pin: version 5:20.10.24~3-0~ubuntu-jammy     
Pin-Priority: 1000  
  
Package: docker-ce-cli  
Pin: version 5:20.10.24~3-0~ubuntu-jammy    
Pin-Priority: 1000
```
==> Ctrl+o  
==> Ctrl+x   
``` bash  
sudo usermod -aG docker pi
su - pi
docker --version   
```
``` bash
jmux connect pi@pi-1.jabl3s
curl -o ~/rke https://raw.githubusercontent.com/jabl3s/rke1-arm64/main/rke_linux-arm64-v1.3.23 && chmod +x rke
curl -o ~/cluster.yml https://raw.githubusercontent.com/jabl3s/rke1-arm64/main/jcluster.yml
curl -o ~/jclusterissuer.yml https://raw.githubusercontent.com/jabl3s/rke1-arm64/main/jclusterissuer.yml
curl -o ~/jnginx-configmap.yaml https://raw.githubusercontent.com/jabl3s/rke1-arm64/main/jnginx-configmap.yaml
```      
``` bash
ssh-keygen -t rsa -b 4096   
ssh-copy-id pi@pi-1.jabl3s    
ssh-copy-id pi@pi-2.jabl3s  
ssh-copy-id pi@pi-3.jabl3s  
ssh-copy-id pi@pi-4.jabl3s     
```
``` bash
./rke up
```
(spam run ./rke up if fail <= 3-attempts and use empty passphrase in key-gen command, rke notoriously fails a ton: https://github.com/rancher/rke/issues/2632#issuecomment-914315247 so my new practice will be to establish a 1 node rke cluster, and add nodes into the cluster with associated roles via uncommenting them one by one in the desired final cluster.yml with each rke up call being made per uncommented node.) Actually seems to be how they are progressing with it in rke2 as well, by adding nodes individually, perhaps to adress this very issue idunno.        
  
## Kubectl cluster management tool (rancher cli at some point too maybe)         
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/  
==> see package manager walk-through then do addtional two steps below:  
###  
mkdir -p ~/.kube  
cp kube_config_cluster.yml ~/.kube/config  
## Helm package manager tool and approach to install further initial cluster resources   
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null   
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/  
stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list  
sudo apt update  
sudo apt install helm  
   
###  
## Cert-manager package    
helm repo add jetstack https://charts.jetstack.io  
helm repo update  
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.crds.yaml  
helm install \  
  cert-manager jetstack/cert-manager \  
  --namespace cert-manager \  
  --create-namespace \  
  --version v1.12.0  
  
kubectl apply -f ~/jclusterissuer.yml  

## Rancher package  
helm install rancher rancher-stable/rancher --namespace cattle-system --create-namespace --set hostname=pi-1.jabl3s --set bootstrapPassword=admin --set ingress.tls.source=letsEncrypt --set letsEncrypt.email=j@jabl3s --set letsEncrypt.ingress.class=nginx    
  
kubectl get svc -n cattle-system  
  
## NGINX pacakge    
  
- Consider rancher helm install without specifying nginx to begin with  
  
((nginx and gzip config, issues in the past with compression techniques for performance and data security)) - gzip attack nginx     
kubectl get pods,svc,configmaps --namespace=ingress-nginx  
  
kubectl delete ingressclass nginx -n default  
kubectl delete validatingwebhookconfiguration ingress-nginx-admission -n default  
kubectl delete serviceaccount nginx-ingress -n default  
kubectl delete clusterrole nginx-ingress -n default  
kubectl delete clusterrolebinding nginx-ingress -n default  
  
  
kubectl apply -f https://raw.githubusercontent.com/jabl3s/rke/main/jnginx-configmap.yaml  
  
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx  
helm repo update  
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace default --set controller.configMapName=jnginx-configmap  
  
helm upgrade nginx-ingress ingress-nginx/ingress-nginx --namespace default --set controller.configMapName=jnginx-configmap 
  
## End initial cluster and package installation, now apply my own .yml files for my own apps to run :D    
  
# UNINSTALL:::   
  
./rke remove  ((make sure all nodes are connected to master))    
=== OR ===  
jmux connect pi@pi-1.jabl3s pi@pi-2.jabl3s pi@pi-3.jabl3s pi@pi-4.jabl3s   
``` bash
docker stop $(docker ps -aq) && docker rm -f $(docker ps -aq) && docker volume prune
```
``` bash
sudo apt purge -y docker-engine docker docker.io docker-ce docker-ce-cli docker-compose-plugin
sudo apt-get autoremove
```
  
=== AND ===  
./cluster.rkestate ==> Delete this file    
  
=====  
  
## Aditional non-sense  
Clear dns/curl cache:  
``` bash
sudo systemctl restart systemd-resolved
```  
jmux connect pi@pi-1.jabl3s pi@pi-2.jabl3s pi@pi-3.jabl3s pi@pi-4.jabl3s  
curl -o ~/filename.ext -LJO https://raw.githubusercontent.com/jabl3s/rke/main/jnginx.conf     
docker run --name jnginx-container -v ~/jnginx.conf:/etc/nginx/nginx.conf:ro -d -p 80:80 --cpus 0.5 --memory 512m nginx  

kubectl get svc -n <namespace>

docker stop jnginx-container && docker rm jnginx-container
  
########## ADDITIONAL commands  
docker ps -a -q | awk '{print $1}' | xargs docker stop 
docker ps -a  
docker ps -a --filter "name=jnginx-container"  

((update all pi nodes ubuntu os kernal))- sudo apt upgrade linux-generic  
sudo apt remove linux-image-generic  
sudo dpkg --remove --force-remove-reinstreq linux-image-6.2.0-31-generic  
sudo apt --fix-broken install  
sudo apt clean  
df -h
du -h myfolder
sudo apt update && sudo apt upgrade  
sudo apt-get dist-upgrade   
lsb_release -a  
##########  
jnginx-configmap.yaml  
  
  
NGINX PLUS FOR max_conns=3;   
https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/ See NGINX PLUS features    
        queue 100 timeout=70; NGINX PLUS FOR LIMMITING QUEUE  
        sticky learn  NGINX PLUS FOR SESSIONS  
            create=$upstream_cookie_examplecookie    
            lookup=$cookie_examplecookie  
            zone=client_sessions:1m  
            timeout=1h  
            sync;  
    
This allows the ability for the execute shell window to remain open for up to 15 minutes. Without this parameter, the default is 1 minute and will automatically close.
  
#########CHATGPT  
Use Helm Hooks: If you originally installed Nginx Ingress using Helm, you can define a Helm hook to handle the update of the ConfigMap and restart the Controller   pods. Here's an example:  
  
yaml  
Copy code  
apiVersion: batch/v1  
kind: Job  
metadata:  
  name: nginx-config-update  
  namespace: ingress-nginx  
spec:  
  template:  
    spec:  
      containers:  
        - name: nginx-config-update  
          image: your-image-with-kubectl  # An image with kubectl installed  
          command:  
            - kubectl  
            - delete  
            - pods  
            - -n  
            - ingress-nginx  
            - -l  
            - app.kubernetes.io/component=controller  
          restartPolicy: OnFailure  
This Helm hook defines a Kubernetes Job that deletes the Nginx Ingress Controller pods when the chart is upgraded or installed.  
Make sure to use an image with kubectl installed.  

nano /boot/config.txt
boot_order=0xf41
nano /boot/cmdline.txt
cgroup_memory=1 cgroup_enable=memory ip=LOCAL_IP_ADDRESS::LOCAL_IP_GATEWAY:NET_MASK_255:HOSTNAME:NIC_eth0:off

===  
- (pi tip for raspbian os) - sudo rpi-eeprom-config => BOOT_ORDER=0xf14 (1-usb 4-sd f-fallback_boot, saw vid where boot order was from the right not left, but hostname is of type usb so eh?!, jeff here https://youtu.be/UT5UbSJOyog?t=499 has the nvme usb boot figure at the end, but my pi is booting from left to right it seems...)  
- (k3s tip for raspbian os) - /boot/cmdline.txt => cgroup_memory=1 cgroup_enable=memory ip=LOCAL_IP_ADDRESS::LOCAL_IP_GATEWAY:NET_MASK_255:HOSTNAME:NIC_eth0:off ((eth0:off is for auto configuration off))  
