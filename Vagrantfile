Vagrant.configure("2") do |config|

  # ------------------------------
  # master
  # ------------------------------
  config.vm.define "edtool-jenkins" do |master|
    master.vm.box = "debian/bullseye64"
    master.vm.hostname = "edtool-jenkins"
    master.vm.network "private_network", ip: "192.168.56.100"

    master.vm.provider :libvirt do |domain|
      domain.memory = 1024
      domain.cpus = 2
    end

    # install java, jenkins, npm, ui5
    master.vm.provision "shell", inline: <<-SHELL
      sudo apt-get install -y curl
      curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
      echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
      sudo apt-get update
      sudo apt-get install -y openjdk-11-jre
      sudo apt-get install -y jenkins
      sudo apt-get install -y npm
      sudo npm install --global @ui5/cli
    SHELL
  end
end