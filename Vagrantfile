# -*- mode: ruby -*-
# vi: set ft=ruby :

# Para aprovechar este Vagrantfile necesita Vagrant y Virtualbox instalados:
#
#   * Virtualbox
#
#   * Vagrant
#
#   * Plugins de Vagrant:
#       + vagrant-proxyconf y su configuracion si requiere de un Proxy para salir a Internet
#       + vagrant-cachier
#       + vagrant-disksize
#       + vagrant-share
#       + vagrant-vbguest

VAGRANTFILE_API_VERSION = "2"

post_up_message_blindmaster0 = <<POST_UP_MESSAGE
------------------------------------------------------
Infraestructura DNS - Blind Master

* PowerDNS master con datos en MySQL mysqlmaster0
* API PowerDNS atiende en 192.168.33.10:8001
* DNS atiende 192.168.44.10:53
* nsedit atiende en 192.168.33.10:443

* https://nsedit.infra.ballardini.com.ar/

------------------------------------------------------
POST_UP_MESSAGE


#generic_box = "alpine-linux/alpine-x86_64" # Alpine not supported Virtualbox Guest Additions
#generic_box = "debian/bullseye64" # 2021-04-15 no instalan las guest additions
#generic_box = "debian/buster64" # usa rsync para el sync folder

generic_box = "ubuntu/focal64"
#generic_box = "debian/contrib-buster64"


# Order is important => precedence
# mysqlmaster0 < blindmaster0
# mysqlmaster0 < ipam0
# testinterna0
boxes = [
    {
        :name => "mysqlmaster0",
        :eth1 => "192.168.33.110", :netmask1 => "255.255.255.0",
        :eth2 => "192.168.44.110", :netmask2 => "255.255.255.0",
        :mem => "1024", :cpu => "1",
        :box => generic_box,
        :autostart => true
    },
    {
        :name => "blindmaster0",
        :eth1 => "192.168.33.10", :netmask1 => "255.255.255.0",
        :eth2 => "192.168.44.10", :netmask2 => "255.255.255.0",
        :mem => "1024", :cpu => "1",
        :box => generic_box,
        :autostart => true,
        :post_up_message => post_up_message_blindmaster0
    },
    {
        :name => "ipam0",
        :eth1 => "192.168.33.20", :netmask1 => "255.255.255.0",
        :eth2 => "192.168.44.20", :netmask2 => "255.255.255.0",
        :mem => "1024", :cpu => "1",
        :box => "debian/buster64",
        :autostart => false
    },
    {
        :name => "testinterna0",
        :eth1 => "192.168.33.100", :netmask1 => "255.255.255.0",
        :eth2 => "192.168.44.100", :netmask2 => "255.255.255.0",
        :mem => "1024", :cpu => "1",
        :box => generic_box,
        :autostart => false
    },
]

