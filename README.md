# Ansible role to setup a Hybrid Docker Swarm cluster

Based on the blogpost by Larry Smith Jr. https://everythingshouldbevirtual.com/automation/containers/ansible-provision-docker-swarm-mode-1-12-cluster/

## Prequisites

* 2 Linux VM's (Ubuntu): 1 Master and 1 Node
* (1 Windows VM with Containers addon installed)
* Ansible installed with following galaxy playbooks
    * `ansible-galaxy install angstwad.docker_ubuntu`

## Windows node preparation

Run this on every Windows node before running the Ansible playbooks. Make sure you replace `$username` and `$password` at the top of the file. These requirements are taken from [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/windows_setup.html#host-requirements).

```
$username = "windows_username"
$password = "windows_password"

$url = "https://raw.githubusercontent.com/jborean93/ansible-windows/master/scripts/Upgrade-PowerShell.ps1"
$file = "$env:temp\Upgrade-PowerShell.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force

&$file -Version 5.1 -Username $username -Password $password -Verbose

$url = "https://raw.githubusercontent.com/jborean93/ansible-windows/master/scripts/Install-WMF3Hotfix.ps1"
$file = "$env:temp\Install-WMF3Hotfix.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
powershell.exe -ExecutionPolicy ByPass -File $file -Verbose

$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)

powershell.exe -ExecutionPolicy ByPass -File $file

Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true

Set-ExecutionPolicy -ExecutionPolicy Restricted -Force
```

## Configuration

Copy `hosts.example` to `hosts` and configure the file to your cluster requirements. You need at least 1 master node, 1 linux node. For Windows, please follow the instructions above to prepare the nodes. Don't forget to fill in the username and password for your Windows nodes. Linux nodes will connect through SSH with local private key.

## Running

Run the following command on your machine. `OBJC_DISABLE_INITIALIZE_FORK_SAFETY` is only needed if you run the [command on MacOS](https://github.com/ansible/ansible/issues/32499).

```
ANSIBLE_HOST_KEY_CHECKING=False OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES ansible-playbook -i hosts ansible/docker-swarm.yml
```
