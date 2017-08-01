# -*- mode: ruby -*-
# vi: set ft=ruby :


$script_javaBuild = <<SCRIPT
#Install Git client 
echo "Installing Git.."
sudo yum -y install git
echo "Done."

#Install OpenJDK 
echo "Installing OpenJDK8.."
sudo yum install -y java-1.8.0-openjdk.x86_64
echo "Done."


#Install Maven 
echo "Install Maven 3 to machine..."
sudo wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo
yum install -y apache-maven
#check Install
mvn -version
echo "Install done..."
SCRIPT

$script_csBuild = <<SCRIPT
#Install Chocolatey
Write-Host "Installing Chocolatey..."
iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))
Write-Host "Done."

#Install Git
Write-Host "Installing Git..."
choco install git.install -y
Write-Host "Done."

#Install MSBuild
Write-Host "Installing MSBuild..."
choco install -y microsoft-build-tools -allowEmptyChecksums

#Install .NET Framework
Install-WindowsFeature NET-Framework-45-Features

Write-Host "Done."
SCRIPT


$script_setupNetwork_linux = <<SCRIPT
echo "Setting up hostname..."
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
mv /etc/hosts /etc/hosts.old
touch /etc/hosts
echo "192.168.56.2 javaBuild" >> /etc/hosts
echo "192.168.56.3 csBuild" >> /etc/hosts
echo "127.0.0.1 localhost" >> /etc/hosts
#Disable IPV6
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
#Disable frewall
service iptables stop
chkconfig iptables off
echo "Networking/Hosts setup done..."
SCRIPT

$script_setupNetwork_windows = <<SCRIPT
#PS script to add hosts file entries
Write-Host "Adding host file entries..."
function add-hostfilecontent {            
 [CmdletBinding(SupportsShouldProcess=$true)]            
 param (            
  [parameter(Mandatory=$true)]            
  [string]$IPAddress,            
              
  [parameter(Mandatory=$true)]            
  [string]$computer            
 )            
 $file = Join-Path -Path $($env:windir) -ChildPath "system32\\drivers\\etc\\hosts"            
            
 $data = Get-Content -Path $file             
 $data += "$IPAddress  $computer"            
 Set-Content -Value $data -Path $file -Force -Encoding ASCII             
}

#Re-creating hosts file to avoid duplicate entries
del $file
file $file

#Actually adding the entries
add-hostfilecontent 192.168.56.2 javaBuild
add-hostfilecontent 192.168.56.3 csBuild
add-hostfilecontent 127.0.0.1 localhost

Write-Host "Finished adding host file entries..."

#Disable frewall
Get-NetFirewallProfile | Set-NetFirewallProfile -Enabled False
SCRIPT

$script_HelloJava = <<SCRIPT
#Pulls a Hello World from a Github repo and compiles the code
cd ~
git clone https://github.com/leereilly/hello-world-java.git
cd hello-world-java
javac HelloWorld.java
java HelloWorld
SCRIPT

$script_HelloCs = <<SCRIPT
#Pulls a Hello World from a Github repo and compiles the code
copy "C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\csc.exe" "C:\\Users\\vagrant\\Desktop\\"
cd "C:\\Users\\vagrant\\Desktop"
git clone https://github.com/salman-bhai/hello-world.git
cd "C:\\Users\\vagrant\\Desktop\\hello-world\\C#"
cmd /c "C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\csc.exe hello-world.cs"
cmd /c "hello-world"
SCRIPT


#Provisioning the VM's
Vagrant.configure("2") do |config|

    config.ssh.username = "vagrant"
    config.ssh.password = "vagrant"

      config.vm.provider "virtualbox" do |vb|
         vb.gui = true
      end

      config.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--usb", "on"]
        vb.customize ["modifyvm", :id, "--usbehci", "off"]
      end
    	  
  config.winrm.retry_limit = 30
  config.winrm.retry_delay = 10

	config.vm.define "javaBuild" do |javaBuild|
    javaBuild.vm.box = "nrel/CentOS-6.7-x86_64"
	  javaBuild.vm.hostname = "javaBuild"
    javaBuild.vm.provision :shell, :inline => $script_javaBuild
	  javaBuild.vm.provision :shell, :inline => $script_setupNetwork_linux
    javaBuild.vm.provision :shell, :inline => $script_HelloJava
		javaBuild.vm.synced_folder ".", "/vagrant", disabled: true
		javaBuild.vm.network "private_network", ip: "192.168.56.2", virtualbox__hostonly: true
    javaBuild.vm.network "forwarded_port", guest: 22, host: 2022, id: "ssh", auto_correct: true
    javaBuild.vm.provider "virtualbox" do |vm|
      vm.name = "javaBuild"
			vm.customize [
        'modifyvm', :id,
        '--memory', '768'
							
      ]
		end
	end

  config.vm.define "csBuild" do |csBuild|
    csBuild.vm.box = "mwrock/Windows2012R2"
    csBuild.vm.hostname = "csBuild"
    csBuild.vm.provision :shell, :inline => $script_csBuild 
    csBuild.vm.provision :shell, :inline => $script_setupNetwork_windows
    csBuild.vm.provision :shell, :inline => $script_HelloCs
    csBuild.vm.synced_folder ".", "C:\\vagrant", disabled: true
    csBuild.vm.network "private_network", ip: "192.168.56.3", virtualbox__hostonly: true
		csBuild.vm.network "forwarded_port", guest: 3839, host: 33839, id: "RDP", auto_correct: true
		csBuild.vm.network "forwarded_port", guest: 80, host: 880, id: "http", auto_correct: true
		csBuild.vm.network "forwarded_port", guest: 443, host: 4443, id: "https", auto_correct: true
	  csBuild.vm.communicator = "winrm"
		csBuild.vm.provider "virtualbox" do |vm|
		    vm.name = "csBuild"
		    vm.customize [
							'modifyvm', :id,
							'--memory', '2048'
							
						]
		end	
  end	

end 

