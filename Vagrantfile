Vagrant.configure("2") do |config|

  config.vm.define "node01" do |node01|
    node01.vm.box = "ubuntu/bionic64"
    node01.vm.hostname = "node01.box"
    node01.vm.network :private_network, ip: "192.168.53.2"
    node01.vm.network :forwarded_port, guest: 22, host: 2222, id: "ssh"
    node01.vm.network :forwarded_port, guest: 6443, host: 6443
    node01.vm.provider :virtualbox do |vb|
      vb.customize [
        "modifyvm", :id,
        "--memory", "1024",
      ]
    end
    config.vm.synced_folder ".", "/vagrant", disabled: true
  end

  config.vm.define "node02" do |node02|
    node02.vm.box = "ubuntu/bionic64"
    node02.vm.hostname = "node02.box"
    node02.vm.network :private_network, ip: "192.168.53.3"
    node02.vm.network :forwarded_port, guest: 22, host: 2233, id: "ssh"
    node02.vm.provider :virtualbox do |vb|
      vb.customize [
        "modifyvm", :id,
        "--memory", "1024",
      ]
    end
    config.vm.synced_folder ".", "/vagrant", disabled: true
  end

end