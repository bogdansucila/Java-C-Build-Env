# Java-C-Build-Env
Mini build environment for Java and C#, created using Vagrant

To power up, install Vagrant and VirtualBox, and run 'vagrant up' from the directory containing the Vagrantfile.

The Java build server is a CentOS 6.7 box, prepped with Git, OpenJDK 8 and Maven 3 using Bash as a scripting language to accomplish this.

The C# build server is a Windows Server 2012R2 box, set up with .NET 4.5 and Git, and configured using Powershell and the Chocolatey package manager.
