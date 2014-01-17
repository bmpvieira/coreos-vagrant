# -*- mode: ruby -*-
# # vi: set ft=ruby :

ip      = ENV.fetch("DOCKER_IP", "192.168.42.43")
port    = ENV.fetch("DOCKER_PORT", "4243")
cpus    = ENV.fetch("DOCKER_CPUS", "1")
memory  = ENV.fetch("DOCKER_MEMORY", "512")
args    = ENV.fetch("DOCKER_ARGS", "-H unix:///var/run/docker.sock -H tcp://0.0.0.0:#{port}")
share    = ENV.fetch("DOCKER_SHARE", ".")


Vagrant.configure("2") do |config|
  config.vm.box = "coreos"
  config.vm.box_url = "http://storage.core-os.net/coreos/amd64-generic/dev-channel/coreos_production_vagrant.box"
  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--memory", memory]
    v.customize ["modifyvm", :id, "--cpus", cpus]
  end

  # Fix docker not being able to resolve private registry in VirtualBox
  config.vm.provider :virtualbox do |vb, override|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  config.vm.provider :vmware_fusion do |vb, override|
    override.vm.box_url = "http://storage.core-os.net/coreos/amd64-generic/dev-channel/coreos_production_vagrant_vmware_fusion.box"
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.vm.network "private_network", :ip => ip
  config.vm.provision :shell, :inline => <<-PREPARE
    INITD=/media/state/units/dockerd.service
    if ! test -e $INITD >/dev/null; then
      echo "---> Configuring docker to bind to tcp/#{port} and restarting"
      sudo sed \
        -e 's|network.target|network.target\\nConflicts=docker.service|' \
        -e 's|docker -d|docker -d #{args}|' \
        -e 's|multi-user.target|local.target|' \
        /usr/lib/systemd/system/docker.service > $INITD
    fi
    sudo systemctl restart local-enable.service
  PREPARE
  config.vm.synced_folder "#{share}", "/home/core/share", id: "core", :nfs => true,  :mount_options   => ['nolock,vers=3,udp']
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "forwarded_port", guest: 3000, host: 3000
  config.vm.network "forwarded_port", guest: 9292, host: 9292
end
