# MinishiftWindows10HyperV

Intructions how to configure instructions for Minishift on Windows 10 (using Hyper-V)

You must use have local Administrator privileges for some of the setup actions

## Required Software

- [GitBash](https://gitforwindows.org/)
- [Minishift](https://github.com/minishift/minishift/releases)
- [Openshift Client](https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-windows.zip)

## Configuring Windows

Enable Hyper-V (if not done yet)

Open PowerShell with **Admintrator** privileges and execute:
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName "Microsoft-Hyper-V" -All
```

Once the reboot was performed to active HyperV, add your local user to Hyper-V Admin group.

Open PowerShell with **Admintrator** privileges and execute:
```powershell
Add-LocalGroupMember -Group "Hyper-V Administrators" -Member "your_user_name"
```

Create a Virtual Switch to provide NAT for the minishift VM:
Open PowerShell with **Admintrator** privileges and execute:
```powershell
# Create an internal virtual switch
New-VMSwitch -SwitchName "MinishiftNAT" -SwitchType Internal

# Find the interface index of the virtual switch you just created.
$IfIndex = (Get-NetAdapter -name "vEthernet (MinishiftNAT)").IfIndex

# Configure the NAT gateway
New-NetIPAddress -IPAddress 192.168.50.1 -PrefixLength 24 -InterfaceIndex $IfIndex

# Configure the NAT network
New-NetNat -Name MinishiftNATnetwork -InternalIPInterfaceAddressPrefix 192.168.50.0/24
```

---
**NOTE**

At this point all the tasks requiring admin privileges are down, the next instructions should all be performed with your regular user.

---

## Configure GIT Bash

Open Git Bash and execute:

```bash
mkdir ~/bin
echo "export PATH=$PATH:$HOME/bin" >> ~/.bashrc
cd  ~/bin ; explorer .

# Using windows copy "minishift.exe"; "kubectl.exe"; "oc.exe" from the downloaded zip files into the 'bin' folder

# Adjust the PATH for the running shell
source ~/.bashrc
```

## Starting Minishift

Open Git Bash and execute:

```bash

# The start will take several minutes depending on your network bandwith and system HW

# Start minishift
minishift.exe start \
  --cpus 4 \
  --memory 8GB \
  --hyperv-virtual-switch  MinishiftNAT \
  --network-ipaddress 192.168.50.10 \
  --network-gateway 192.168.50.1 \
  --network-nameserver 8.8.8.8
```

## Using Minishift

Open Git Bash:
```sh
# Test it
minishift console 

# Login to the consile with: developer/developer
# Get the "oc login" command from the Copy Login Command action

# Setup env for docker
eval $(minishift --shell bash docker-env)

# Login to the Minishift internal docker registry
docker login -u developer -p $(oc whoami -t) $(minishift openshift registry)

# Get the docker hub bash image
docker pull bash
# Tag it for the minishift repo location
docker tag bash $(minishift openshift registry)/myproject/my-bash

# Push it to the minishift repo
docker push $(minishift openshift registry)/myproject/my-bash
```


