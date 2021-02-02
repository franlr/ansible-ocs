# -*- mode: ruby -*-
# vi: set ft=ruby :

# Requirements:
#
#   * Virtualbox
#
#   * Vagrant
#
#   * Vagrant plugins:
#       + vagrant-proxyconf (if needed)
#       + vagrant-cachier
#       + vagrant-vbguest

VAGRANTFILE_API_VERSION = "2"

HOSTNAME = "testvm"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    if Vagrant.has_plugin?("vagrant-cachier")
      config.cache.synced_folder_opts = {
        owner: "_apt"
      }
      config.cache.scope = :box
   end

 config.vm.define HOSTNAME do |srv|

    srv.vm.box = "ubuntu/focal64"
    srv.vm.network "private_network", ip: "192.168.33.11"
    srv.vm.boot_timeout = 3600
    srv.vm.box_check_update = false
    srv.ssh.forward_agent = true
    srv.ssh.forward_x11 = true
    srv.vm.hostname = HOSTNAME

    srv.vm.provider :virtualbox do |vb|
      vb.gui = false
      vb.cpus = 2
      vb.memory = "1536"
    end
  end

    config.vm.provision "fix-no-tty", type: "shell" do |s|
        s.privileged = false
        s.inline = "sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile"
    end
    config.vm.provision "operating-system-update", type: "shell" do |s|  # http://foo-o-rama.com/vagrant--stdin-is-not-a-tty--fix.html
        s.privileged = false
        s.inline = <<-SHELL
          export DEBIAN_FRONTEND=noninteractive
          export APT_LISTCHANGES_FRONTEND=none
          export APT_OPTIONS=' -y --allow-downgrades --allow-remove-essential --allow-change-held-packages -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold '

          # https://serverfault.com/a/717770
          sudo ex +"%s@DPkg@//DPkg" -cwq /etc/apt/apt.conf.d/70debconf
          sudo dpkg-reconfigure debconf -f noninteractive -p critical

          sudo apt-get update -y -qq
          sudo apt-get upgrade ${APT_OPTIONS}
          sudo apt-get full-upgrade ${APT_OPTIONS}
          sudo apt-get autoremove -y

        SHELL
    end

    proxy_host_port = ENV['all_proxy'] || ENV['http_proxy']  || ""
    proxy_host_port = if proxy_host_port.empty? then "" else proxy_host_port.scan(/\/\/([0-9\.]*):/)[0][0]+':'+proxy_host_port.scan(/:([0-9]*)$/)[0][0] end

    config.vm.provision "pip-ansible-install", type: "shell" do |s| 
        s.privileged = false
        s.inline = <<-SHELL
          export DEBIAN_FRONTEND=noninteractive
          export APT_LISTCHANGES_FRONTEND=none
          export APT_OPTIONS=' -y --allow-downgrades --allow-remove-essential --allow-change-held-packages -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold '

          sudo apt-get install python3-pip ${APT_OPTIONS}
          sudo -H python3 -m pip install pip setuptools wheel
          sudo -H python3 -m pip install ansible

        SHELL
    end

    config.vm.provision "ansible-provision", type: :ansible, run: "never" do |ansible|
      ansible.playbook = "./tests/test.yml"
      ansible.config_file = "./vagrant-inventory/ansible.cfg"
      ansible.inventory_path = "./vagrant-inventory/"
      ansible.verbose= "-v"
      ansible.become = true

      ansible.extra_vars = {
        all_proxy:   ENV['all_proxy']   || ENV['http_proxy']  || "",
        http_proxy:  ENV['http_proxy']  || "",
        https_proxy: ENV['https_proxy'] || "",
        ftp_proxy:   ENV['ftp_proxy']   || "",
        no_proxy:    ENV['no_proxy']    || "",
      }
    end

end
