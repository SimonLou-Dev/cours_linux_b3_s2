Vagrant.configure("2") do |config|

    # Select box
    config.vm.box = "rockylinux/9"
    config.vm.box_url = "rockylinux/9"

    etcHosts = ""

    # some settings for common server (not for haproxy)
    common = <<-SHELL
    sudo yum update -qq 2>&1 >/dev/null
    sudo yum install -y -qq unzip iftop curl git vim python3-pip tree net-tools telnet policycoreutils-python-utils 2>&1 >/dev/null
    sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
    sudo systemctl restart sshd
    SHELL

    # Listserveur
    nodes = [
      { :name => "rsyslog-server", :ip => "192.168.56.10", :cpus => 1, :mem => 1024, :disk => '10GB' },
      { :name => "rsyslog-client", :ip => "192.168.56.11", :cpus => 1, :mem => 1024, :disk => '10GB' },
    ]

    # prepare /etc/host
    nodes.each do |node|
        etcHosts += "echo '" + node[:ip] + "   " + node[:name] + "'>> /etc/hosts" + "\n"
    end #end NODES

    nodes.each do |node|
      config.vm.define node[:name] do |cfg|
        cfg.vm.hostname = node[:name]
        cfg.vm.network 'public_network', ip: node[:ip]
        if Vagrant.has_plugin?("vagrant-disksize")
          cfg.disksize.size = node[:disk] if Vagrant::Util::Platform.windows?
        end

        cfg.vm.provider "virtualbox" do |v|
            v.customize [ "modifyvm", :id, "--cpus", node[:cpus] ]
            v.customize [ "modifyvm", :id, "--memory", node[:mem] ]
            v.customize [ "modifyvm", :id, "--name", node[:name] ]
            v.customize [ "modifyvm", :id, "--ioapic", "on" ]
            v.customize [ "modifyvm", :id, "--nictype1", "virtio" ]
        end #end provider

        cfg.vm.provider "vmware_desktop" do |v|
            v.vmx["numvcpus"] = node[:cpus].to_s
            v.vmx["memsize"] = node[:mem].to_s
            v.vmx["displayName"] = node[:name]
            # Note : VMware does not support changing the name via `--name` like VirtualBox
        end #end provider

        cfg.vm.provision :shell, :inline => etcHosts
        cfg.vm.provision :shell, :inline => common
      end
    end
  end
