# DRAFT - Satellite Installation Instructions - DRAFT

In this guide, I'm documenting the steps for a "lab" install of Satellite 6.9.  The infrastructure is deployed to a small vSphere 6.7 lab environment which has internet access for the installation.  For this lab Satellite will provide DNS and DHCP services.  Satellite can be configured to work with ISC compliant DNS and DHCP services.

- updated 2021-10-06

### Pre-Reqs

- For this example, I created a VM on vSphere with 4 vCPUS, 20GB RAM and 400GB "local" drive

- Install RHEL 7.9.  For this example we have enabled SCA on the Red Hat Customer portal and do not need to attach a subscription.

- Register Satellite Server to RHSM
```
# sudo subscription-manager register --org=<org id> --activationkey=<activation key>
```
- You can verify the registration
```
# sudo subscription-manager status
```       
  Install all patches on your RHEL 7.9 instance:
```
# sudo yum -y update
```
 

- Update firewall rules
```
# sudo firewall-cmd \
--add-port="80/tcp" --add-port="443/tcp" \
--add-port="5647/tcp" --add-port="8000/tcp" \
--add-port="8140/tcp" --add-port="9090/tcp" \
--add-port="53/udp" --add-port="53/tcp" \
--add-port="67/udp" --add-port="69/udp" \
--add-port="5000/tcp"
```

- Make the changes permanent
```
# sudo firewall-cmd --runtime-to-permanent
```

- Verify the firewall changes
```
# sudo firewall-cmd --list-all
```
- Setup System Clock with chrony.  I have a stratum 0 time server that my sytems use for synching time.  Type the following command to check the the time synch status (I like the verbose option)
```
# chronyc sources -v
```
- Check host name and local DNS resolution.  Use dig to test for and reverse lo
```
# ping -c3 localhost
# ping -c3 `hostname -f`
# dig sat01.example.com +short
# dig -x 10.1.10.251 +short
```    
- To avoid discrepancies with static and transient host names, set all the host names on the system 
```
# hostnamectl set-hostname sat01.example.com
```
#### Configure and enable repositories

- Disable all repos
```    
# sudo subscription-manager repos --disable "*"
```       
- enable the following repos
```    
# sudo subscription-manager repos --enable=rhel-7-server-rpms \
--enable=rhel-7-server-satellite-6.9-rpms \
--enable=rhel-7-server-satellite-maintenance-6-rpms \
--enable=rhel-server-rhscl-7-rpms \
--enable=rhel-7-server-ansible-2.9-rpms
```
- Clear any meta data
```    
# sudo yum clean all
```          
- Verify repos enabled
```    
# sudo yum repolist enabled
# sudo subscription-manager repos --list-enabled
```          

### Satellite Installation
- Install Satellite Server packages and the install Satellite
```
# sudo yum -y update       
# sudo yum install satellite
```
??? What's this for???
- Install SOS package on base OS
```
# sudo yum install sos
```

- We will intially run the satellite-installer to create a userid and password and generate an answer file.  
```
# sudo satellite-installer --scenario satellite \
--foreman-initial-admin-username admin \
--foreman-initial-admin-password Passw0rd!
```
- Rerun the satellite-install to create DNS and DHCP services to support provisioing from Satellite
```
# satellite-installer --foreman-proxy-dhcp true \
--foreman-proxy-dhcp-managed true \
--foreman-proxy-dhcp-gateway "10.1.10.1" \
--foreman-proxy-dhcp-interface "ens192" \
--foreman-proxy-dhcp-nameservers "10.1.10.254" \
--foreman-proxy-dhcp-range "10.1.10.149 10.1.10.199" \
--foreman-proxy-dhcp-server "10.1.10.254" \
--foreman-proxy-dns true \
--foreman-proxy-dns-managed true \
--foreman-proxy-dns-forwarders "10.1.1.254" \
--foreman-proxy-dns-interface "ens192" \
--foreman-proxy-dns-reverse "10.1.10.in-addr.arpa" \
--foreman-proxy-dns-server "127.0.0.1" \
--foreman-proxy-dns-zone "example.com" \
--foreman-proxy-tftp true \
--foreman-proxy-tftp-managed true
```



### Resources
- [Installing Satellite Server from a Connected Network](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.9/html/installing_satellite_server_from_a_connected_network/index)
