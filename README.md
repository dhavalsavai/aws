# Site Deployment with Load Balancer, Custom Domain, and SSL Certificate on AWS

This guide provides step-by-step instructions for deploying a site on AWS using an EC2 instance with an Application Load Balancer, a custom domain, and SSL certificate configuration.

## Prerequisites

- AWS account with Administrator access.
- Custom domain registered and managed via a domain hosting service.
- SSL certificate (CRT and key files) available.

## Steps

### 1. Log in to AWS Console
- Log in to your AWS account with Administrator access.

### 2. Launch EC2 Instances
- Go to the **EC2** service.
- Click on **Launch Instances**.
- Enter the instance details:
  - **Server Name**: Enter a name for your server.
  - **AMI**: Select **Ubuntu**.
  - **Instance Type**: Choose **t3.micro**.
  - **Key Pair**: Create or select an SSH key for server access.
- **Network Settings**:
  - Enable **HTTP**, **HTTPS**, and **SSH** ports in the security group.
- Configure **Storage**:
  - Allocate the required storage.
- **Advanced Details**:
  - In the **User data** section, add the following script:

    ```bash
    #!/bin/bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install nginx -y
    echo "<html>
      <head>
        <title>Welcome to Test Site</title>
      </head>
      <body>
        <h1>Success! The test site is working!</h1>
      </body>
    </html>" | sudo tee /var/www/html/index.html
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```

- **Number of Instances**: Set the number of instances to 2.
- Click on **Launch Instance**.

### 3. Create Target Group
- Go to **EC2** service and scroll down to **Target Groups** under the Load Balancing options.
- Click on **Create Target Group**.
- Choose **Instances** as the target type.
- Enter a name for the target group.
- Select **HTTP** as the protocol and configure IP type, VPC, and health checks.
- Click **Next**.
- Select the instances you deployed earlier and include them in the target group.
- Click **Create Group**.

### 4. Create Application Load Balancer
- Go to **EC2** services and click on **Load Balancers**.
- Click on **Create** under **Application Load Balancer**.
- Enter the details:
  - **Load Balancer Name**: Enter a name.
  - **Scheme**: Set to **Internet-facing**.
  - **IP Type**: Set to **IPv4**.
  - **Network Mapping**: Use the same VPC as the EC2 instances.
  - **Availability Zones**: Select availability zones for high availability.
  - **Security Group**: Select the security group used during instance launch.
- **Listeners and Routing**:
  - Specify your target group in the default action.
- Click **Create Load Balancer**.

### 5. Configure Domain and SSL
- Copy the DNS name of the load balancer.
- Go to your domain hosting panel and create a CNAME record pointing to the load balancer DNS name.
- Go to **ACM (AWS Certificate Manager)** and import your SSL certificate (CRT and key files).
- Go back to **Load Balancers** and click on **Add Listener**.
  - Select **HTTPS** protocol and your target group.
  - Choose the imported SSL certificate.
  - Click **Add**.

### 6. Redirect HTTP to HTTPS
- Go to the load balancer and click on the created load balancer.
- Select the **HTTP** (port 80) listener and click on **Create Listener Rules**.
- Add a rule to redirect HTTP to HTTPS:
  - **Condition**: Host header matches your domain URL.
  - **Action**: Redirect to URL with HTTPS.
- Click **Create**.

### 7. Test High Availability
- Stop the Nginx server on one instance and verify that the site is still accessible, confirming that the load balancer is properly distributing traffic between the instances.

## Conclusion

Your site is now deployed on AWS with a load balancer, custom domain, and SSL certificate, ensuring high availability and secure access.
