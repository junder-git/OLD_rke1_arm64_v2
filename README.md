# rke  - rerunzzz
k3s - /boot/cmdline.txt cgroup_memory=1 cgroup_enable=memory ip=192.168.3.11::192.168.3.1:255.255.255.0:pi-1:eth0:off
==========
jmux connect pi@192.168.3.11 pi@192.168.3.12 pi@192.168.3.13 pi@192.168.3.14
sudo apt update
sudo apt full-upgrade
apt-get source linux-image-$(uname -r)
sudo apt install apt-transport-https ca-certificates curl software-properties-common
sudo add-apt-repository "deb [arch=arm64] https://download.docker.com/linux/ubuntu focal stable"
  
https://www.suse.com/suse-rke1/support-matrix/all-supported-versions/rke1-v1-24/  
  
sudo apt-get install docker-ce=5:20.10.24~3-0~debian-bullseye docker-ce=5:20.10.24~3-0~debian-bullseye
sudo usermod -aG docker pi  
su - pi  
exit  
##  
jmux connect pi@192.168.3.11
scp cluster.yml clusterissuer.yml and correct_rke_binary to master  
ssh-keygen -t rsa -b 4096 (empty passphrase)  
ssh-copy-id pi@192.168.3.11  
ssh-copy-id pi@192.168.3.12  
ssh-copy-id pi@192.168.3.13  
ssh-copy-id pi@192.168.3.14  
mv rke_binary rke  
chmod +x rke  
./rke up  
## Helm approach to install cert-manager && rancher  
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null  
sudo apt install apt-transport-https --yes  
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/  
stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list  
sudo apt update  
sudo apt install helm  
## Kubectl
sudo apt install -y apt-transport-https ca-certificates curl
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
  
kubectl apply -f clusterissuer.yml

helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=rancher.jabl3s.uk \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=j@jabl3s.uk \
  --set letsEncrypt.ingress.class=nginx


##########
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
## Rancher  



### CHATGPT
On ARM64-based systems, you should use kernel packages specifically built for that architecture. The package names for ARM64 kernels typically have "arm64" or "aarch64" in their names. To find the appropriate kernel package for your ARM64 system, you can run:  

apt-cache search linux-image | grep aarch64  

This command will list the available Linux kernel packages for ARM64 architecture on your system. Choose the one that matches your needs and install it accordingly. Make sure to select a package that is compatible with your ARM64 hardware.
  
