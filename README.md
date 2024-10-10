# Project Management System

## Overview
A full-stack project management system that allows users to track products, manage stock, and generate reports. The project consists of a client built with [Next.js](https://nextjs.org/) and a server using [Node.js](https://nodejs.org/) and [Prisma](https://www.prisma.io/).


## Technologies Used
- Client: Next.js, React, TailwindCSS
- Server: Node.js, Prisma, PostgreSQL
- Database: PostgreSQL
- Production : AWS, EC2, S3, RDS, Amplify, Cognito, Lambda

## DataModelling
![Schema Relations](client/public/schemaRelations.png)

## AWS Architecture
![Diagram](client/public/AWS-architecture.jpg)

## Cognito Integration
![Diagram](client/public/cognitoDiagram.png)



## Installation Steps

1. **Clone the repository**:
    ```bash
    git clone [git-url]
    cd project-management
    ```

2. **Install dependencies for both client and server**:
    ```bash
    # Client setup
    cd client
    npm i
    cd ..

    # Server setup
    cd server
    npm i
    ```

3. **Set up the database**:
    ```bash
    npx prisma generate
    npx prisma migrate dev --name init
    npm run seed
    ```

4. **Configure environment variables**:
    - For server settings, configure the `.env` file:
        ```plaintext
        PORT=your-port
        DATABASE_URL=your-database-url
        ```
    - For client settings, configure the `.env.local` file:
        ```plaintext
        NEXT_PUBLIC_API_BASE_URL=your-api-base-url
        ```

5. **Run the project**:
    ```bash
    npm run dev
    ```

## EC2 Setup Instructions(Amazon Linux)s

1. **Connect to EC2 Instance**:
    - Use EC2 Instance Connect from your AWS Console to access your instance.

2. **Install Node Version Manager (nvm) and Node.js**:
    - Switch to superuser:
      ```bash
      sudo su -
      ```
    - Install nvm:
      ```bash
      curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
      ```
    - Activate nvm:
      ```bash
      . ~/.nvm/nvm.sh
      ```
    - Install the latest version of Node.js:
      ```bash
      nvm install node
      ```
    - Verify installation:
      ```bash
      node -v
      npm -v
      ```

3. **Install Git**:
    - Update system and install Git:
      ```bash
      sudo yum update -y
      sudo yum install git -y
      ```
    - Check Git version:
      ```bash
      git --version
      ```
    - Clone your repository:
      ```bash
      git clone [your-github-link]
      cd project-management
      npm i
      ```

4. **Create Env File and Start the Application**:
    - Set the server to use port 80:
      ```bash
      echo "PORT=80" > .env
      ```
    - Start the application:
      ```bash
      npm start
      ```

5. **Install pm2 (Production Process Manager for Node.js)**:
    - Install pm2 globally:
      ```bash
      npm i pm2 -g
      ```
    - Create a pm2 ecosystem configuration file (inside the server directory):
      ```javascript
      module.exports = {
        apps: [{
          name: 'project-management',
          script: 'npm',
          args: 'run dev',
          env: {
            NODE_ENV: 'development',
            ENV_VAR1: 'environment-variable',
          }
        }],
      };
      ```
    - Modify the ecosystem file if necessary:
      ```bash
      nano ecosystem.config.js
      ```
    - Set pm2 to restart on reboot:
      ```bash
      sudo env PATH=$PATH:$(which node) $(which pm2) startup systemd -u $USER --hp $(eval echo ~$USER)
      ```
    - Start the application with pm2:
      ```bash
      pm2 start ecosystem.config.js
      ```

### Useful pm2 Commands:
- Stop all processes:
  ```bash
  pm2 stop all
  ```

- Delete all processes:
  ```bash
  pm2 delete all
  ```

- status of processes:
  ```bash
  pm2 status
  ```

- Monitor processes:
  ```bash
  pm2 monit
  ```

## RDS PostgreSQL 

To connect to your RDS PostgreSQL database, you will need to set up a connection string in your environment variables. The connection string format is as follows:

```plaintext
DATABASE_URL="postgresql://<username>:<password>@<hostname>:<port>/<database_name>?schema=<schema>"
```
remember to set the password only in alphanumeric format otherwise exceptions may occur




## S3 Bucket Policy

To allow public read access to the objects in your S3 bucket, you can use the following bucket policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "[bucket-arn]/*"
        }
    ]
}
```

## Lambda function
 ```plaintext
  import https from "node:https";

export const handler = async (event) => {
  const postData = JSON.stringify({
    username:event.request.userAttributes['preferred_username'] || event.userName,
    cognitoId : event.userName,
    profilePictureUrl:"i1.jpg",
    teamId: 1
  });
  const options={
    hostname:"[api-gateway-hostname]",
    port:443,
    path:"/prod/create-user",
    method:"POST",
    headers:{
      "Content-Type":"application/json",
      "Content-Length":Buffer.byteLength(postData)
    }
  };
  const responseBody= await new Promise((resolve,reject)=>{
    const req= https.request(options,(res)=>{
      res.setEncoding("utf8");
      let body = "";
      res.on("data",(chunk)=> body+=chunk);
      res.on("end",()=>resolve(body));
    });
    req.on("error",reject);
    req.write(postData);
    req.end();
  });
  return event;
};

 ```