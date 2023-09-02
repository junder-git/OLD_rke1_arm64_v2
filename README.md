# rke  
  
## Arm64 linux remote machine bash console to arm64 rke1 cluster - single point of failure kube master - rke requires docker on all machines. Note, jmux pretty much requires the same user and password on all host machines to run.   
  
## Step1) Download linux tools to automate rke1 setup from remote machine console   
### Step 1.1) - downloads and updates jmux dependencies  
sudo apt install curl sshpass tmux ssh-askpass ssh_copy_id git -y  
### Step 1.2) - downloads and updates jmux commands  
curl -o ~/jmux.sh https://raw.githubusercontent.com/jabl3s/jmux/main/jmux.sh && echo "source ~/jmux.sh" >> ~/.bashrc && source ~/.bashrc
## Step2) Get docker on all machines  
jmuxconnect username password host_server_1 host_server_2 host_server_3 (...atm as many vertical panes that fit) 
## Step2) install dependencies
sudo apt install git  
source ~/.env