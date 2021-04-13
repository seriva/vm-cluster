# -*- mode: ruby -*-
# vi: set ft=ruby :

cfg = {
  :provider => "hyperv", # Set the which vagrant provider we want to use. "hyperv" on Windows and "libvirt" on Linux and MacOS.
  :group => "group1", # Group included in a VM name. 
  :vm_count => 4, # If set to 0 a set of VM`s will be created with the definitions in boxes below. Of > 0 then the general cfg (vm_cpus, vm_memory) is used.
  :vm_prefix => "vm", # Prefix that will be used for the vm name and hostname when VM`s will notbe created with the definitions in boxes below.
  :vm_cpus => 2, #Number of CPU cores each VM should have when using the vm_count.
  :vm_memory => 2048, # Amount of memory each VM should have when using the vm_count.
  :vm_box => "generic/ubuntu1804",  #"generic/rhel7", "centos/7", # Image to use for the VM`s.
  :ssh_priv => "/path/to/ssh/private/key",  # Path of the private SSH key that you want to use to connect to the VMs
  :ssh_public => "/path/to/ssh/public/key",  # Path of the public SSH key that you want to use to connect to the VMs
  :network_switch => "switchname"  # HYPER-V ONLY - The network switch you want to use to connect the VMs to. This must be created in Hyper-V manager before hand and needs to be an external switch.
}

# if vm_count is set to 0 the bolow config will be used to create the VMs with the specifications.
boxes = [
  {
    :name => "Kubernetes Master", 
    :hostname => "kmaster",
    :memory => 2048,
    :cpus => 2
  },
  {
    :name => "Kubernetes Node 1", 
    :hostname => "knode1",
    :memory => 2048,
    :cpus => 2
  }, 
  {
    :name => "Kubernetes Node 2", 
    :hostname => "knode2",
    :memory => 2048,
    :cpus => 2
  }
]
  
if cfg[:vm_count] > 0
  boxes = []
  for i in 1..cfg[:vm_count] do
    m = {
      :name => cfg[:vm_prefix] + '-' + i.to_s, 
      :hostname => cfg[:vm_prefix] + '-' + i.to_s,
      :memory => cfg[:vm_memory],
      :cpus => cfg[:vm_cpus], 
    } 
    boxes << m
  end
end
  
Vagrant.configure(2) do |config|
  config.vm.box = "#{cfg[:vm_box]}"
  if cfg[:provider] == "hyperv"
    config.vm.network "public_network", bridge: "#{cfg[:network_switch]}"
  end

  config.vm.provision "file", source: "#{cfg[:ssh_priv]}", destination: "/home/vagrant/.ssh/id_rsa"
  public_key = File.read("#{cfg[:ssh_public]}") 
  config.vm.provision :shell, :inline =>"
          echo 'Copying SSH Keys to the VM'
          mkdir -p /home/vagrant/.ssh
          chmod 700 /home/vagrant/.ssh
          echo '#{public_key}' >> /home/vagrant/.ssh/authorized_keys
          chmod -R 600 /home/vagrant/.ssh/authorized_keys
          echo 'Host *' >> /home/vagrant/.ssh/config
          echo 'StrictHostKeyChecking no' >> /home/vagrant/.ssh/config
          echo 'UserKnownHostsFile /dev/null' >> /home/vagrant/.ssh/config
          chmod -R 600 /home/vagrant/.ssh/config
          echo 'Connection information:'
          ip=$(hostname -I | awk '{print $1}')
          echo ip: $ip
          echo hostname: $HOSTNAME
          ", privileged: false
  
  boxes.each do |machine|
    hostname = machine[:hostname]
    config.vm.define "#{hostname}" do |m|
      m.vm.hostname = hostname
      m.vm.provider cfg[:provider] do |v|
        v.cpus = machine[:cpus]
        v.memory = machine[:memory]
        if cfg[:provider] == "libvirt"
          v.title = cfg[:group] + ": " + machine[:name]
        end
        if cfg[:provider] == "hyperv"
          v.vmname = cfg[:group] + ": " + machine[:name]
          v.linked_clone = true
          v.maxmemory = machine[:memory]
          v.mac = "525400" + Array.new(6){[*"A".."F", *"0".."9"].sample}.join
        end;
      end
    end
  end
end
