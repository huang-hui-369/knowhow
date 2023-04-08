# How to change docker images and containers location

1. ## 　Configure data-root path

   ```sh
   Create the daemon.json (C:\ProgramData\docker\config)and set the data-root path as you wish
   
   Example:
   {
   "data-root": "D:\\ProgramData\\Docker"
   }
   
   
   2. Restart the Docker Service
   ```

   

2. ## Configure WSL 2 Data Location

   This step is specific for the Docker Desktop installation with WSL 2. The WSL 2 Docker Desktop Data Virtual Machine default location is **%USERPROFILE%\AppData\Local\Docker\wsl\data\ext4.vhdx**. You can follow the below-listed steps to relocate the docker data.

   **Step 1** - Shut down the Docker Desktop by right-clicking the Docker Tray Icon as shown in Fig 1.

   ![image-20220603121835696](D:\github\knowhow\windows\wsl2\dockerDesktop.assets\image-20220603121835696.png)

   **Step 2** - Relocate the docker-desktop-data virtual machine disk image using the below-mentioned commands

   ```bat
   wsl --shutdown
   
   wsl --export docker-desktop-data docker-desktop-data.tar
   
   wsl --unregister docker-desktop-data
   
   wsl --import docker-desktop-data e:\docker\wsl\data docker-desktop-data.tar --version 2
   ```

   **Step 3** - Delete **%USERPROFILE%\docker-desktop-data.tar**.

   

   **Step 4** - Start Docker Desktop. It should start using the new location of WSL data.

   

3. ## create Symlinks

   In this approach, we will simply stop the Docker Desktop, move the space-eating directories to another drive having sufficient space, and finally creating symlinks.

   

   **Step 1** - Stop Docker Desktop.

   

   **Step 2** - Relocate the existing directories **%USERPROFILE%\AppData\Local\Docker** and **C:\ProgramData\Docker** to new directories. For example - move **%USERPROFILE%\AppData\Local\Docker** to **E:\Docker\AppData** and **C:\ProgramData\Docker** to **E:\Docker\ProgramData**. Make sure that you completely remove the existing directories from the C drive before creating the symlinks as shown below.

   ```bat
   # Link AppData - Replace youruser with actual user
   mklink /j "C:\Users\youruser\AppData\Local\Docker" "d:\Docker\AppData"
   
   # Link ProgramData
   mklink /j "C:\ProgramData\Docker" "d:\Docker\ProgramData"
   ```

   **Step 3** - Now start the Docker Desktop. It should start the Docker Engine using the new directories.

   

   ## 确认docker root path

```
docker info

Client:
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc., v0.8.2)
  compose: Docker Compose (Docker Inc., v2.5.1)
  sbom: View the packaged-based Software Bill Of Materials (SBOM) for an image (Anchore Inc., 0.6.0)
  scan: Docker Scan (Docker Inc., v0.17.0)

Server:
 Containers: 3
  Running: 3
  Paused: 0
  Stopped: 0
 Images: 3
 Server Version: 20.10.14
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runtime.v1.linux runc io.containerd.runc.v2
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 3df54a852345ae127d1fa3092b95168e4a88e2f8
 runc version: v1.0.3-0-gf46b6ba
 init version: de40ad0
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 5.10.102.1-microsoft-standard-WSL2
 Operating System: Docker Desktop
 OSType: linux
 Architecture: x86_64
 CPUs: 4
 Total Memory: 12.35GiB
 Name: docker-desktop
 ID: XNJ5:BUJD:CJ65:JUR7:EJX7:K4MY:ZRCA:RCGJ:N7VS:YLC2:OFFG:FU7X
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 HTTP Proxy: http.docker.internal:3128
 HTTPS Proxy: http.docker.internal:3128
 No Proxy: hubproxy.docker.internal
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  hubproxy.docker.internal:5000
  127.0.0.0/8
 Live Restore Enabled: false

```

