# rke1-arm64 setup guide        
  
## Pre-requisites   
- Assumes home dns is already running with remote machine root ssh access to all node types, (needs dns wildcard setup at some point)
- raspberry pi imager flash 32gb-fat san disk sd && 32gb-fat san disk usb-3.0 speeds type UHS-I only available with raspOS(64-bit)lite, potential nixos to replace raspbian to aid os state management via saltstack/ansible.  
- (pi tip) - sudo rpi-eeprom-config --edit => BOOT_ORDER=0xf14 (1-usb 4-sd f-fallback_boot)  
- (k3s tip) - /boot/cmdline.txt => cgroup_memory=1 cgroup_enable=memory ip=LOCAL_IP_ADDRESS::LOCAL_IP_GATEWAY:NET_MASK_255:HOSTNAME:NIC_eth0:off ((eth0:off is for auto configuration off))  
  
## Start kubernetes cluster intitialization with cluster wide packages/resorces needed for all apps     
  
jmux connect pi@pi-1.jabl3s pi@pi-2.jabl3s pi@pi-3.jabl3s pi@pi-4.jabl3s  
sudo apt update  
sudo apt full-upgrade  
sudo apt install apt-transport-https ca-certificates curl software-properties-common  
DOCKER INSTALL VERSION => https://www.suse.com/suse-rke1/support-matrix/all-supported-versions/rke1-v1-24/  
for rke 1.25 (jcluster.yml)=> sudo apt-get install docker-ce=5:20.10.24~3-0~debian-bullseye docker-ce=...  
sudo usermod -aG docker pi  
su - pi  
exit  
##  
jmux connect pi@pi-1.jabl3s  
scp jcluster.yml jclusterissuer.yml jnginx.conf and correct_rke_binary to master  
ssh-keygen -t rsa -b 4096 (empty passphrase)  
ssh-copy-id pi@pi-1.jabl3s    
ssh-copy-id pi@pi-2.jabl3s  
ssh-copy-id pi@pi-3.jabl3s  
ssh-copy-id pi@pi-4.jabl3s  
mv rke_linux-arm64 rke  
chmod +x rke  
./rke up  (spam run if fail <= 3-attempts)  
## Kubectl cluster management tool (rancher cli at some point too maybe)         
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg  
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list  
sudo apt update && sudo apt install -y kubectl  
## Helm package manager approach to install initial cluster resources   
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null   
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/  
stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list  
sudo apt update  
sudo apt install helm  
###  
mkdir -p ~/.kube  
cp kube_config_cluster.yml ~/.kube/config  
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
helm install rancher rancher-stable/rancher --namespace cattle-system --create-namespace --set hostname=pi-4.jabl3s --set bootstrapPassword=admin --set ingress.tls.source=letsEncrypt --set letsEncrypt.email=j@jabl3s --set letsEncrypt.ingress.class=nginx    
  
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
  
=====  
  
## Aditional non-sense  
  
jmux connect pi@pi-1.jabl3s pi@pi-2.jabl3s pi@pi-3.jabl3s pi@pi-4.jabl3s  
curl -o ~/filename.ext -LJO https://raw.githubusercontent.com/jabl3s/rke/main/jnginx.conf     
docker run --name jnginx-container -v ~/jnginx.conf:/etc/nginx/nginx.conf:ro -d -p 80:80 --cpus 0.5 --memory 512m nginx  

kubectl get svc -n <namespace>

docker stop jnginx-container && docker rm jnginx-container
  
########## ADDITINAL commands    
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
Use Helm Hooks: If you originally installed Nginx Ingress using Helm, you can define a Helm hook to handle the update of the ConfigMap and restart the Controller pods. Here's an example:

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
This Helm hook defines a Kubernetes Job that deletes the Nginx Ingress Controller pods when the chart is upgraded or installed. Make sure to use an image with kubectl installed.
