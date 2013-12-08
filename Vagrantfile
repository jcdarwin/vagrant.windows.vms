# -*- mode: ruby -*-
# vi: set ft=ruby :

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