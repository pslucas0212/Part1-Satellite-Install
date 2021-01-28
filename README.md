# Satellite Installation Instructions

1. Install RHEL 7.9
2. Update firewall rules
  firewall-cmd \
--add-port="80/tcp" --add-port="443/tcp" \
--add-port="5647/tcp" --add-port="8000/tcp" \
--add-port="8140/tcp" --add-port="9090/tcp" \
--add-port="53/udp" --add-port="53/tcp" \
--add-port="67/udp" --add-port="69/udp" \
--add-port="5000/tcp"

firewall-cmd --runtime-to-permanent

firewall-cmd --list-all

4. Check host name and local DNS resolution
ping -c3 localhost

ping -c3 `hostname -f`

5. Set static and transient hostname
hostnamectl set-hostname sat01.example.com

6. Register Satellite Server to RHSM
subscription-manager register --org=14029827 --activationkey=rhel_premium

This both registers the server and attaches a RHEL subscription to the Satellite Server

7. Attach Satellite Infrastructure Subscription




