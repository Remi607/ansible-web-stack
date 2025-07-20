Prerequisites:
4 VMs (2 HAProxy, 2 Backend) with RHEL-based OS (I used RHEL-9.6):
HAProxy 1 192.168.1.136 
HAProxy 2 192.168.1.139
Backend 1 192.168.1.137
Backend 2 192.168.1.138

VIP for HA in the playbook set to 192.168.1.140 (will be installed)

Network connectivity between all nodes
SSH access to all servers with root & pass 

A machine to run below playbooks and commands with ansible installed, with SSH access to the 4VMS mentioned above. 

1. Generate keyair: ssh-keygen -t rsa -b 4096
2. Get the public key: cat ~/.ssh/id_rsa.pub
3. Push the public key to other servers (replace Public key with your generated): 

for ip in 192.168.1.{136..139}; do
  ssh root@$ip <<'EOF'
mkdir -p /home/admin001/.ssh
cat > /home/admin001/.ssh/authorized_keys <<'KEY'
ssh-rsa 
PASTE PUBLIC KEY HERE IN THIS LINE
KEY
chown -R admin001:admin001 /home/admin001/.ssh
chmod 700 /home/admin001/.ssh
chmod 600 /home/admin001/.ssh/authorized_keys
echo "admin001 ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/admin001
sudo chmod 440 /etc/sudoers.d/admin001
EOF
done
4. Run the playbooks (ammend ansible_project_name to your named location)
cd ansible_project_name/playbooks/
ansible-playbook -i ../inventory/hosts.ini site.yml -b

After deployment tests:

1. Access the website: https://192.168.1.140 
and http://192.168.1.140 (check if it is redirecting to https)
also check: https://192.168.1.140/app/hello.jsp (to get the Hello from Tomcat!)
2. Check admin001 login using SSH key run:
ssh admin001@192.168.1.136 
(check other servers ending: 137/138/139) and check if it logs in without password. 
3. To Check HA Setup:
run: 
curl -k https://192.168.1.140 
(you should get a response form Apache Web Server)
then run: 
ssh -t admin001@192.168.1.136 "sudo systemctl stop keepalived; curl -k https://192.168.1.140" 
(you should get a response form Apache Web Server, because haproxy2 is still running)
then run:
ssh -t admin001@192.168.1.139 "sudo systemctl stop keepalived; curl -k https://192.168.1.140" 
(response: Failed to connect to 192.168.1.140 port 443: Connection refused)
