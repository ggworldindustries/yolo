Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end
  
  config.vm.network "forwarded_port", guest: 3000, host: 3000
end
