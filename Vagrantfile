VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box_url = "http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_centos-7.0_chef-provisionerless.box"
  config.ssh.insert_key = false

  config.vm.define "app" do |web|
    web.vm.box = "kibana_practice"
    web.vm.network "private_network", ip: "192.168.33.11"
  end
end

