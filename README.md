#What is this?

This is instructions on how to create a VirtualBox Windows VM guest for use with Vagrant.

The VM created using these instructions is [available for download here](https://drive.google.com/file/d/0B4d7a4alPxxDZV9NRXQ1NGhIWk0/edit?usp=sharing).

#Create our Vagrant project
* Install vagrant
* Install the [vagrant-windows plugin](https://github.com/WinRb/vagrant-windows):

        vagrant plugin install vagrant-windows

* Create a new vagrant project folder:

        mkdir vagrant-windows
        vagrant init

* As per [the instructions](https://github.com/WinRb/vagrant-windows), edit the VagrantFile:

        Vagrant.configure("2") do |config|

          # Max time to wait for the guest to shutdown
          config.windows.halt_timeout = 25

          # Admin user name and password
          config.winrm.username = "vagrant"
          config.winrm.password = "vagrant"

          # Configure base box parameters
          config.vm.box = "vagrant-windows8.1"
          config.vm.box_url = "./vagrant-windows8.1.box"
          config.vm.guest = :windows

          # Port forward WinRM and RDP
          config.vm.network :forwarded_port, guest: 3389, host: 3389
          config.vm.network :forwarded_port, guest: 5985, host: 5985

        end

#Creating a Windows VM

* Download the appropriate VM files from the [Windows site](http://www.modern.ie/en-us/virtualization-tools)

* The VM files are typically in a number of parts (a .sfx file and a number of .rar files). As [per instructions](https://modernievirt.blob.core.windows.net/vhd/virtualmachine_instructions_2013-07-22.pdf): 

          chmod a+x IE11.Win8.1Preview.For.MacVirtualBox.part1.sfx
          ./IE11.Win8.1Preview.For.MacVirtualBox.part1.sfx.sfx

* Import the VM into VirtualBox and ensure that the network adaptors are set correctly

  * adaptor 1 should be set to NAT (to allow the guest to connect to the host), with an entry to port forward TPC port 2200 for 127.0.0.1 to port 22 on  the guest IP (obtained by doing an {{{ipconfig}}} on the guest machine once it is started. Therefore, it should look something like:

              ssh | TCP | 127.0.0.1 | 2200 | 10.0.2.15 | 22

  * adaptor 2 should be set to Host-only Adaptor (to allow the Host to connect to the guest)

* Start the VM:

  * user: IEUSER

  * password: passw0rd!

* Install [[freesshd|http://www.freesshd.com/]] onto the guest, and:

  * create an administrator user with a password

  * create a vagrant user, with the password vagrant

* Ensure that the freesshd server is running, and from the host check ssh:

          ssh -p 2200 administrator@127.0.0.1

* In freesshd on the guest, set a path to a known directory (e.g. C\Users\IEUser\Downloads), and on the host check sftp:

          touch stuff.txt
          sftp -P 2200 administrator@127.0.0.1
          put stuff.txt

* Download the [vagrant public key](https://github.com/mitchellh/vagrant/tree/master/keys), and place it in the directory listed in freesshd's Authentication tab (typically C:\Program Files\freeSSHd), renaming it to the name of the user (i.e. vagrant)

* Start powershell and type the following to elevate it to administrator:

          Start-Process powershell -Verb runAs

* as per [this post](http://social.technet.microsoft.com/Forums/scriptcenter/en-US/7ebe6048-688d-4c8c-92a9-402cd5e235d1/novice-config-troubleshooting-question?forum=ITCG) and [this post](http://www.minasi.com/newsletters/nws1304.html) ensure that the guest networks are configured to be private by entering the following in powershell: 

          Set-NetConnectionProfile -name "Network" -NetworkCategory private
          Set-NetConnectionProfile -name "Unidentified network" -NetworkCategory private

* In powershell enter the following:

          Enable-PSRemoting

* Change the the Remote UAC LocalAccountTokenFilterPolicy registry setting as per [this post](http://support.microsoft.com/kb/942817) and [this post](http://social.msdn.microsoft.com/Forums/en-US/ad02461a-878c-49a9-bc08-a0199d69b85c/winrm-error-access-denied?forum=wcf):

  * Using a local account within Local Administrators group, Right-click CMD.EXE, 'Run as Administrator'

  * Within Elevated CMD prompt ran the following with no errors from local account:

            reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
            winrm quickconfig
            winrm get winrm/config/client/auth

  * It is important to note these steps disable UAC and enable the local 'Administrator' account.  For security reasons these steps may be reversed.

          reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 0 /f

* Bring up the run box (Windows+R, or right-command + R) and enter the following:

          cmd runas /user:IEUser

* and then proceeding to change the winrm settings:

          winrm quickconfig -q
          winrm set winrm/config/winrs @{MaxMemoryPerShellMB="512"}
          winrm set winrm/config @{MaxTimeoutms="1800000"}
          winrm set winrm/config/service @{AllowUnencrypted="true"}
          winrm set winrm/config/service/auth @{Basic="true"}
          sc config WinRM start= auto

#Package the Windows VM

* Power down the VM and give it a more sensible name:

          mv "/Users/jasondarwin/VirtualBox VMs/Windows 8.1 Preview" /Users/jasondarwin/VirtualBox VMs/windows_8.1
          cd /Users/jasondarwin/VirtualBox VMs/windows_8.1/
          mv "Windows 8.1 Preview.vbox" windows_8.1.vbox
          mv "Windows 8.1 Preview.vbox-prev" windows_8.1.vbox-prev

* edit the <code>windows_8.1.vbox</code> file and ensure that the settings are consistent

  * Machine: name="vagrant_windows_8.1"

  * HardDisk: location="windows_8.1_disk1.vmdk"

* In our vagrant project folder, create the package:

          vagrant package --base vagrant_windows_8.1 --output vagrant-windows8.1.box

* bring up the vm:

          vagrant up
