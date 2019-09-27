1. Create an empty git repository for all the config backup files - call it Splunk_Config
2. Have an extra server or use one of the servers you are backing up which has considerable memory to use as a base. 
3. This will be the server- call is server X, where you will be copying config files from all the servers to.
4. In server X (not as root), Install git using - 
```
sudo yum install git
```
5. Create a directory called git under /opt using -
```
 mkdir -p /opt/git
```
6. Clone the git repository onto the storage server using your application key using - (if using azure devops or others)    -
``` 
git clone https://<username>:<application-key>@<domain_name>/group_name/_git/Splunk_Config
```
7. Make sure the SSH connection is open from all the servers to server X
8. To do this, generate a private key (only used for backups) under your home directory in server X. Copy this private key and paste it in a file, (let's call the file cloud_key) under .ssh folder of all your servers under your home directory in the servers. 
9. Each Splunk server pushes a copy of the /opt/splunk/etc directory and sub-directories to the server X using SCP. This is automated using a cron job and SSH keys for authentication. 
10. An example of this command from server Y would be - 
```
scp -qr -i /home/<username>/.ssh/cloud_key /opt/splunk/etc/* <username>@<server X>:/opt/git/Splunk_Config/<server Y>/etc
```
11. Schedule this command to run as a cron job in all the servers, make sure they are at least 15 mins apart from each other - 
```
crontab -e    
00 00 * * * <scp command>
```
12. The server X uses git to commit the newly uploaded files and push them to the [DevOps] repo. This is automated in the splunkbackup.sh script. Create this script under /opt/git of your home directory of server X
```
cd /opt/git/Splunk_Config
git add .
git commit -m "$(date +%Y-%m-%d_%H:%M:%S)"
git push -u origin --all
rm -Rf /opt/git/Splunk_Config/*/etc/*
```
13. The code adds all the files under the /Splunk_Config folder, commits to the repository using the time so you can keep track of your backups and deletes the files to allow for new files to be written the next day. 
14. Automate this script such that it runs after all the copy crons runs in the other servers.
