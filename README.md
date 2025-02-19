# Speaches-Storyteller-connection-docs
How to connect Speaches and Storyteller to run Whisper transcription on your PC while keeping the services on your server.


# Guide to Setting Up Storyteller Using Speeches

## Overview

Speeches (formerly faster-whisper-server) is a tool that leverages Whisper technology for audio transcription. When combined with Storyteller, it enables the creation of synchronized guided narrated ebooks. This guide focuses on setting up Speeches on a Windows PC with WSL2 to work with a network-connected Storyteller installation.

## Why Use a Dedicated Setup?

While running Storyteller on a dedicated server (like a mini PC) is useful for:
- Syncing progress
- Downloading books when your main PC is asleep
- Resource management (when WSL needs to be disabled)

You can leverage your main PC's powerful processor (CPU/GPU) for transcription by using Speeches over your network. This setup allows Speeches to utilize Storyteller's OpenAI Cloud Platform settings for local Whisper processing.

## System Requirements

### Hardware Requirements
- **RAM:** Minimum 16GB recommended
  - 10GB dedicated to WSL2 (based on tested configuration)
- **Processor:** These requirements are for running WSL2 and Docker, Speaches has additional requirements
  - 64-bit processor with Second Level Address Translation (SLAT)
  - CPU virtualization extensions must be enabled in BIOS/UEFI:
    - Intel processors: Intel VT-x
    - AMD processors: AMD-V
  - Minimum 2 CPU cores recommended
  - 4+ cores recommended for optimal performance
  - For GPU acceleration support:
    - NVIDIA: Compute Capability 3.5 or higher
    - AMD: Speaches does not appear to support AMD GPUs
- **Storage:** Sufficient space for WSL2 and Docker images
  - 100GB+ recommended for WSL2 and Docker images, especially if using GPU acceleration

### Software Requirements
- Windows 10 (version 1903 or higher, Build 18362 or higher) or Windows 11

## Installation Steps

#### Installing WSL2

1. Open PowerShell as Administrator and run:
```powershell
wsl --install
```

2. Restart your computer when prompted

##### Verify Installation
After using either method, verify the installation:
```powershell
wsl --status
wsl --version
```

If needed, update WSL:
```powershell
wsl --update
```

#### Configuring Resource Allocation
1. Create or edit `.wslconfig` file in your Windows user directory:
```powershell
notepad "$env:USERPROFILE/.wslconfig"
```

2. Some recommended confiuration for WSL. Add the following configuration:
```ini
[wsl2]
memory=10GB
localhostForwarding=true
nestedVirtualization=true
```

3. Apply WSL configuration:
```powershell
wsl --shutdown
wsl --start
```

### Ubuntu Installation (Required)
1. Install Ubuntu distribution:
```powershell
wsl --install -d Ubuntu
```

2. Launch Ubuntu to complete setup:
```powershell
wsl -d Ubuntu
```

3. Create your Linux username and password when prompted. This can be different from your Windows username and password but it will be used later so take note of it.

### NVIDIA CUDA Support (Optional)
If using GPU acceleration with an NVIDIA card:

1. Inside WSL Ubuntu terminal:
```bash
# Add NVIDIA package repositories
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb

# Update package list
sudo apt-get update

# Install CUDA and development tools
sudo apt-get install -y cuda-toolkit-12-3
```

2. Verify CUDA installation:
```bash
nvidia-smi
nvcc --version
```

#### Verification Steps
1. Check WSL version and status:
```powershell
wsl -l -v
```

2. Verify memory allocation:
```bash
free -h
```

3. Test CPU cores:
```bash
nproc
```

#### Troubleshooting WSL2 Setup
- If virtualization is disabled, enable it in BIOS/UEFI
- For "WSL 2 requires an update to its kernel component" error:
  ```powershell
  wsl --update
  ```
- For networking issues:
  ```powershell
  netsh winsock reset
  ```

### 2. Docker Configuration
1. Install Docker in WSL Ubuntu:
```bash
# Update package list
sudo apt-get update

# Install required packages
sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  software-properties-common

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package list again
sudo apt-get update

# Install Docker
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

2. Set up Docker permissions:
```bash
# Add your user to the docker group
sudo usermod -aG docker $USER

# Apply changes (you'll need to log out and back in)
newgrp docker
```

3. Verify Docker installation:
```bash
# Test Docker
docker --version
docker run hello-world
```

4. Install Docker Compose:
```bash
# Install Docker Compose
sudo apt-get install -y docker-compose-plugin

# Verify installation
docker compose version
```

### Network Setup
> 🚧 **Under construction*: I am not an expert on Docker networking, and this network is not used by the compose files created below. Assistance is requested.

1. Create a Docker network:
```bash
# Create a network for Speeches
docker network create speeches-network
```

2. Check network configuration:
```bash
# List Docker networks
docker network ls
```

3. Get WSL IP address (you'll need this later):
```bash
# Show IP address
ip addr show eth0
```

Note: Remember the IP address starting with 172.* - you'll need it when configuring the Windows firewall to work with Docker

### 3. Speeches Deployment
- Create a folder within the WSL enviornment and enter it. For example:
 ```bash 
 mkdir ~/speaches
 cd ~/speaches
 ```
 
You will need a docker compose file configured to run speaches
Instructions from [Speaches documentation](https://speaches.ai/installation/)
 
 ### For NVIDIA GPU:

 ```bash
 curl --silent --remote-name https://raw.githubusercontent.com/speaches-ai/speaches/master/compose.yaml