DOMAIN   = "infra.ballardini.com.ar"



Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|


  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.manage_guest = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true

    # uso cachier con NFS solamente si el hostmanager gestiona los nombres en /etc/hosts del host
    if Vagrant.has_plugin?("vagrant-cachier")

      config.cache.auto_detect = false
      # W: Download is performed unsandboxed as root as file '/var/cache/apt/archives/partial/xyz' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)

      config.cache.synced_folder_opts = {
        owner: "_apt"
      }
      # Configure cached packages to be shared between instances of the same base box.
      # More info on http://fgrehm.viewdocs.io/vagrant-cachier/usage
      config.cache.scope = :box
   end

  end

  boxes.each do |nodo_opts|
    config.vm.define nodo_opts[:name], autostart: nodo_opts[:autostart] do |nodo|
      nodo.vm.hostname = nodo_opts[:name]
      nodo.vm.box = nodo_opts[:box]

      if nodo_opts.key?(:post_up_message)
        nodo.vm.post_up_message = nodo_opts[:post_up_message]
      end
      nodo.vm.boot_timeout = 3600
      nodo.vm.box_check_update = true
      nodo.ssh.forward_agent = true
      nodo.ssh.forward_x11 = true

      nodo.vm.provider :virtualbox do |vb|
        vb.gui = false
        vb.cpus = nodo_opts[:cpu]
        vb.memory = nodo_opts[:mem]

        # https://www.virtualbox.org/manual/ch08.html#vboxmanage-modifyvm mas parametros para personalizar en VB
      end

      if Vagrant.has_plugin?("vagrant-hostmanager")
        nodo.hostmanager.aliases = %W(#{nodo_opts[:name]}.#{DOMAIN} )
      end

      if Vagrant.has_plugin?("vagrant-vbguest") then
        nodo.vbguest.auto_update = true
        nodo.vbguest.no_install = false
      end

      nodo.vm.synced_folder ".", "/vagrant", disabled: false, SharedFoldersEnableSymlinksCreate: false
      nodo.vm.network :private_network, ip: nodo_opts[:eth1], :netmask => nodo_opts[:netmask1]                           # mgmt
      nodo.vm.network :private_network, ip: nodo_opts[:eth2], :netmask => nodo_opts[:netmask2], virtualbox__intnet: true # infra
    end
  end


    ##
    # Aprovisionamiento
    #
    config.vm.provision "fix-no-tty", type: "shell" do |s|
        s.privileged = false
        s.inline = "sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile"
    end

    config.vm.provision "actualiza", type: "shell" do |s|  # http://foo-o-rama.com/vagrant--stdin-is-not-a-tty--fix.html
        s.privileged = false
        s.inline = <<-SHELL
          export DEBIAN_FRONTEND=noninteractive
          export APT_LISTCHANGES_FRONTEND=none
          export APT_OPTIONS=' -y --allow-downgrades --allow-remove-essential --allow-change-held-packages -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold '

          sudo -E apt-get --purge remove apt-listchanges -y > /dev/null 2>&1
          sudo -E apt-get update -y -qq > /dev/null 2>&1
          sudo dpkg-reconfigure --frontend=noninteractive libc6 > /dev/null 2>&1
          [ $( lsb_release -is ) != "Debian" ] && sudo -E apt-get install linux-image-generic ${APT_OPTIONS}
          sudo -E apt-get upgrade ${APT_OPTIONS} > /dev/null 2>&1
          sudo -E apt-get dist-upgrade ${APT_OPTIONS} > /dev/null 2>&1
          sudo -E apt-get autoremove -y > /dev/null 2>&1
          sudo -E apt-get autoclean -y > /dev/null 2>&1
          sudo -E apt-get clean > /dev/null 2>&1
        SHELL
    end

    config.vm.provision "ssh_pub_key", type: :shell do |s|
      begin
          ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
          s.inline = <<-SHELL
            mkdir -p /root/.ssh/
            touch /root/.ssh/authorized_keys
            echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
            echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
          SHELL
      rescue
          puts "No hay claves publicas en el HOME de su pc"
          s.inline = "echo OK sin claves publicas"
      end
    end

    config.vm.provision "provision", type: :ansible do |ansible|
      ansible.playbook = "provision/site.yml"
      ansible.config_file = "./vagrant-inventory/ansible.cfg"
      ansible.inventory_path = "./vagrant-inventory/"
      ansible.verbose= "-vv"
      ansible.become = false
      # heredo la configuracion de Proxy del entorno del host Vagrant:
      ansible.extra_vars = {
        organizacion: "My organization",
        all_proxy:   ENV['all_proxy']   || ENV['http_proxy']  || "",
        http_proxy:  ENV['http_proxy']  || "",
        https_proxy: ENV['https_proxy'] || "",
        ftp_proxy:   ENV['ftp_proxy']   || "",
        no_proxy:    ENV['no_proxy']    || "",
      }
    end

    config.vm.provision "test", type: :ansible do |ansible|
      ansible.playbook = "provision/goss-tests.yml"
      ansible.config_file = "./vagrant-inventory/ansible.cfg"
      ansible.inventory_path = "./vagrant-inventory/"
      ansible.verbose= "-vv"
      ansible.become = false
      # heredo la configuracion de Proxy del entorno del host Vagrant:
      ansible.extra_vars = {
        organizacion: "My organization",
        all_proxy:   ENV['all_proxy']   || ENV['http_proxy']  || "",
        http_proxy:  ENV['http_proxy']  || "",
        https_proxy: ENV['https_proxy'] || "",
        ftp_proxy:   ENV['ftp_proxy']   || "",
        no_proxy:    ENV['no_proxy']    || "",
      }
    end

end
