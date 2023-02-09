# Automatic-start-docker-in-wsl

Scripts/steps for 
* docker service setup in wsl 
* automatic start docker and selected containers in **wsl** after **Windows** reboot. 

## (optional) Setup ssh on server, run in Windows PowerShell
```
net start sshd
#net stop sshd
```

## Mount new drive to wsl

```
# run in wsl to mount new drive in Win10 (in Win11 simply use wsl --mount in windows powershell)
sudo mkdir /mnt/f 
sudo mount -t drvfs f: /mnt/f # contd
```

In case you want to unmount:
```
sudo umount /mnt/foldername 
sudo rmdir /mnt/foldername
```

## Start docker service in wsl
```
sudo service --status-all # check service status
sudo service docker start # start service
```

## Automatic start docker service after reboot
Since wsl is not boot system, we can not use systemctl for automatic service start after "reboot" (wsl as a sub system never got rebooted anyway). 

For automatic start, we need to turn off the requirement for a password for our command.
* run "sudo visudo" in wsl
* add the following command to the bottom of the sudoers file, and then press Ctrl+o to save and Ctrl+x to exit the file.
```
%sudo ALL=NOPASSWD:/usr/sbin/service docker start
```

Then put the .bat file that contains command 
```
@echo off
wsl sudo service docker start
```
to folder "C:\Users\USER\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup" and/or "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp".


Alternatively, we can use Windows Task Scheduler
* open Windows Task Scheduler, create a new basic task.
* setup Triggers "At startup"
* setup Actions: In "Program/Scirpts" input "C:\Windows\System32\wsl.exe", in "arguments" input "sudo service docker start"
* setup Conditions, uncheck the "start only when on AC power" box. (Yeah weird enough but you have to uncheck it for the task to auto-run.)
* save the task

## Run docker container in wsl
For example run the auto-bilibili-recorder container from https://github.com/valkjsaaa/auto-bilibili-recorder.
```
docker run -d --restart=always --gpus all -e NVIDIA_DRIVER_CAPABILITIES=video,compute,utility --name recorder -v /mnt/f/cfm:/storage ghcr.io/valkjsaaa/auto-bilibili-recorder-gpu:master
```

## Set container to auto start 
```
docker update --restart unless-stopped recorder
```
This should complete the setup for "auto start docker and container after Windows reboot". Below are some useful commands.

## Useful commands
```
#check all containers info
docker ps -a

# container logs
docker logs <container>

# Exchange files between local drive and container dirve
docker cp /mnt/f/cfm/my_repo/session.py recorder:/webhook/session.py 
docker cp /mnt/f/cfm/my_repo/record_upload_manager.py recorder:/webhook/record_upload_manager.py 
docker cp  /mnt/f/cfm/my_repo/recorder_config.py recorder:/webhook/recorder_config.py
docker cp recorder:/webhook/recorder_config.py /mnt/f/test.py

# restart container 
docker restart recorder
```



