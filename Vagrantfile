# -*- mode: ruby -*-
# vim: set ft=ruby :
home = 'F:/VirtualBoxVMs-Vagrnat/stands-02-disks-master/disks/'


MACHINES = {
  :otuslinux => {
        :box_name => 'centos/7',
        #:ip_addr => '10.10.10.29',
	:disks => {
		:sata1 => {
			:dfile => home + 'sata1.vdi',
			:size => 250,
			:port => 1
		},
		:sata2 => {
            :dfile => home + 'sata2.vdi',
            :size => 250, # Megabytes
			:port => 2
		},
        :sata3 => {
            :dfile => home + 'sata3.vdi',
            :size => 250,
            :port => 3
        },
        :sata4 => {
            :dfile => home + 'sata4.vdi',
            :size => 250, # Megabytes
            :port => 4
        },
        :sata5 => {
            :dfile => home + 'sata5.vdi',
            :size => 250,
            :port => 5
        },
        :sata6 => {
            :dfile => home + 'sata6.vdi',
            :size => 250,
            :port => 6
        }
	}
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          # _network
          # public_  - https://www.vagrantup.com/docs/networking/public_network.html
          # private_ - https://www.vagrantup.com/docs/networking/private_network.html
          #
          # box.vm.network "private_network", ip: boxconfig[:ip_addr]
          box.vm.network "private_network", type: "dhcp"

          box.vm.provider :virtualbox do |vb|
            	  vb.customize ["modifyvm", :id, "--memory", "256"]
                  needsController = false
		  boxconfig[:disks].each do |dname, dconf|
			  unless File.exist?(dconf[:dfile])
				vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                needsController =  true
                          end

		  end
                  if needsController == true
                     vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                     boxconfig[:disks].each do |dname, dconf|
                         vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                     end
                  end
          end

      box.vm.provision "shell", inline: <<-SHELL
	      mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
	      yum install -y mdadm smartmontools hdparm gdisk lvm2-2.02.180-10.el7_6.3.x86_64
	      mdadm --zero-superblock --force /dev/sd{b,c,d,e,f,g}
	      mdadm --create --verbose /dev/md0 -l 5 -n 6 /dev/sd{b,c,d,e,f,g}	      
	      echo "DEVICE partitions" > /etc/mdadm.conf
	      mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm.conf
	      parted -s /dev/md0 mklabel gpt
	      parted /dev/md0 mkpart primary ext4 0% 20%
	      parted /dev/md0 mkpart primary ext4 20% 40%
	      parted /dev/md0 mkpart primary ext4 40% 60%
	      parted /dev/md0 mkpart primary ext4 60% 80%
	      parted /dev/md0 mkpart primary ext4 80% 100%
	      for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
	      mkdir -p /mnt/raid/part{1,2,3,4,5}
	      for i in $(seq 1 5); do mount /dev/md0p$i /mnt/raid/part$i; done
  	  SHELL

      end
  end
end

