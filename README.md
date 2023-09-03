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
  