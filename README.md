# Launch a Windows EC2 Instance and Connect from a Local Linux Machine

## Project Overview
This project demonstrates how to launch a Windows EC2 instance on AWS and connect to it from a local Ubuntu Linux machine using Remote Desktop Protocol (RDP).

## Technologies Used
- Amazon Web Services (AWS)
- Amazon EC2
- Windows Server
- Ubuntu Linux
- Remmina Remote Desktop Client
- Remote Desktop Protocol (RDP)

## Implementation Steps
1. Set up Ubuntu Linux locally.
2. Launch a Windows EC2 instance on AWS.
3. Configure the Security Group to allow RDP traffic on TCP port 3389 from your IP.
4. Retrieve the Windows Administrator password using the EC2 key pair.
5. Open Remmina in Ubuntu.
6. Connect to the Windows EC2 instance using its Public IPv4 address and Administrator credentials.

## Project Demonstration
YouTube Video: https://www.youtube.com/watch?v=wtNYvoChs90

## Learning Outcome
Learned how to deploy a Windows virtual machine on AWS and remotely access it from a local Linux environment using RDP.
