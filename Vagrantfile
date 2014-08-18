# -*- coding: utf-8 -*-
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
Vagrant.require_version ">= 1.4.0"

warn <<EOT unless File.directory?(".librarian")
Please run:
  gem install librarian-puppet
  librarian-puppet install --path vendor

EOT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box     = "CentOS6.5-x86_64"
  config.vm.box_url = "https://s3-ap-northeast-1.amazonaws.com/paperboy-vagrant-boxes/CentOS-6.5-x86_64-minimal.box"

  config.vm.provision :shell do |shell|
    shell.inline = <<-'SCRIPT'
      # mysql-libsが入ってるとconflictするので抜いとく
      rpm -qi mysql-libs && rpm -e --nodeps mysql-libs
      # cent6で名前解決が遅いの回避
      (tail -n 1 /etc/resolv.conf | egrep 'options single-request-reopen' /etc/resolv.conf) ||
          echo 'options single-request-reopen' >> /etc/resolv.conf

      # cent6でsudo時のPATHリセットを無効化する
      if (uname -a | grep -q el6) && ! (grep -q secure_path /etc/sudoers.d/vagrant); then
          echo 'Defaults:vagrant !secure_path' >> /etc/sudoers.d/vagrant
      fi
    SCRIPT
  end

  config.vm.provision :puppet do |puppet|
    puppet.manifests_path    = "manifests"
    puppet.manifest_file     = "site.pp"
    puppet.module_path       = ["modules", "roles", "vendor/modules"]
    puppet.hiera_config_path = "hiera.yaml"

    options = ["--verbose", "--environment development", "--vardir /vagrant"]
    options << "--noop"  if ENV['NOOP']
    options << "--debug" if ENV['DEBUG']
    puppet.options = options
  end

  def define_vbox(c, private_ip: nil, memory: 256, cpu: 2)
    c.vm.network :private_network, ip: private_ip if private_ip

    # ref http://vboxmania.net/content/vboxmanage-modifyvm%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89
    c.vm.provider :virtualbox do |vbox|
      # cent6でcpu増やしてsshの反応がもっさりするの回避
      vbox.customize ["modifyvm", :id, "--hpet", "on"]
      vbox.customize ["modifyvm", :id, "--acpi", "off"]

      # IPv6とDNSでのネットワーク遅延対策で追記
      vbox.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]

      # machine spec
      vbox.customize ["modifyvm", :id, "--memory", memory]
      vbox.customize ["modifyvm", :id, "--cpus",   cpu]
    end
  end

  config.vm.define :node001 do |c|
    c.vm.host_name  = "node001.mha.dev"
    define_vbox c, private_ip: '192.168.80.2'
  end

  config.vm.define :node002 do |c|
    c.vm.host_name  = "node002.mha.dev"
    define_vbox c, private_ip: '192.168.80.3'
  end

  config.vm.define :node003 do |c|
    c.vm.host_name  = "node003.mha.dev"
    define_vbox c, private_ip: '192.168.80.4'
  end

  config.vm.define :manager001 do |c|
    c.vm.host_name  = "manager001.mha.dev"
    define_vbox c, private_ip: '192.168.80.100'
  end
end
