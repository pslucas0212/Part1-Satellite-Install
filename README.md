# Satellite Installation Instructions

### Resources
- https://access.redhat.com/blogs/1169563/posts/3640721

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
        
10.  Setup System Clock with chrony.  I have a stratum 0 time server that my sytems use for synching time.  Type the following command to check the the time synch status (I like the verbose option)

          # chronyc sources -v
          
     - The system is prepped now and you may want to take a snapshot if running in a virtualized environment

11. We will intially run the satellite-installer to create a userid and password and generate an answer file.  

          # satellite-installer --scenario satellite \
          --foreman-initial-admin-username admin \
          --foreman-initial-admin-password Passw0rd!
          
11. Rerun the satellite-install to create DNS and DHC services to support provisioing from Satellite

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

12. On the customer portal create a manifest

13. On the Satellite UI create an organization - operations
    - Create a location - moline and add the location to the operations organzation

14. Ensure that your organization is set to operations and location is set to moline and import the manifest that you create on the customer portal

15. Enable RHEL 8 BaseOS, AppStream and RHEL Satellite Tools and sync the repos/products

16. Add organizations and locations to the domain target created in step 11.  
    - Infrastructure | Domains | example.com -> Locations - moline - Submit button -> Organizations - Operations - Submit button

17. Create Lifecyce Environment (LE)

        # hammer lifecycle-environment create --description le-ops-rhel8-prem-server --prior Library --name le-ops-rhel8-prem-server --organization "operations"
18. Create a Content View (CV)

        # hammer content-view create --description cv-rhel8-prem-server --name cv-rhel8-prem-server --organization "operations"
19. LIst Repository IDs

        # hammer repository list --organization operations
20. Add Repositories to Convent view
     
        # hammer content-view update --repository-ids 1,3,4 --name "cv-rhel8-prem-server" --organization "operations"
21. Publish Content to Library
      
        # hammer content-view publish --name "cv-rhel8-prem-server" --organization "operations" --async
22. Promote Content into Content View
        
        # hammer content-view version promote --content-view "cv-rhel8-prem-server" --to-lifecycle-environment "le-ops-rhel8-prem-server" --organization "operations" --async
23. Create an Activation Key

        # hammer activation-key create --content-view "cv-rhel8-prem-server" --lifecycle-environment "le-ops-rhel8-prem-server" --name "ak-ops-rhel8-prem-server" --organization "operations"
24.  List Available Subscriptions
 
         # hammer subscription list --fields id,name,quantity,consumed --organization "operations"
25. Add Subscriptions to the Activation key

        # hammer activation-key add-subscription --name "ak-ops-rhel8-prem-server" --subscription-id 7 --organization "operations"
26. Create subnet via the command line:
          
        hammer subnet create --name operations_subnet \
        --locations moline \
        --organizations operations \
        --domains example.com \
        --network 10.1.10.0 \
        --mask 255.255.255.0 \
        --dns-primary 10.1.10.200 \
        --from 10.1.10.98 \
        --to 10.1.10.148 \
        --dns sat01.example.com \
        --dhcp sat01.example.com \
        --boot-mode DHCP \
        --ipam DHCP

27. Create compute resource

28. Create a compute profile
    - Infrastructure | Compute Profiles | click Create Compute Profile button
    - Give the Compute Profile a name - "vmware-small" - click the submit button
    - For the vmware-small Compute Profle click the vSphere Lab DataCenter link (previously create Compute Resource).
    - Complete Attributes
        - Chose VMWare EXSi Cluster - LabCluster
        - Guest OS RHEL 8
        - Virtual H/W Version 14 EXSi 6.7
        
        
## Tempate Preparation in vMware ???? Move this section up to the front???



29. Create Empty Virtual Machine
    - Follow VMWare steps to create a New Virtual Machine - We are creating a base system from which we will create a template for future provisioning
    - When registering the system use the following command:
        
          # rpm -ivh http://sat01.example.com/pub/katello-ca-consumer-latest.noarch.rpm
          # subscription-manager register --org=operations --activationkey=ak-ops-rhel8-prem-server
          
 30. Get IDs for Operating System, Architecture and Compute Resrouce for the following step.
 
          # hammer os list; hammer architecture list; hammer compute-resource list
 
 31. Create Image in Satellite - 
          
          
          # hammer compute-resource image create --operatingsystem-id 2 --architecture-id 1 --compute-resource-id 1 --user-data true --uuid template-rhel8-cloudinit --username root --name img-rhel8-prem-server


## Continuing on
??.  See section on configuring provisioning templates

??.  Install Provisioning Templates

          # hammer template create --name vmware-cloud-init --file ~/vmware-cloud-init-template.erb --locations moline --organizations operations --operatingsystem-ids 1 --type cloud-init
          
          # hammer template create --name vmware-userdata --file ~/vmware-userdata-template.erb --locations loc-example --organizations org-example --operatingsystem-ids 1 --type user_data




32. Create a Compute Profile

33. Create a Host Group

34. ?? Setting up the Discovery Service for iPXE
    -  See section 5.1 -> https://access.redhat.com/documentation/en-us/red_hat_satellite/6.8/html-single/provisioning_guide/index#Configuring_Networking-Configuring_gPXE_to_Reduce_Provisioning_Times
