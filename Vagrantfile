# Vagrant configuration file for multi machines with inter connectivity via SSH key based authentication
numnodes=3
baseip="192.168.10"
 
# global script
$global = <<SCRIPT
 
# Allow SSH to accept SSH password authentication. Find and replace if the line is commented out
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
 
# Add Google DNS to access internet. 
echo "nameserver 8.8.8.8" | sudo tee -a  /etc/resolv.conf 
 
# Download and install  Centos 7 EPEL package to configure the YUM repository
sudo rpm -ivh https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
# Update yum
#sudo yum update -y
# Install wget curl and sshpass
sudo yum install wget curl sshpass -y
 
# Exclude node* from host checking
cat > ~/.ssh/config <<EOF
Host node*
   StrictHostKeyChecking no
   UserKnownHostsFile=/dev/null
EOF
 
# Populate /etc/hosts with the IP and node names 
for x in {11..#{10+numnodes}}; do
  grep #{baseip}.${x} /etc/hosts &>/dev/null || {
      echo #{baseip}.${x} node${x##?} | sudo tee -a /etc/hosts &>/dev/null
  }
 
done
yes y |ssh-keygen -f /home/vagrant/.ssh/id_rsa -t rsa -N ''
echo " **** SSH Key Pair created for node$c ****"
 
SCRIPT
 
# SSH configuration script
$ssh = <<SCRIPT1
numnodes=3
 
for (( c=1; c<$numnodes+1; c++ ))
do
    echo "$c"
    echo "node$c"
    if [ "$HOSTNAME" = "node1" ]; then
      echo "**** Install ansible on node1 ****"
      sudo yum install ansible -y
    fi
    # Skip the current host.
    if [ "$HOSTNAME" = "node$c" ]; then
        echo "node$c"
        continue
    fi
 
    # Copy the current host's id to each other host.
    # Asks for password.
    # create ssh key
     
    sshpass -p vagrant ssh-copy-id "node$c"
    echo "**** Copied public key to node$c ****"   
done
 
# Get the id's from each host.
#for (( c=1; c<$numnodes+1; c++ ))
#do
#    # Skip the current host.
#    if [ "$HOSTNAME" = "node$c" ]; then
#        continue
#    fi
#
#    sshpass -p vagrant ssh "node$c" 'cat .ssh/id_rsa.pub' >> /home/vagrant/host-ids.pub
#    echo "**** Copy id_rsa.pub contentes to host-ids.pub for host node$c ****"
#done
 
#for (( c=1; c<$numnodes+1; c++ ))
#do
#    # Skip the current host.
#    if [ "$HOSTNAME" = "node$c" ]; then
#        continue
#    fi
# 
#    # Copy public keys to the nodes
#    sshpass -p vagrant ssh-copy-id -f -i /home/vagrant/host-ids.pub "node$c"
#    echo "**** Copy public keys to node$c ****"
 
#done
# Set the permissions to config
sudo chmod 0600 /home/vagrant/.ssh/config
# Finally restart the SSHD daemon
sudo systemctl restart sshd
echo "**** End of the Multi Machine SSH Key based Auth configuration ****"
 
SCRIPT1
 
# Vagrant configuration
Vagrant.configure("2") do |config|
  # Execute global script
  config.vm.provision "shell", privileged: false, inline: $global
  prefix="node"
  #For each node run the config and apply settings
  (1..numnodes).each do |i|
    vm_name = "#{prefix}#{i}"
    config.vm.define vm_name do |node|
      node.vm.box = "centos/7"
      node.vm.hostname = vm_name
      ip="#{baseip}.#{10+i}"
      node.vm.network "private_network", ip: ip    
	  node.vm.provider "virtualbox" do |a|
	    a.memory=4096
	end
    end
    # Run the SSH configuration script
    config.vm.provision "ssh", type: "shell", privileged: false, inline: $ssh
  end
end