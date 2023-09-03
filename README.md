# rke  
scp cluster.yml and rke binary to master  
ssh-keygen -t rsa -b 4096 (empty passphrase)  
ssh-copy-id pi@192.168.3.11  
ssh-copy-id pi@192.168.3.12  
ssh-copy-id pi@192.168.3.13  
ssh-copy-id pi@192.168.3.14  
mv rke_binary rke  
chmod +x rke  
./rke up  
## Helm approach to install rancher  
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null  
sudo apt-get install apt-transport-https --yes  
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/  
stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list  
sudo apt-get update  
sudo apt-get install helm  
  
