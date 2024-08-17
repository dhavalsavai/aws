# VPC Peering Connection Between Two EC2 Instances on AWS

This guide provides a step-by-step process to set up a VPC peering connection between two EC2 instances on AWS. This setup includes creating two separate VPCs, setting up subnets, route tables, internet gateways, and finally establishing a peering connection between the VPCs.

## Prerequisites

- AWS account with Administrator access.
- AWS CLI configured with necessary permissions.

## Steps

### 1. Create VPCs

- Log in to your AWS account.
- Navigate to **VPC** under **Services**.
- Create the first VPC:
  - **Name**: `demo1`
  - **IPv4 CIDR block**: `12.0.0.0/16`
  - Click **Create VPC**.
- Create the second VPC:
  - **Name**: `demo2`
  - **IPv4 CIDR block**: `13.0.0.0/16`
  - Click **Create VPC**.

### 2. Create Route Tables

- In the VPC dashboard, click on **Route Tables**.
- Create a route table for the first VPC:
  - **Name**: `demo1-route`
  - **VPC**: `demo1`
  - Click **Create Route Table**.
- Create a route table for the second VPC:
  - **Name**: `demo2-route`
  - **VPC**: `demo2`
  - Click **Create Route Table**.

### 3. Create Subnets

- In the VPC dashboard, click on **Subnets**.
- Create a subnet for the first VPC:
  - **VPC**: `demo1`
  - **Subnet name**: `demo1-subnet`
  - **IPv4 CIDR block**: `12.0.1.0/24`
  - Click **Create Subnet**.
- Create a subnet for the second VPC:
  - **VPC**: `demo2`
  - **Subnet name**: `demo2-subnet`
  - **IPv4 CIDR block**: `13.0.1.0/24`
  - Click **Create Subnet**.

### 4. Associate Subnets with Route Tables

- For `demo1-route`:
  - Go to **Route Tables** > **demo1-route** > **Subnet Associations**.
  - Click **Edit** and select `demo1-subnet`.
  - Click **Save Associations**.
- For `demo2-route`:
  - Go to **Route Tables** > **demo2-route** > **Subnet Associations**.
  - Click **Edit** and select `demo2-subnet`.
  - Click **Save Associations**.

### 5. Create Internet Gateways

- Create an Internet Gateway for `demo1`:
  - Go to **Internet Gateways** and click **Create Internet Gateway**.
  - **Name**: `demo1-igw`
  - Click **Create Internet Gateway**.
  - Attach it to `demo1` VPC.
- Create an Internet Gateway for `demo2`:
  - Go to **Internet Gateways** and click **Create Internet Gateway**.
  - **Name**: `demo2-igw`
  - Click **Create Internet Gateway**.
  - Attach it to `demo2` VPC.

### 6. Update Route Tables with Internet Gateway

- For `demo1-route`:
  - Go to **Route Tables** > **demo1-route** > **Routes**.
  - Click **Edit** and add a route:
    - **Destination**: `0.0.0.0/0`
    - **Target**: `demo1-igw`
  - Click **Save Changes**.
- For `demo2-route`:
  - Go to **Route Tables** > **demo2-route** > **Routes**.
  - Click **Edit** and add a route:
    - **Destination**: `0.0.0.0/0`
    - **Target**: `demo2-igw`
  - Click **Save Changes**.

### 7. Launch EC2 Instances

- Launch an EC2 instance in `demo1`:
  - **AMI**: Ubuntu
  - **Instance Type**: t2.micro
  - **Network**: Select `demo1`
  - **Subnet**: Select `demo1-subnet`
  - **Security Group**: Allow HTTP, HTTPS, and SSH
  - **User Data**:
    ```bash
    #!/bin/bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install nginx -y
    echo "<html><h1>Success! The test site is working on demo1!</h1></html>" | sudo tee /var/www/html/index.html
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```
- Launch an EC2 instance in `demo2`:
  - **AMI**: Ubuntu
  - **Instance Type**: t2.micro
  - **Network**: Select `demo2`
  - **Subnet**: Select `demo2-subnet`
  - **Security Group**: Allow HTTP, HTTPS, and SSH
  - **User Data**:
    ```bash
    #!/bin/bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install nginx -y
    echo "<html><h1>Success! The test site is working on demo2!</h1></html>" | sudo tee /var/www/html/index.html
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```

### 8. Test EC2 Instances

- Use the public IP addresses to access the test sites via a browser and ensure that they are working.

### 9. Create VPC Peering Connection

- Go to **VPC** > **Peering Connections**.
- Click **Create Peering Connection**:
  - **Name**: `peering-connection-demo1-to-demo2`
  - **Requester VPC**: `demo1`
  - **Accepter VPC**: `demo2`
  - Click **Create Peering Connection**.
- Accept the peering request for `demo2`.

### 10. Update Route Tables for VPC Peering

- For `demo1-route`:
  - Go to **Route Tables** > **demo1-route** > **Routes**.
  - Click **Edit** and add a route:
    - **Destination**: `13.0.0.0/16`
    - **Target**: `peering-connection-demo1-to-demo2`
  - Click **Save Changes**.
- For `demo2-route`:
  - Go to **Route Tables** > **demo2-route** > **Routes**.
  - Click **Edit** and add a route:
    - **Destination**: `12.0.0.0/16`
    - **Target**: `peering-connection-demo1-to-demo2`
  - Click **Save Changes**.

### 11. Test VPC Peering

- SSH into both EC2 instances using their public IP addresses.
- Use the private IP addresses to ping each instance from the other.
- Run `curl` to test connectivity and ensure that VPC peering is functioning correctly.

## Conclusion

You have successfully set up a VPC peering connection between two EC2 instances on AWS, enabling them to communicate securely over private IP addresses across different VPCs.
