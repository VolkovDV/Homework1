
# Describe VMs
MACHINES = {
  # VM name "kernel update"
  :"kernel-update" =>{
              :box_name => "VolkovDV/Centos7.8",
              # VM CPU count
              :cpus => 2,
              # VM RAM size (Mb)
              :memory => 1024,
              # networks
              :net => [],
              # forwarded ports
              :forwarded_port => [],
              :disks => {
		    :sata1 => {
		      :dfile => './sata1.vdi',
		      :size => 250,
		      :port => 1},
		    :sata2 => {
		      :dfile => './sata2.vdi',
		      :size => 250, # Megabytes
		      :port => 2},
		    :sata3 => {
		      :dfile => './sata3.vdi',
		      :size => 250,
		      :port => 3},
		    :sata4 => {
		      :dfile => './sata4.vdi',
		      :size => 250, # Megabytes
		      :port => 4}
                    }
	      }
}
Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    # Disable shared folders
    config.vm.synced_folder ".", "/vagrant", disabled: true
    # Apply VM config
    config.vm.define boxname do |box|
      # Set VM base box and hostname
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      # Additional network config if present
      if boxconfig.key?(:net)
        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
      end
      # Port-forward config if present
      if boxconfig.key?(:forwarded_port)
        boxconfig[:forwarded_port].each do |port|
          box.vm.network "forwarded_port", port
        end
      end
      # VM resources config
      box.vm.provider "virtualbox" do |v|
        # Set VM RAM size and CPU count
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
        # For disks
        v.customize ["modifyvm", :id, "--memory", "1024"]
        needsController = false
        boxconfig[:disks].each do |dname, dconf|
	    unless File.exist?(dconf[:dfile])
	        v.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                needsController =  true
            end

        end
        if needsController == true
            v.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
            boxconfig[:disks].each do |dname, dconf|
                v.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
            end
        end
    end
        box.vm.provision "shell", inline: <<-SHELL
	      mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
	      yum install -y mdadm smartmontools hdparm gdisk
              mdadm --zero-superblock --force /dev/sd{b,c,d,e}
              mdadm --create --verbose /dev/md0 -l 6 -n 4 /dev/sd{b,c,d,e}


        SHELL
  end
end
end
