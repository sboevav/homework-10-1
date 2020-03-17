# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']

MACHINES = {
  :'prod-nginx-01' => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.150',
  },
  :'prod-nginx-02' => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.151',
  },
  :'staging-nginx-01' => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.200',
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "256"]
        #   vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
          vb.name = boxname.to_s

        #   boxconfig[:disks].each do |dname, dconf|
        #       unless File.exist?(dconf[:dfile])
        #         vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
        #       end
        #       vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
        # end
          end
      box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
      SHELL

      end
   end
end
