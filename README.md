# Part 1: Satellite Installation   

[Tutorial Menu](https://github.com/pslucas0212/RedHat-Satellite-VM-Provisioning-to-vSphere-Tutorial)  

Red Hat Satellite is a powerful content management and provisioning tool that you can add to any Red Hat Enterprise Linux subscription with the addition of a Smart Management subscription.  With Red Hat Satellite you can curate specific content across mutliple lifecycle environments through out your entire RHEL enviroment whether it is on-prem, in the cloud or hybrid.  In fact you can use Red Hat Satellite with your market-place instances of RHEL.  

In this multi-part tutorial we will covering how to provision RHEL VMs to a vSphere environment from Red Hat Satellite.

In part 1, I'm documenting the steps for a simple "lab" install of Satellite 6.9.  The purpose of this setup is to give you a quick hands-on experience with Satellite.  The lab infrastructure is deployed to a small vSphere 6.7 lab environment with three EXSi servers that have internet access for the installation.  For this lab, Satellite will provide DNS and DHCP services for the network that is hosting vSphere environment.  Note: Satellite can be configured to work with ISC compliant DNS and DHCP services.  Also, in a production environment you would also want to configure Satellite to interact with your directory/security services.  

After Satellite is installed point your vSphere environment to the Satellite server for DNS services.  I would also recommend creating a local time server and pointing all system in this lab environment to the same local time source.


### Pre-Reqs


Create a VM for Satellite and install RHEL 7.9.  The VM was sized with 4 vCPUS, 20GB RAM and 400GB "local" drive.  Note: For this example I have enabled Simple Content Access on the Red Hat Customer portal and do not need to attach a subscription to the RHEL or Satellite repositories.  After you have created and started the RHEL 7.9 VM, we will ssh to the RHEL VM and work from the command line.

For this lab environment I chose sat01.example.com for the hostname of the server hosting Satellite.  

Check hostname and local DNS resolution.  Use dig to test forward and reverse lookup of the server hosgint Satellite.  If the Satellite hostname is not available from DNS, the intial installation will fail.  After installing Satellite, we will use the DNS service on Statellite.
```
# ping -c3 localhost
# ping -c3 `hostname -f`
# dig sat01.example.com +short
# dig -x 10.1.10.251 +short
```   

Register Satellite Server to RHSM.
```
# sudo subscription-manager register --org=<org id> --activationkey=<activation key>
```
You can verify the registration with following command.
```
# sudo subscription-manager status
```    
#### Configure and enable repositories

Disable all repos.
```    
# sudo subscription-manager repos --disable "*"
```       
Enable the following repositories.
```    
# sudo subscription-manager repos --enable=rhel-7-server-rpms \
--enable=rhel-7-server-satellite-6.9-rpms \
--enable=rhel-7-server-satellite-maintenance-6-rpms \
--enable=rhel-server-rhscl-7-rpms \
--enable=rhel-7-server-ansible-2.9-rpms
```
Clear any meta-data.   
```    
# sudo yum clean all
```          
Verify that repositories are enabled.  
```    
# sudo yum repolist enabled
# sudo subscription-manager repos --list-enabled
```          

#### Update the RHEL 7.9 instance and finish final pre-reqs
Install all patches on your RHEL 7.9 instance.
```
# sudo yum -y update
```
 
I would also recommend registering this server to Insights.  
```
# yum -y install insights-client
# insights-client --enable
```

Update the firewall rules for Satellite.
```
# sudo firewall-cmd \
--add-port="80/tcp" --add-port="443/tcp" \
--add-port="5647/tcp" --add-port="8000/tcp" \
--add-port="8140/tcp" --add-port="9090/tcp" \
--add-port="53/udp" --add-port="53/tcp" \
--add-port="67/udp" --add-port="69/udp" \
--add-port="5000/tcp"
```

Make the firewall changes permanent
```
# sudo firewall-cmd --runtime-to-permanent
```

Verify the firewall changes
```
# sudo firewall-cmd --list-all
```
Setup system Clock with chrony.  I have local time server that my sytems use for synching time.  Type the following command to check the the time synch status.  
```
# chronyc sources -v
```


### Satellite Installation
Install Satellite Server packages and then install Satellite.  
```     
# sudo yum install satellite
```

Install SOS package on base OS for intial systems analysis in case you need to collect problem determination for any system related issues.  
```
# sudo yum install sos
```

We will  run the satellite-installer to create a userid and password along with the information to configure the DNS, DHCP and TFTP services.  This will take several minutes to complete.  
```
# satellite-installer --scenario satellite \
--foreman-initial-admin-username admin \
--foreman-initial-admin-password Passw0rd! \
--foreman-proxy-dhcp true \
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

Use the following command to find the name of the Satellite server you just updated.
```
# hammer proxy list
```

See which services are configured on your Satellite server.  We want to verify that the DNS and DHCP services are enabled.
```
# hammer proxy info --name sat01.example.com
```

If recently add services such as DNS or DHCP are not part of the output from the previous command, try refreshing the Satellite features.
```
# hammer proxy refresh-features --name sat01.example.com
```
 

### Login into the Satellite console  

We can now launch and login to the Satellite console by entering [http://sat01.example.com](http://sat01.example.com) for the Satellite url.  Satellite will redirect the browser to Satellite's secure login page.  You will need to accept Satellite's certificate for your browser.  

For this example we are using a local login.  For production work you will want to integrate your directory service with Satelllite. Enter the user id and password and click Login button.  

![Click Login button](/images/sat01.png)  

You are now at the Satellite home screen.  

![Satellite Home Srceen](/images/sat02.png)  



## References  
[Installing Satellite Server from a Connected Network](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.9/html/installing_satellite_server_from_a_connected_network/index)   
[Simple Content Access](https://access.redhat.com/articles/simple-content-access)  
[Provisioning VMWare using userdata via Satellite 6.3-6.6](https://access.redhat.com/blogs/1169563/posts/3640721)  
[Understanding Red Hat Content Delivery Network Repositories and their usage with Satellite 6](https://access.redhat.com/articles/1586183)
[Getting started with Red Hat Satellite remote execution](https://www.redhat.com/en/blog/getting-started-red-hat-satellite-remote-execution)

