# -*- mode: ruby -*-
# vi: set ft=ruby :

#-------------------------------------------------------------------------------
# VM_ALL_x values apply to all vms

# name of the box from Vagrant Cloud
VM_ALL_BOX = "rocky/8"

# number of CPUs
VM_ALL_CPU = "1"

# MB of RAM
VM_ALL_MEM = "1024"

# network type of second adapter ("hostonly" or "natnetwork")
VM_ALL_NET = "natnetwork"

# network domain for the VM network
VM_ALL_DOMAIN = "local"

#-------------------------------------------------------------------------------
# Array of VMs

VMS = [
  {
    # name of the vm as tracked by vagrant
    VM_VAGRANT_NAME: "rke2n01",

    # name and description of the vm, as it shows up in Virtualbox
    VM_VBOX_NAME: "rke2n01",
    VM_VBOX_DESCRIPTION: "rke2n01 dev",

    # static IP address
    VM_IP: "192.168.56.21",

    # short and long host names
    VM_HOSTNAME_SHORT: "rke2n01",
    VM_HOSTNAME_LONG: "rke2n01." + VM_ALL_DOMAIN,

    # forwarded port for connecting with
    #     ssh -p <VM_SSH_PORT> vagrant@localhost
    VM_SSH_PORT: 2201,
    VM_FW_PORTS: [
      { NAME: "http", H: 8080, G: 80 },
    ],
    BOX: VM_ALL_BOX,
    VM_CPU: VM_ALL_CPU,
    VM_MEM: VM_ALL_MEM,
    VM_NET: VM_ALL_NET
  },
  {
    VM_VAGRANT_NAME: "dlkdc02",
    VM_VBOX_NAME: "BDP DEV KDC02",
    VM_VBOX_DESCRIPTION: "BDP DEV KDC, OpenLDAP, BIND9 (DNS)",
    VM_IP: "192.168.56.22",
    VM_HOSTNAME_SHORT: "dlkdc02",
    VM_HOSTNAME_LONG: "dlkdc02." + VM_ALL_DOMAIN,
    VM_SSH_PORT: 2202,
    VM_FW_PORTS: [
      { NAME: "kdc", H: 9088, G: 88 },
      { NAME: "ldap", H: 9389, G: 389 },
      { NAME: "ldaps", H: 9636, G: 636 },
      { NAME: "http", H: 4443, G: 8080 },
    ],
    BOX: VM_ALL_BOX,
    VM_CPU: VM_ALL_CPU,
    VM_MEM: VM_ALL_MEM,
    VM_NET: VM_ALL_NET
  },
  {
    VM_VAGRANT_NAME: "dltest01",
    VM_VBOX_NAME: "BDP DEV test VM",
    VM_VBOX_DESCRIPTION: "BDP DEV VM to test BDP services",
    VM_IP: "192.168.56.23",
    VM_HOSTNAME_SHORT: "dltest01",
    VM_HOSTNAME_LONG: "dltest01." + VM_ALL_DOMAIN,
    VM_SSH_PORT: 2203,
    VM_FW_PORTS: [],
    BOX: VM_ALL_BOX,
    VM_CPU: VM_ALL_CPU,
    VM_MEM: VM_ALL_MEM,
    VM_NET: VM_ALL_NET
  }
]

Vagrant.configure("2") do |config|
  # loop over array of VM definitions using index i
  (0..(VMS.length - 1)).each do |i|
    config.vm.define VMS[i][:VM_VAGRANT_NAME] do |node|
      node.vm.box = VMS[i][:BOX]

      # set forwarding port for ssh
      node.vm.network :forwarded_port, guest: 22, host: VMS[i][:VM_SSH_PORT],
                      id: 'ssh'

      # set other forwarding ports
      (0..(VMS[i][:VM_FW_PORTS].length - 1)).each do |p|
        node.vm.network :forwarded_port,
                        id: VMS[i][:VM_FW_PORTS][p][:NAME],
                        host: VMS[i][:VM_FW_PORTS][p][:H],
                        guest: VMS[i][:VM_FW_PORTS][p][:G]
      end

      # define a second, host-only network
      node.vm.network :private_network, ip: VMS[i][:VM_IP]

      node.vm.hostname = VMS[i][:VM_HOSTNAME_LONG]

      # ssh forwarding allows to use my ssh keys defined on the host from within
      # the vms. I don't know how this works. See
      # https://dev.to/levivm/how-to-use-ssh-and-ssh-agent-forwarding-more-secure-ssh-2c32
      node.ssh.forward_agent = true

      # virtualbox settings
      node.vm.provider "virtualbox" do |vb|
        vb.cpus = VMS[i][:VM_CPU]
        vb.memory = VMS[i][:VM_MEM]
        vb.name = VMS[i][:VM_VBOX_NAME]
        vb.customize ["modifyvm", :id, "--description",
                      VMS[i][:VM_VBOX_DESCRIPTION]]

        # the io-apic affects CPU usage on some guests
        vb.customize ["modifyvm", :id, "--ioapic", "on"]

        # dns settings to use the host dns resolver, see
        # http://station.clancats.com/3-vagrant-settings-you-should-check-out-to-optimize-your-vm/
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]

        # set the network type of the second adapter
        vb.customize ["modifyvm", :id, "--nic2", VMS[i][:VM_NET]]
      end

      # add host names to /etc/hosts
      node.vm.provision :hosts do |provisioner|
        provisioner.sync_hosts = false
        (0..(VMS.length - 1)).each do |j|
          provisioner.add_host VMS[j][:VM_IP], [VMS[j][:VM_HOSTNAME_SHORT],
                                                VMS[j][:VM_HOSTNAME_LONG]]
        end
      end

      # add my public key, so I can ssh as user vagrant with my public key
      #     ssh -p <port> vagrant@localhost
      node.vm.provision "file", source: "~/.ssh/id_rsa.pub",
                        destination: "~/.ssh/" + ENV["USER"] + ".pub"
      node.vm.provision "shell",
                        inline: "cat /home/vagrant/.ssh/" + ENV["USER"] +
                        ".pub >> /home/vagrant/.ssh/authorized_keys"
    end
  end
end
