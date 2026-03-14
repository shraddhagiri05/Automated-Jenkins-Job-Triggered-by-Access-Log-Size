# Automated-Jenkins-Job-Triggered-by-Access-Log-Size


## Introduction
This project implements an automated log monitoring and archival system using Linux, Jenkins, and AWS. A cron-based shell script continuously monitors an Apache access log file and automatically triggers a Jenkins job when the log size exceeds 1GB. Jenkins then uploads the log to Amazon S3 and clears the local log file to free disk space.

## Objective
Monitor Apache access log size automatically

Trigger Jenkins job when log exceeds 1GB

Upload log file to Amazon S3

Verify successful upload

Clear the log file after upload

Achieve full automation with no manual intervention

## Tools Used
Linux (Ubuntu)

Shell Scripting (Bash)

Cron Scheduler

Jenkins

AWS CLI

Amazon S3

Apache Web Server

AWS IAM

## Steps to Execute the Project
## Step 1: Launch Ubuntu Server
Create an EC2/VM with Ubuntu

SSH into the server

## Step 2: Install Apache (Log Generator)
           sudo apt update

           sudo apt install apache2 -y

           sudo systemctl start apache2

           sudo systemctl enable apache2
Verify:

         ls /var/log/apache2/access.log
## Step 3: Install Jenkins
          sudo apt install openjdk-17-jdk -y

           sudo apt install jenkins -y

          sudo systemctl start jenkins

          sudo systemctl enable jenkins
Open Jenkins:

http://<SERVER_IP>:8080

Complete initial setup and create admin user.

## Step 4: Install AWS CLI
sudo apt install awscli -y
## Step 5: Create IAM User for Jenkins
Create IAM user: jenkins-user

Attach S3 permissions (PutObject, GetObject, ListBucket)

Generate Access Key & Secret Key

## Step 6: Configure AWS CLI for Jenkins User

       sudo su - jenkins

       aws configure
Verify:

aws sts get-caller-identity
Exit:

exit

## Step 7: Create S3 Bucket
      aws s3 mb access-log-backup-shraddha
## Step 8: Create Jenkins Job
Job Name: log-upload-job

Job Type: Freestyle Project

Enable This project is parameterized

String Parameter

Name: LOG_FILE_PATH

Value: /opt/log-archive/access.log

## Step 9: Jenkins Execute Shell Script
           LOG_FILE=$LOG_FILE_PATH

           BUCKET=access-log-backup-adarsh

           TIME=$(date +%F-%H-%M-%S)

          if [ ! -f "$LOG_FILE" ]; then
          exit 1
          fi

     aws s3 cp "$LOG_FILE" s3://  $BUCKET/access-$TIME.log

        if [ $? -eq 0 ]; then
          : > "$LOG_FILE"
         else
        exit 1
         fi
Save the job.

## Step 10: Create Directory for Jenkins Log Access
     sudo mkdir -p /opt/log-archive

    sudo chown jenkins:jenkins /opt/log-archive

## Step 11: Create Monitoring Script
nano /home/ubuntu/check_log.sh

Paste:

     #!/bin/bash

     SOURCE_LOG="/var/log/apache2/access.log"
     JENKINS_LOG="/opt/log-archive/access.log"
     LIMIT=$((1024*1024*1024))

    SIZE=$(stat -c%s "$SOURCE_LOG")

    if [ "$SIZE" -ge "$LIMIT" ]; then
     sudo cp "$SOURCE_LOG" "$JENKINS_LOG"
    sudo chown jenkins:jenkins "$JENKINS_LOG"

          curl -X POST http://localhost:8080/job/log-upload-job/buildWithParameters \
        --user jenkinsadmin:JENKINS_API_TOKEN \
       --data LOG_FILE_PATH="$JENKINS_LOG"
        fi
Give permission:

     chmod +x /home/ubuntu/check_log.sh
## Step 12: Schedule Cron Job
      crontab -e
Add:

       */5 * * * * /home/ubuntu/check_log.sh >> /var/log/log_monitor.log 2>&1
## Step 13: Testing

Increase log size:

       sudo dd if=/dev/zero of=/var/log/apache2/access.log bs=1M count=1100
✔ Jenkins triggers ✔ Log uploaded to S3 ✔ Log cleared

## Error Handling & Security
Jenkins runs as a non-root user

AWS credentials are configured per IAM user (no hardcoding)

Script validates log file existence before upload

Log file is cleared only after successful S3 upload

Controlled directory access avoids giving Jenkins root privileges

## Project Summary
This project demonstrates a real-world DevOps automation use case by integrating Linux cron jobs, Jenkins automation, and Amazon S3 storage. It ensures efficient log management, secure cloud storage, and optimized disk usage while following best practices such as least-privilege IAM access and automated monitoring.

![](./Img/Gdrive.png)



![](./Img/Gdrive%201%20(2).png)

