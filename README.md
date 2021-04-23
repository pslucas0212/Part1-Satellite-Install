# DRAFT - Satellite Installation Instructions - DRAFT

In this guide, I'm documenting the steps for a "lab" install of Satellite 6.9.  The infrastructure is build on small lab vSphere 6.7 environment which has internet access for the installation.

- updated 2021-04-22

### Pre-Reqs

- For this example, I created a VM on vSphere with 4 vCPUS, 20GB RAM and 400GB "local" drive

- Install RHEL 7.9
  - Install all patches on your RHEL 7.9 instance
        
          # yum -y update


- **Note:** If you want to have Satellite grab an IP address via DHCP and automatically udpate your DNS with your provisioned RHEL instance, you'll need to store the need to make the *** DHCP and the DNS forward/reverse zone files available to Satellite on a shared drive. 

- Configured nfs client shared mount point for external DNS and DHCP
  - create shared DNS and DHCP config directory.  ex. /mnt/satshare
  - update /etc/fstab file with the line and reload systemctl daemon
      
        ds01.example.com:/volume2/DNS_DHCP_SHARE	/mnt/satshare	nfs	defaults	0 0
        systemctl daemon-reload
        
  - mount shared folder

        mount -t nfs ds01.example.com:/volume2/DNS_DHCP_SHARE /mnt/satshare
        
        
       

- Update firewall rules
  
        # firewall-cmd \
        --add-port="80/tcp" --add-port="443/tcp" \
        --add-port="5647/tcp" --add-port="8000/tcp" \
        --add-port="8140/tcp" --add-port="9090/tcp" \
        --add-port="53/udp" --add-port="53/tcp" \
        --add-port="67/udp" --add-port="69/udp" \
        --add-port="5000/tcp"

    - Make the changes permanent

          # firewall-cmd --runtime-to-permanent

    - Verify the firewall changes

          # firewall-cmd --list-all

- Check host name and local DNS resolution

        # ping -c3 localhost

        # ping -c3 `hostname -f`

- Set static and transient hostname

        # hostnamectl set-hostname sat01.example.com

- Register Satellite Server to RHSM

        # subscription-manager register --org=<org id> --activationkey=<activation key>
        
    - This both registers the server and attaches a Satellite Infrastructure subscription to the  Server
    
    - You can verify the subscription
        
          # subscription-manager list --consumed
       

- Config Repos

    - Disable all repos
    
          # subscription-manager repos --disable "*"
          
    - enable the following repos
    
          # subscription-manager repos --enable=rhel-7-server-rpms \
          --enable=rhel-7-server-satellite-6.8-rpms \
          --enable=rhel-7-server-satellite-maintenance-6-rpms \
          --enable=rhel-server-rhscl-7-rpms \
          --enable=rhel-7-server-ansible-2.9-rpms

    - Clear any meta data
    
          # yum clean all
          
    - Verify repos enabled
    
          # yum repolist enabled
          
- Install Satellite Server packages
 
        # yum update
        
        # yum install satellite

- Insall SOS package on base OS

        # yum install sos
        
- Setup System Clock with chrony.  I have a stratum 0 time server that my sytems use for synching time.  Type the following command to check the the time synch status (I like the verbose option)

          # chronyc sources -v
          
     - The system is prepped now and you may want to take a snapshot if running in a virtualized environment

- We will intially run the satellite-installer to create a userid and password and generate an answer file.  

          # satellite-installer --scenario satellite \
          --foreman-initial-admin-username admin \
          --foreman-initial-admin-password Passw0rd!
          
- Rerun the satellite-install to create DNS and DHC services to support provisioing from Satellite

        # satellite-installer --scenario satellite \
        --foreman-proxy-dns true \
        --foreman-proxy-dns-managed true \
        --foreman-proxy-dns-interface ens192 \
        --foreman-proxy-dns-zone example.com \
        --foreman-proxy-dns-forwarders 10.1.1.254 \
        --foreman-proxy-dns-reverse 10.1.10.in-addr.arpa \
        --foreman-proxy-dhcp true \
        --foreman-proxy-dhcp-managed true \
        --foreman-proxy-dhcp-interface ens192 \
        --foreman-proxy-dhcp-range "10.1.10.98 10.1.10.148" \
        --foreman-proxy-dhcp-gateway 10.1.10.1 \
        --foreman-proxy-dhcp-nameservers 10.1.10.200 

### Resources
- [Installing Satellite Server from a Connected Network](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.9/html/installing_satellite_server_from_a_connected_network/index)
