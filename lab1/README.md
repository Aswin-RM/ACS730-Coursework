# Lab 1
Overview of the four scripts.

Script 1: create-security-group.sh
Fetches our public IP and creates a new security inbound rule allowing traffic on SSH from our IP address. It names the new security group
acs730-week1-sg

Script 2: create-instance.sh
Creates an instance with the name acs730-week1.

Script 3: delete-instance.sh
Searches for an instance with name acs730-week1. After finding the instance, the script terminates it. If no such instances are found, it displays 
"Nothing to delete".

Script 4: delete-security-group.sh
Deletes the security group with name acs730-week1-sg 


