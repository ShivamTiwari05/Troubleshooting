The DNS was not resolving to the ALIAS (newly created loadbalancer): application was not opening while hiiting the DNS.

troubleshooting steps:
1. checked if application is running on the ec2 or not?: application was not running on the EC2 with https://<IP-port>:80, because the target port mentiones was 80 but application
running on 8080; thus target port was update in CloudFormation. ------> Now application was accessible on https://<IP-port>:8080
2. now, application was also opening on LoadBalancer DNS
3. But, the loadbalancer DNS detail was not being updated in route53 DNSrecord type.

4. initailly the DNS record was updated manually and then the application was accessible easily on https://xyz.com, later AWS script was used to update the loadbalancer details on DNSRecord.

