# Satellite Installation Instructions

1. Install RHEL 7.9
2. Update firewall rules
  
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

4. Check host name and local DNS resolution

        # ping -c3 localhost

        # ping -c3 `hostname -f`

5. Set static and transient hostname

        # hostnamectl set-hostname sat01.example.com

6. Register Satellite Server to RHSM

        # subscription-manager register --org=14029827 --activationkey=rhel_premium
        
    - This both registers the server and attaches a Satellite Infrastructure subscription to the  Server
    
    - You can verify the subscription
        
          # subscription-manager list --consumed
       

7. Config Repos

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
          
8. Install Satellite Server packages
 
        # yum update
        
        # yum install satellite

9. Insall SOS package on base OS

        # yum install sos
        
10.  Setup System Clock with chrony.  I have a startum 0 time server that my sytems use.  Type the following command to check the the time synch status (I like the verbose option)

          # chronyc sources -v
          
     - The system is prepped now and you may want to take a snapshot if running in a virtualized environment

11. We will intially run the satellite-installer to create a userid and password and generate an answer file.  

          # satellite-installer --scenario satellite \
          --foreman-initial-admin-username admin \
          --foreman-initial-admin-password Passw0rd!
          
11. Rerun the satellite-install to create DNS and DHC services to support provisioing from Satellite

        # satellite-installer --scenario capsule \
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
