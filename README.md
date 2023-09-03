# rke  
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
  --version v1.12.0 \
  ### --set installCRDs=true
### if fail re-run
helm upgrade \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.12.0  
  
kubectl apply -f clusterissuer.yml
((update all pi nodes ubuntu os kernal))- sudo apt upgrade linux-generic  
## Rancher  




  
