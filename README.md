# Mern Application Deployment Guide

- This guide offers a detailed walkthrough for deploying a full-stack web application on the AWS cloud, incorporating NGINX server configuration.
- The application features comprehensive user authentication and interaction capabilities, including user registration, login, and access to personalized functionalities.
- The frontend utilizes React for an intuitive user interface, while the backend, powered by Node.js and Express, handles data processing and communicates with an RDS MySQL database.
- NGINX is employed to enhance both performance and security and to re-route the request from <IP:PORT> to <IP>.
- It also features an Autoscaling group that allows machines to be scaled up or down based on CPU usage.
- ASG uses a custom AMI and launch template with custom script in order to facilitate application setup and running the application.
- Monitoring and observability is achieved through cloudwatch and SNS.
- Jenkins is used on <IP:8080> to perform complete CICD.

## Prerequisites

Before you begin, you should have the following:

- An AWS account with RDS MySQL, EC2(local machine can be used instead), and security groups set up.
- Basic knowledge of AWS services and the command line.

## Deployment Steps

1. **Create an RDS MySQL Database and an EC2 Instance**:

   - Create an RDS MySQL database in your AWS account.
   - Launch an EC2 instance according to your preferences.

2. **Install Required Software**:

   - Install Node.js, npm, and MySQL on your EC2 instance.

3. **Clone the Project Repository**:

   - Clone the project repository from GitHub using "Git Clone" command

4. **Navigate to the Project Directory**:

   - Change your current directory to the project directory:
     ```
     cd mern-app-Deployment
     ```
     
5. **Edit the Server Configuration**:

   - Navigate to the `/server` directory and edit the `server.js` file to configure the MySQL database connection with your RDS instance.
     ```
     const db = mysql.createConnection({
     host: '<RDS_Endpoint>',
     port: '3306',
     user: '<username>',
     password: '<password>',
     database: '<db_name>',
     });
     ```

6. **Connect to RDS MySQL**:

   - Connect to your RDS MySQL database using the MySQL command-line client:
     ```
     mysql -h <RDS_Endpoint> -P 3306 -u <user_name> -p 
     ```
     
6. **Create a Database**:

   - Create the database you want to use:
     ```
     Create <db_name>
     ```

7. **Select Database**:

   - Select the database you want to use:
     ```
     use <db_name>
     ```

8. **Create a Table**:

   - Create a table named "users" with two columns: "username" and "password".
     ```
     CREATE TABLE users (username VARCHAR(20), password VARCHAR(20));
     ```

10. **Start the Frontend**:
    - Navigate to the `/client` directory and run the following command to start the frontend application on port 3000:
      ```
      npm start &
      ```

11. **Start the Backend**:

    - Navigate to the `/server` directory and run the following command to start the backend server on port 3001:
      ```
      npm run dev &
      ```

12. **Check Both Ends**:

    - Open a web browser and access `<instance_ip>:3000` to verify that both the frontend and backend are working correctly.

13. **Install NGINX and Configure Firewall**:

    - Install NGINX on your EC2 instance and ensure that the necessary ports are allowed in the security group.

14. **Edit NGINX Configuration**:

    - Use a text editor to edit the NGINX configuration file:
      ```
      sudo vim /etc/nginx/sites-available/default
      ```

15. **NGINX Configuration**:

    - Edit the file with the following configuration, replacing `<instance_public_ip>` with your instance's public IP address:
      ```
      server_name localhost;

      location / {
                proxy_pass http://localhost:3000; # Redirects the traffic to port 3000 where Node.js app is running
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
       }
      ```
16. **Remove Default NGINX Configuration**:

    - Remove the default NGINX configuration file from the enabled sites:
      ```
      sudo rm /etc/nginx/sites-enabled/default
      ```

17. **Enable Custom Configuration**:

    - Create a symbolic link to enable your custom NGINX configuration:
      ```
      sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
      ```

18. **Test NGINX Configuration**:

    - Verify that your NGINX configuration is correct without syntax errors:
      ```
      sudo nginx -t
      ```

19. **Reload NGINX**:

    - Reload NGINX to apply the new configuration:
      ```
      sudo service nginx reload
      ```

20. **Access the Application**:

    - Access the application on your EC2 instance's public IP address without specifying a port.

## Conclusion

Following these steps will enable you to deploy a full-stack application with NGINX on AWS. Remember to replace placeholders like `<instance_public_ip>`, `<RDS_Endpoint>`, `<user_name>`, and `<db_name>` with your specific configuration details. You can now access your application via your EC2 instance's public IP address.


