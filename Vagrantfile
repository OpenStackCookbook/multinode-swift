# -*- mode: ruby -*-
# vi: set ft=ruby :

# Uncomment the next line to force use of VirtualBox provider when Fusion provider is present
# ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'

nodes = {
    'controller'  => [1, 200],
    'swift'  => [5, 221],
    'swift-proxy'   => [1, 209],
}

Vagrant.configure("2") do |config|
    
  # Virtualbox
  config.vm.box = "bunchc/trusty-x64"
  config.vm.synced_folder ".", "/vagrant", type: "nfs"

  # VMware Fusion / Workstation
  config.vm.provider :vmware_fusion or config.vm.provider :vmware_workstation do |vmware, override|
    override.vm.box = "bunchc/trusty-x64"
    override.vm.synced_folder ".", "/vagrant", type: "nfs"

    # Fusion Performance Hacks
    vmware.vmx["logging"] = "FALSE"
    vmware.vmx["MemTrimRate"] = "0"
    vmware.vmx["MemAllowAutoScaleDown"] = "FALSE"
    vmware.vmx["mainMem.backing"] = "swap"
    vmware.vmx["sched.mem.pshare.enable"] = "FALSE"
    vmware.vmx["snapshot.disabled"] = "TRUE"
    vmware.vmx["isolation.tools.unity.disable"] = "TRUE"
    vmware.vmx["unity.allowCompostingInGuest"] = "FALSE"
    vmware.vmx["unity.enableLaunchMenu"] = "FALSE"
    vmware.vmx["unity.showBadges"] = "FALSE"
    vmware.vmx["unity.showBorders"] = "FALSE"
    vmware.vmx["unity.wasCapable"] = "FALSE"
    vmware.vmx["vhv.enable"] = "TRUE"
  end

  #Default is 2200..something, but port 2200 is used by forescout NAC agent.
  config.vm.usable_port_range= 2800..2900 

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
    config.cache.enable :apt
    config.cache.synced_folder_opts = {
      type: :nfs,
      mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
    }
  else
    puts "[-] WARN: This would be much faster if you ran vagrant plugin install vagrant-cachier first"
  end

  nodes.each do |prefix, (count, ip_start)|
    count.times do |i|
      if prefix == "swift"
        hostname = "%s-%02d" % [prefix, (i+1)]
      else
        hostname = "%s" % [prefix, (i+1)]
      end

      config.vm.define "#{hostname}" do |box|
        box.vm.hostname = "#{hostname}.cook.book"
        box.vm.network :private_network, ip: "172.16.0.#{ip_start+i}", :netmask => "255.255.0.0"
        box.vm.network :private_network, ip: "10.10.0.#{ip_start+i}", :netmask => "255.255.255.0" 
      	box.vm.network :private_network, ip: "192.168.100.#{ip_start+i}", :netmask => "255.255.255.0" 

        box.vm.provision :shell, :path => "#{prefix}.sh"

        # If using Fusion or Workstation
        #box.vm.provider :vmware_fusion or box.vm.provider :vmware_workstation do |v|
        box.vm.provider :vmware_fusion do |v|
          v.vmx["memsize"] = 1024
          if prefix == "controller" 
            v.vmx["memsize"] = 2048
            v.vmx["numvcpus"] = "1"
          end

          if prefix == "swift" 
            v.vmx["memsize"] = 2048
            v.vmx["numvcpus"] = "1"

	    vdiskmanager = '/Applications/VMware\ Fusion.app/Contents/Library/vmware-vdiskmanager'
 
            dir = "#{ENV['HOME']}/vagrant-additional-disk"
 
            unless File.directory?( dir )
                Dir.mkdir dir
            end
 
            file_to_disk = "#{dir}/#{hostname}-sdb.vmdk"
 
            unless File.exists?( file_to_disk )
                `#{vdiskmanager} -c -s 20GB -a lsilogic -t 1 #{file_to_disk}`
            end
 
            v.vmx['scsi0:1.filename'] = file_to_disk
            v.vmx['scsi0:1.present']  = 'TRUE'
            v.vmx['scsi0:1.redo']     = ''
	  end
        end

        box.vm.provider :vmware_workstation do |v|
          v.vmx["memsize"] = 1024
          if prefix == "controller" 
            v.vmx["memsize"] = 2048
            v.vmx["numvcpus"] = "1"
          end

          if prefix == "swift" 
            v.vmx["memsize"] = 2048
            v.vmx["numvcpus"] = "1"

	    vdiskmanager = '/usr/bin/vmware-vdiskmanager'
 
            dir = "#{ENV['HOME']}/vagrant-additional-disk"
 
            unless File.directory?( dir )
                Dir.mkdir dir
            end
 
            file_to_disk = "#{dir}/#{hostname}-sdb.vmdk"
 
            unless File.exists?( file_to_disk )
                `#{vdiskmanager} -c -s 20GB -a lsilogic -t 1 #{file_to_disk}`
            end
 
            v.vmx['scsi0:1.filename'] = file_to_disk
            v.vmx['scsi0:1.present']  = 'TRUE'
            v.vmx['scsi0:1.redo']     = ''
	  end
        end

        # Otherwise using VirtualBox
        box.vm.provider :virtualbox do |vbox|
          # Defaults
          vbox.customize ["modifyvm", :id, "--memory", 1024]
          vbox.customize ["modifyvm", :id, "--cpus", 1]
          if prefix == "controller"
            vbox.customize ["modifyvm", :id, "--memory", 2048]
            vbox.customize ["modifyvm", :id, "--cpus", 1]
          end

          if prefix == "controller"
            vbox.customize ["modifyvm", :id, "--memory", 2048]
            vbox.customize ["modifyvm", :id, "--cpus", 1]

            dir = "#{ENV['HOME']}/vagrant-additional-disk"
 
            unless File.directory?( dir )
                Dir.mkdir dir
            end
 
            file_to_disk = "#{dir}/#{hostname}-sdb.vmdk"
 
            unless File.exists?( file_to_disk )
    		vbox.customize ['createhd', '--filename', file_to_disk, '--size', 20 * 1024]
            end
  	    vbox.customize ['storageattach', :id, '--storagectl', 'IDE Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
  	  end
        end
      end
    end
  end
end
