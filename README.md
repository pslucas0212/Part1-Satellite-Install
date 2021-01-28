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
       
    - The system is prepped now and you may want to take a snapshot if running in a virtualized environment

   
10.  Setup System Clock with chrony.  I have a startum 0 time server that my sytems use.  Type the following command to check the the time synch status (I like the verbose option)

          # chronyc sources -v

