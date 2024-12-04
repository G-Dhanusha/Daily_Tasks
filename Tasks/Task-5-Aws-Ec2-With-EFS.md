* Creating a real-time application using AWS EC2 with an Elastic File System (EFS) volume allows multiple instances to share a common file system,
making it ideal for scenarios like real-time data processing, content management, or collaborative applications. 
* Below, I’ll guide you through setting up a simple real-time chat application that utilizes EFS for storing chat messages.

#### Project Overview: Real-Time Chat Application Using AWS EC2 with EFS

#### Prerequisites
- An AWS account
- AWS CLI installed and configured
- Basic knowledge of Node.js and Express

####  Step-by-Step Guide

#### What is EFS(Elastic File System) ?

* Amazon Elastic File System (Amazon EFS) provides a simple, scalable, elastic file system for general purpose workloads for use with AWS Cloud services and on-premises resources.

#### Step 1: Create and Launch EC2 Instances

1. **Navigate to the EC2 Dashboard** in the AWS Management Console.

2. **Launch EC2 Instances:**

   - Click on `Launch Instance`
   - Choose an Amazon Machine Image (AMI), such as **Amazon Linux 2**.
   - Select an instance type (ex: `t2.micro` for the free tier).
   - Configure instance details, ensuring they are in the same VPC as your EFS.
   - **Add Storage:** Leave the default settings.
   - **Configure Security Group:**
     - Allow inbound traffic on port `22` (SSH) and port `3000` (for the chat application).
   - Review and launch the instance.

        ![preview](images/104.png)

        ![preview](images/105.png)

3. **Note the public DNS or IP address** of the instance for accessing the application later.

#### Step 2: Create an EFS File System

1. **Log in to the AWS Management Console** and navigate to the **EFS** service.

2. **Create a New File System:**

   - Click on `Create file system`
   - Choose the `default settings`, and make sure to create the `file system in the same VPC where your EC2 instances` will reside.
   - Click `Create`

        ![preview](images/102.png)
        ![preview](images/103.png)

   - Note the **`File System ID`** for later use and click `Attach`.

        ![preview](images/106.png)

        ![preview](images/107.png)

        ![preview](images/108.png)


3. **Configure Access Points (optional):** You can set access points for permissions and paths, but for simplicity, we’ll use the default settings.

#### Step 3: Mount EFS on the EC2 Instances

1. **SSH into Your EC2 Instance:**

   ```bash
   ssh -i your-key.pem ec2-user@<your-instance-public-dns>
   ```
2. **Install the EFS Utilities:**

   ```bash
   sudo yum install -y amazon-efs-utils
   ```

3. **Create a Mount Point:**

   ```bash
   sudo mkdir /mnt/efs
   ```

4. **Mount the EFS File System:**

   - Replace `fs-XXXXXX` with your EFS File System ID.

   ```bash
   sudo mount -t efs fs-XXXXXX:/ /mnt/efs

   or like below

   sudo mount -t efs -o tls fs-04b19a61c07622f0f:/ /mnt/efs
   ```
    ![preview](images/109.png)
    

5. **Ensure EFS is Mounted on Reboot:**

   - Edit the fstab file:

        ```bash
        echo "fs-XXXXXX:/ /mnt/efs efs defaults,_netdev 0 0" | sudo tee -a /etc/fstab

        ```
        ![preview](images/110.png)

#### Step 4: Set Up the Node.js Application

1. **Install Node.js and NPM:**

    ```bash
    sudo yum install -y nodejs npm
    ```

2. **Create a New Directory for Your Project:**

   ```bash

   mkdir /mnt/efs/chat-app
   cd /mnt/efs/chat-app

   ```

3. **Initialize a New Node.js Project:**

   ```bash
   npm init -y
   ```
   ![preview](images/111.png)

4. **Install Required Packages:**

   ```bash
   npm install express ws body-parser
   ```

5. **Create the Server:**

   - Create a file named `server.js`:

     ```javascript
     const express = require('express');
     const WebSocket = require('ws');
     const bodyParser = require('body-parser');
     const fs = require('fs');
     const path = require('path');

     const app = express();
     const server = require('http').createServer(app);
     const wss = new WebSocket.Server({ server });

     const DATA_FILE = '/mnt/efs/messages.txt'; // EFS file path

     app.use(bodyParser.json());
     app.use(express.static('public'));

     app.get('/messages', (req, res) => {
         fs.readFile(DATA_FILE, 'utf8', (err, data) => {
             if (err) return res.status(500).send('Error reading messages');
             res.send(data);
         });
     });

     wss.on('connection', (ws) => {
         console.log('New client connected');

         ws.on('message', (message) => {
             console.log(`Received: ${message}`);
             fs.appendFile(DATA_FILE, message + '\n', (err) => {
                 if (err) console.error('Error saving message:', err);
             });

             // Broadcast the message to all connected clients
             wss.clients.forEach((client) => {
                 if (client.readyState === WebSocket.OPEN) {
                     client.send(message);
                 }
             });
         });

         ws.on('close', () => {
             console.log('Client disconnected');
         });
     });

     const PORT = process.env.PORT || 3000;
     server.listen(PORT, () => {
         console.log(`Server is running on port ${PORT}`);
     });
     ```

6. **Create a Simple HTML Client:**
   - Create a directory named `public`, and inside it create an `index.html` file:
     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>Real-Time Chat</title>
         <style>
             body { font-family: Arial, sans-serif; }
             #messages { border: 1px solid #ccc; height: 300px; overflow-y: scroll; }
         </style>
     </head>
     <body>
         <h1>Chat Room</h1>
         <div id="messages"></div>
         <input type="text" id="message" placeholder="Type a message..." />
         <button id="send">Send</button>

         <script>
             const ws = new WebSocket('ws://' + window.location.host);
             const messagesDiv = document.getElementById('messages');
             const messageInput = document.getElementById('message');

             ws.onmessage = function(event) {
                 const message = document.createElement('div');
                 message.textContent = event.data;
                 messagesDiv.appendChild(message);
             };

             document.getElementById('send').onclick = function() {
                 const message = messageInput.value;
                 ws.send(message);
                 messageInput.value = '';
             };
         </script>
     </body>
     </html>
     ```

     ![preview](images/112.png)

#### Step 5: Run Your Application

1. **Start the Server:**
   ```bash
   node server.js
   ```
     ![preview](images/115.png)

2. **Access Your Application:**
   - Open a web browser and navigate to `http://your-instance-public-dns:3000`.

        ![preview](images/113.png)

   -  Write any `message in chatbox` and click `send`. Message will display like `Received` in terminal

    ![preview](images/114.png)

    ![preview](images/115.png)


#### Step 6: (Optional) Setup Auto Scaling

If you want to deploy this application across multiple EC2 instances, you can set up an **Auto Scaling Group** and a **Load Balancer** through the EC2 console or using CloudFormation.

#### Step 7: Cleanup

`Note`: When you are done with your project, remember to terminate your EC2 instances and delete the EFS to avoid unnecessary charges.

####  Conclusion

You have successfully created a real-time chat application using AWS EC2 with EFS for storage! 
This setup allows multiple EC2 instances to access shared data in real-time. 