curl --silent --remote-name https://raw.githubusercontent.com/speaches-ai/speaches/master/compose.cuda.yaml
export COMPOSE_FILE=compose.cuda.yaml
```

### For CPU:
```bash
curl --silent --remote-name https://raw.githubusercontent.com/speaches-ai/speaches/master/compose.yaml
curl --silent --remote-name https://raw.githubusercontent.com/speaches-ai/speaches/master/compose.cpu.yaml
export COMPOSE_FILE=compose.cpu.yaml
```

You may wish to change the Port that is outputted incase of any conflicts.
- Open `compose.yaml`
> **Note:** You may use a Linux tool like `vi` or `nano` but for simplicity you can open files in the subsystem in Windows Notepad by running `notepad.exe compose.yaml`
- Edit the line 
```compose.yaml
    ports:
      - **8000**:8000
```
Replace the number on the **left** with a different number, such as `8007`
 - Container management
 To download and start the container, run 
 ```bash
 docker compose pull
 docker compose up --detach
```
> **Note: You may be required to run Docker commands as `sudo`. If you get an error or run into problems, 
 > ```bash
 > sudo docker compose pull
 > sudo docker compose up --detach
> ```

When prompted for your password, enter the one you created for *Linux*.


### 4. Network Configuration
1. WSL2 to Windows port forwarding
- Find your WSL2 IP address
```bash
# Show IP address
ip addr show eth0
```

- Within an elevated command prompt (cmd.exe as administrator)
> **Note:** Replace `listenport=**8007**` and `connectport=**8007**` with the port you have in `compose.yaml` which you may have changed
> Replace `connectaddress=**172.25.48.1**` with the port used for WSL2 (see step 1)

```cmd
netsh interface portproxy add v4tov4 listenport=8007 listenaddress=0.0.0.0 connectport=8007 connectaddress=172.25.48.1
```
2. Windows Firewall rules setup

To allow network access to Speeches through Windows Firewall:

a. Open Windows Defender Firewall with Advanced Security
```powershell
Start-Process "wf.msc"
```
Or find the Windows firewall within `Run` with "wf.msc"

b. Create new inbound rules:
- Click "New Rule" in the right panel
- Select "Port"
- Choose "TCP"
- Enter the specific port (e.g., 8007)
- Allow the connection
- Apply to all applicable network profiles (i.e. just private if you are using a laptop and don't want to expose a port while outside of your house)
- Name it "Speeches Server"

c. Command line alternative:
```powershell
# Replace 8007 with your configured port
New-NetFirewallRule -DisplayName "Speeches Server" -Direction Inbound -Protocol TCP -LocalPort 8007 -Action Allow -Profile Private
```

d. Verify the rule:
```powershell
Get-NetFirewallRule -DisplayName "Speeches Server"
```

3. Docker network integration with local network
> 🚧 **TO DO** I am unsure about how I got this running and could use help from others 

4. Verify that Speaches can be seen from other devices in your network.
a. Find your Windows PC's IP address:
   - Open PowerShell and run:
   ```powershell
   ipconfig
   ```
   - Look for the "IPv4 Address" under your main network adapter:
     - If using Wi-Fi, look under "Wireless LAN adapter Wi-Fi"
     - If using ethernet, look under "Ethernet adapter"
   - The address will be in format: `192.168.x.x` or `10.0.x.x`

b. From another device on your network, try accessing:
   ```
   http://[YOUR_IP_ADDRESS]:[PORT]
   ```
   Example: `http://192.168.1.100:8007`
   If a `Speaches Playground` website comes up, you should be good. You can test the engine with the UI if you'd like.


### 5. Storyteller Integration
1. On your Storyteller server web app, go to Settings > Transcription Settings and set 
 **Transcription engine:** `OpenAI Cloud Platform`
 **API Key** This is marked as required in Storyteller but since you are not officially using OpenAI, it can be anything
 **Base URL:** http://[YOUR_IP_ADDRESS]:[PORT]/**v1**
    Example: `http://192.168.1.100:8007/v1`
> The v1 is required and is how the Speaches API is accessed

 **Model Name** `Systran/faster-distil-whisper-large-v3` provides the best results. You may consider changing based on the options in `Speaches Playground` (see above) if there are issues.

2. System testing and verification
Try transcribing a book. Note: the compression and chapter matching processes will still run on Storyteller, but they do not use as much memory or take as much time as the transcription. Check Task Manager on your PC to get an indication as to whether or not it is working. 
## Troubleshooting
You can check the Docker logs for both Storyteller and Speaches to get an indication of what is going on.

On the cooresponding devices:
```bash
docker ps
# or
sudo docker ps
```
Find the `CONTAINER ID` that cooresponds to the correct image and name. Then run 
```bash
docker logs [CONTAINERID]
# or
sudo docker logs [CONTAINERID]
```
### Support Resources
[Speaches documentation](https://speaches.ai/installation/#__tabbed_2_3)

## Contributing

Feel free to submit issues and enhancement requests.

## Acknowledgments

- Speeches project
- Storyteller project & Shane Friedman in particular for creating Storyteller and enlightening me on this way of using the two projects together.
