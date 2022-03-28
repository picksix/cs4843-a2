# CS 4843 - Assignment 2

## Deployment

To avoid charges, I have taken down the environment. To deploy this stack, you will need an AMI with a snapshot of your website files in `/var/www/html`. Start first by deploying `network.yml` with an environment name. After that, you can deploy the webservers with `servers.yml`. It is important to note that you must use the same environment name that you specified for the network stack. Finally, you can deploy the `database.yml` stack, using the same environment name.

## network.yml

This file creates the network for the deployment. The whole network is in a single VPC, with an internet gateway attached to allow traffic ingress. A private subnet and a public subnet are deployed on two availability zones, creating 4 subnets in total. In both of the public subnets, there is a NAT gateway allowing traffic to egress the network from the private subnets. Without this NAT gateway, the private subnets would not be able to access the internet. It is worth noting, however, that the NAT gateway does not allow traffic from the internet to reach the private subnets. Both of the NAT gateways are assigned an elastic IP.

## servers.yml

This file deploys an auto-scaling load-balanced cluster on both of the private subnets defined previously. A load balancer is deployed on the VPC, routing traffic through the public subnets to the private subnets An auto-scaling cluster automatically populates servers into the private subnets. The load balancer is actually just a clever DNS server that balances requests between the listeners in the public subnet in each availability zone, forwarding traffic to the private subnet. One of the required parameters for this file is the AMI to deploy to the servers. By having the user specify a custom AMI, the website files can be packaged into an AMI, which is what I did for my test deployment.

## database.yml

This file deploys the database servers. Using the previously exported private subnets, servers are instantiated in both regions, and they install mariadb (a mysql fork) and start the service. In this stack, you are required to specify the volume ids for each server - this is to avoid them being destroyed when you take the stack down. It'd be bad news if your entire database got deleted! Because they are in the private subnet, they can be accessed by the webserver, but NOT by the outside world. This is important, as you would not want random users being able to access the database, even if it is protected with authentication.

## Jumpbox

To create a jumpbox, create an instance in the VPC that has the environment name you specified during deployment. For the security group, use the http+ssh security group. Deploy the instance in the public subnet in the same availability zone as the target machine. When the machine is deployed, SSH into it and use a tool like scp to transfer the key for the target machine. Then, you can SSH into the target machine in the private subnet by using its internal IP (found in the instance viewer). If you are unable to access the jumpbox, ensure that the jumpbox is on the correct VPC and on a public subnet. I made extensive use of a jumpbox in my test deployment, and it was a real life saver!
