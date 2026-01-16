we will use the github repo that help us with the script on chaper 1 
https://github.com/community-scripts/ProxmoxVE/blob/main/ct/docker.sh

so i have run that script with the following command:
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/docker.sh)"
```bash

installation screenshot:
/Users/ofrika/Documents/GitHub/homelab-journey/assets/screenshots/chapter - 02/Screenshot 2026-01-16 at 15.57.19.png

i have asked if i want to install portainer this is new to me so i have did some research about it and found out that its a great tool to manage docker containers with a web ui so i have installed it.
but currently i dont have many containers to manage so i will explore it later if needed 
i have check that i can add it later if needed 
and if i need to mange it i can access it via doker command also

docker compose up -d
docker compose ps
docker compose logs -f
docker compose pull ואז up כדי לעדכן

then i asked if install the Portainer Agent (for remote management)? <y/N> i have said no we will connect to proxmox directly 

then i asked if  Expose Docker TCP socket (insecure) ? [n = No, l = Local only (127.0.0.1), a = All interfaces (0.0.0.0)] <n/l/a>: 
currently i dont find a usecase for it i will make avilble n8n out but i dont need my docker socket to be open so i have said n

we done 
![alt text](image.png)

lets test it 
i run 
pct enter 100
to enter the container 

then i run 
ip a | grep 10.100.102.15 -n || hostname -I
to check the ip adress of the container

i got 10.100.102.15

then i run
docker --version 
to check docker is installed
Docker version 29.1.4

mkdir -p /opt/n8n && cd /opt/n8n
open the file with nano and paste the following content 
here i had to do some resech this is a first forme docker compose file i have ever made 
so i have checked the release notes of n8n and postgrace to find the latest version i want it to be stable and not beta and of cuorse for securrity reasons after all i am security automation engineer 
learn about docker compose here https://www.youtube.com/watch?v=DM65_JyGxCo 
