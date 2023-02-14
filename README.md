 ## Configure an OpenVPN Server and Client on Ubuntu
Introduction
A virtual private network (VPN) provides a secure connection for users to access a private network remotely. This grants access to resources on the private network and prevents third parties from accessing sensitive information. In this hands-on lab, you will be tasked with configuring an OpenVPN server that includes a public key infrastructure (PKI) that is capable of receiving connections from an OpenVPN client.

 1. In one browser window or tab, open the Cloud Server OpenVPN Server.
 
 2. Log in to the server using the credentials provided:
 
  `` ssh cloud_user@<PUBLIC_IP_ADDRESS>   ``
 
 3. Create the certificate authority (CA) directory:
 
 `` sudo make-cadir /etc/openvpn/easy-rsa   `` 
 
 
4. Assume the root user role:

`` sudo -i   ``


5. Go into the openvpn/easy-rsa directory:

`` cd /etc/openvpn/easy-rsa   ``

6. Use the easy-rsa utility to initialize the PKI:

`` ./easyrsa init-pki   ``

7. Build the CA:

`` ./easyrsa build-ca   ``


8. Enter in a memorable passphrase for the CA key.

9. Enter in the common name:

`` openvpn-ca   `` 

10. Generate a request for the OpenVPN server, which we'll call vpnserver:

`` ./easyrsa gen-rq vpnserver nopass   ``

11. Generate a request for the OpenVPN client, which we'll call vpnclient:

`` ./easyrsa gen-rq vpnclient nopass   ``

12. When prompted to enter a common name, hit Enter.


13. Sign the certificate for the OpenVPN server:

`` ./easyrsa sign-req server vpnserver   ``


14. When prompted, enter yes to confirm request details.


15. When prompted, enter the passphrase for the CA key that you created previously.


16. Sign the certificate for the OpenVPN client:

`` ./easyrsa sign-req client vpnclient   ``

17. When prompted, enter yes to confirm request details.


18. When prompted, enter the passphrase for the CA key that you created previously.


19. Generate the Diffie-Hellman parameters:

`` ./easyrsa gend-h   ``

20. Copy the dh.pem, ca.crt, vpnserver.crt, and vpnserver.key to /etc/openvpn:

`` cp pki/dh.pem pki/ca.crt pki/issued/vpnserver.crt pki/private/vpnserver.key /etc/openvpn   ``


21. Copy the ca.crt, vpnclient.crt, and key to the /home/cloud_user directory:

`` scp pki/ca.crt pki/issued/vpnclient.crt pki/private/vpnclient.key cloud_user@10.0.1.102 /home/cloud_user   ``

22. When prompted, enter yes to continue connecting.

23. When prompted for a password, copy the cloud_user password from the lab credentials and paste it in.


## Configure the OpenVPN Server

1. Back out of being the root user and continue as the cloud_user:

`` exit   `` 

2. Copy the server.conf.gz file to the /etc/openvpn directory, and name the configuration file vpnserver.conf:

`` sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/ vpnserver.conf.gz   `` 

3. Unzip the configuration file:

`` sudo gzip -d /etc/openvpn/vpnserver.conf.gz   `` 

4. List the contents of the openvpn directory:

`` ll /etc/openvpn   You should see the vpnserver.conf file, certificates, Diffie-Hellman parameters, and VPN server key. `` 


5. Open the configuration file:

`` sudo vim /etc/openvpn/vpnserver.conf   `` 

6. Page down to the ca ca.crt line.


7. Update the certificate name and key:

`` cert vpnserver.crt key vpnserver.key   ``

8. Under Diffie hellman parameters., change the parameters file:

`` dh dh.pem   ``

9. Save and quit by pressing the Escape key and entering 

`` :wq. ``
10. Generate the TLS authentication (TA) key:

`` sudo openvpn --genkey --secret ta.key   ``

11. Move the TA key to the /etc/openvpn directory: 

`` sudo mv ta.key /etc/openvpn `` 

12. Copy the TA key to the VPN client:

 `` sudo scp /etc/openvpn/ta.key cloud_user@10.0.1.102:/home/cloud_user   ``
 
 
13. When prompted for a password, copy the cloud_user password from the lab credentials and paste it here.


14. View the status of ip_forward:

`` sudo sysctl -a | grep ip_forward   You should see that net.ipv4.ip_forward is set to 0. ``

15. Set the net.ipv4.ip_forward status to 1:

`` sudo sysctl -w net.ipv4.ip_forward=1   ``

16. View the status of ip_forward:

`` sudo sysctl -a | grep ip_forward   You should see that net.ipv4.ip_forward is set to 1 and is enabled. ``

17. Start and enable the openvpn service:

`` sudo systemctl enable openvpn@vpnserver --now   ``

18. Check the status of the openvpn service: 

`` sudo systemctl status openvpn@vpnserver   ``

## Configure the OpenVPN Client ##

1. Go to the browser window or tab with the OpenVPN Client terminal open.

2. Update the app cache:

 `` sudo apt update   `` 

3. Install the openvpn service:

`` sudo apt install openvpn -y   `` 


4. Copy the sample configuration file to the /etc/openvpn directory:

 `` sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn   `` 

5. Copy the certificates to /etc/opevpn:

``  sudo cp /home/cloud/user/*.crt /etc/openvpn   `` 


6. Copy the keys to /etc/opevpn:

`` sudo cp /home/cloud/user/*.key /etc/openvpn    `` 

7. View the contents of /etc/openvpn:

`` ll /etc/openvpn/   You should see client.conf, ca.crt, ta.key, vpnclient.crt, and vpnclient.key listed. `` 

8. Go back to the VPN Server terminal.

9. Check the IP address:

`` ip addr   The private IP address should be 10.0.1.101. ``


10. Copy the private IP address.


11. Go back to the VPN Client terminal.


12. Open the client configuration file:


`` sudo vim /etc/openvpn/client.conf   ``


13. Page down to the cert client.crt line.

14. Update the certificate name and key:

 `` cert vpnclient.crt key vpnclient.key   `` 


15. Page down to the end of the configuration file.


16. Type in /remote to search for the remote server.


17. Update the remote server:

 `` remote 10.0.1.101:1194   `` 


18. Save and quit by pressing Escape and entering :wq.


19. Start and enable the openvpn service:

`` sudo systemctl enable openvpn@client --now   ``

20. Check the status of the openvpn@client service:

 `` sudo systemctl status openvpn@client  ``


You should see the service is running and enabled.






















