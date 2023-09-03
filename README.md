# rke arm64 setup guide    
- raspberry pi imager flash 32gb-fat san disk sd && 32gb-fat san disk usb-3.0 speeds type with raspOS(64-bit)lite  
- (pi tip) - sudo rpi-eeprom-config --edit => BOOT_ORDER=0xf14 (1-usb 4-sd f-fallback_boot)  
- (k3s tip) - /boot/cmdline.txt => cgroup_memory=1 cgroup_enable=memory ip=LOCAL_IP_ADDRESS::LOCAL_IP_GATEWAY:NET_MASK_255:HOSTNAME:NIC_eth0:off  
  
==========  
jmux connect pi@pi-1.jabl3s.home pi@pi-2.jabl3s.home pi@pi-3.jabl3s.home pi@pi-4.jabl3s.home  
sudo apt update  
sudo apt full-upgrade  
sudo apt install apt-transport-https ca-certificates curl software-properties-common  
DOCKER INSTALL VERSION => https://www.suse.com/suse-rke1/support-matrix/all-supported-versions/rke1-v1-24/  
for rke 1.25 (jcluster.yml)=> sudo apt-get install docker-ce=5:20.10.24~3-0~debian-bullseye docker-ce=...  
sudo usermod -aG docker pi  
su - pi  
exit  
##  
jmux connect pi@pi-1.jabl3s.home  
scp jcluster.yml jclusterissuer.yml jnginx.conf and correct_rke_binary to master  
ssh-keygen -t rsa -b 4096 (empty passphrase)  
ssh-copy-id pi@pi-1.jabl3s.home    
ssh-copy-id pi@pi-2.jabl3s.home  
ssh-copy-id pi@pi-3.jabl3s.home  
ssh-copy-id pi@pi-4.jabl3s.home  
mv rke_linux-arm64 rke  
chmod +x rke  
./rke up  (spam run if fail <= 3-attempts)
## Helm approach to install cert-manager && rancher  
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null   
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/  
stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list  
sudo apt update  
sudo apt install helm  
## Kubectl  
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg  
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list  
sudo apt update && sudo apt install -y kubectl  
###  
mkdir -p ~/.kube  
cp kube_config_cluster.yml ~/.kube/config  
###  
## Cert-manager  
helm repo add jetstack https://charts.jetstack.io  
helm repo update  
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.crds.yaml  
helm install \  
  cert-manager jetstack/cert-manager \  
  --namespace cert-manager \  
  --create-namespace \  
  --version v1.12.0  
  
kubectl apply -f ~/jclusterissuer.yml  
  
helm install rancher rancher-stable/rancher \  
  --namespace cattle-system \  
  --set hostname=rancher.jabl3s.uk \  
  --set ingress.tls.source=letsEncrypt \  
  --set letsEncrypt.email=j@jabl3s.uk \  
  --set letsEncrypt.ingress.class=nginx  
  
exit  
  
## NGINX access  
jmux connect pi@pi-1.jabl3s.home pi@pi-2.jabl3s.home pi@pi-3.jabl3s.home pi@pi-4.jabl3s.home
curl -o ~/jnginx.conf https://raw.githubusercontent.com/jabl3s/rke/main/jnginx.conf?token=GHSAT0AAAAAACGQ7F63BBSFV5TJV7QE76S2ZHVAZRQ  
docker run --name jnginx-container -v ~/jnginx.conf:/etc/nginx/nginx.conf:ro -d -p 8080:80 --cpus 0.5 --memory 512m nginx  
  
########## ADDITINAL NOTES  
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
jnginx conf  
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
