Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=755,fmode=755"]
  config.vm.provision "shell", inline: "apt -y install fakeroot dpkg-dev lintian"
end
