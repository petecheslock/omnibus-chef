# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'bundler/setup'
require 'omnibus/vagrant/omnibus'
require 'pathname'

module Vagrant
  module Guest
    class FreeBSD < Base
      def configure_networks(networks)
        vm.channel.sudo("sed -i '' -e '/^#VAGRANT-BEGIN/,/^#VAGRANT-END/ d' /etc/rc.conf")

        networks.each do |network|
          entry = TemplateRenderer.render("guests/freebsd/network_#{network[:type]}",
                                          :options => network)
          temp = Tempfile.new("vagrant")
          temp.binmode
          temp.write(entry)
          temp.close
          vm.channel.upload(temp.path, "/tmp/vagrant-network-entry")
          vm.channel.sudo("su -m root -c 'cat /tmp/vagrant-network-entry >> /etc/rc.conf'")
          if network[:type].to_sym == :static
            vm.channel.sudo("ifconfig em#{network[:interface]} inet #{network[:ip]} netmask #{network[:netmask]}")
          elsif network[:type].to_sym == :dhcp
            vm.channel.sudo("dhclient em#{network[:interface]}")
          end
        end
      end
    end
  end
end

Vagrant::Config.run do |config|

  host_ip_counter = 100

  config.vm.define 'ubuntu-10.04' do |c|
    c.vm.box     = "opscode-ubuntu-10.04"
    c.vm.box_url = "http://opscode-vm.s3.amazonaws.com/vagrant/boxes/opscode-ubuntu-10.04.box"
    configure_vm c
  end

  config.vm.define 'ubuntu-11.04' do |c|
    c.vm.box     = "opscode-ubuntu-11.04"
    c.vm.box_url = "http://opscode-vm.s3.amazonaws.com/vagrant/boxes/opscode-ubuntu-11.04.box"
    configure_vm c
  end

  config.vm.define 'ubuntu-12.04' do |c|
    c.vm.box     = "opscode-ubuntu-12.04"
    c.vm.box_url = "http://opscode-vm.s3.amazonaws.com/vagrant/boxes/opscode-ubuntu-12.04.box"
    configure_vm c
  end

  config.vm.define 'centos-5.5' do |c|
    c.vm.box     = "opscode-centos-5.5"
    c.vm.box_url = "http://opscode-vm.s3.amazonaws.com/vagrant/boxes/opscode-centos-5.5.box"
    configure_vm c
  end

  config.vm.define 'centos-5.7' do |c|
    c.vm.box     = "opscode-centos-5.7"
    c.vm.box_url = "http://opscode-vm.s3.amazonaws.com/vagrant/boxes/opscode-centos-5.7.box"
    configure_vm c
  end

  config.vm.define 'centos-6.0' do |c|
    c.vm.box     = "opscode-centos-6.0"
    c.vm.box_url = "http://opscode-vm.s3.amazonaws.com/vagrant/boxes/opscode-centos-6.0.box"
    configure_vm c
  end

  config.vm.define 'centos-6.2' do |c|
    c.vm.box     = "opscode-centos-6.2"
    c.vm.box_url = "http://opscode-vm.s3.amazonaws.com/vagrant/boxes/opscode-centos-6.2.box"
    configure_vm c
  end

  config.vm.define 'freebsd-9.1-i386' do |c|
    c.vm.box     = "taximagic-freebsd-9.1-i386"
    c.vm.guest   = :freebsd
    #c.vm.box_url = "http://opscode-vm.s3.amazonaws.com/vagrant/boxes/opscode-centos-6.2.box"
    c.vm.network :hostonly, "172.30.30.#{host_ip_counter+=1}"
    configure_vm c, :use_nfs => true
  end

  config.vm.define 'freebsd-9.1' do |c|
    c.vm.box     = "taximagic-freebsd-9.1"
    c.vm.guest   = :freebsd
    #c.vm.box_url = "http://opscode-vm.s3.amazonaws.com/vagrant/boxes/opscode-centos-6.2.box"
    c.vm.network :hostonly, "172.30.30.#{host_ip_counter+=1}"
    configure_vm c, :use_nfs => true
    c.vm.provision :shell, :inline => "sed -i '' -E 's%^([^#].*):setenv=%\1:setenv=PACKAGESITE=ftp://ftp.freebsd.org/pub/FreeBSD/ports/amd64/packages-9-stable/Latest,%' /etc/login.conf"
  end

end

def configure_vm(config, options={})

  use_nfs = options[:use_nfs] || false

  config.vm.share_folder("v-root", "/vagrant", ".", :nfs => use_nfs)

  # Share an additional folder to the guest VM. The first argument is
  # an identifier, the second is the path on the guest to mount the
  # folder, and the third is the path on the host to the actual folder.
  # config.vm.share_folder "v-data", "/vagrant_data", "../data"
  config.vm.share_folder "omnibus-chef", "~/omnibus-chef", Pathname.new(File.expand_path("..", __FILE__)).realpath.to_s, :nfs => use_nfs
  config.vm.share_folder "omnibus-ruby", "~/omnibus-ruby", Pathname.new(File.expand_path("../../omnibus-ruby", __FILE__)).realpath.to_s, :nfs => use_nfs
  config.vm.share_folder "omnibus-software", "~/omnibus-software", Pathname.new(File.expand_path("../../omnibus-software", __FILE__)).realpath.to_s, :nfs => use_nfs

  # Enable provisioning with chef solo, specifying a cookbooks path (relative
  # to this Vagrantfile), and adding some recipes and/or roles.
  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = ["cookbooks", File.join(Bundler.definition.specs["omnibus"][0].gem_dir, "cookbooks")]
    chef.add_recipe "omnibus"
    chef.nfs = use_nfs
    chef.json = {
      "omnibus" => {
        "install-dirs" => ["/opt/chef", "/opt/chef-server"]
      }
    }
  end

  # Enable SSH agent forwarding for git clones
  config.ssh.forward_agent = true

  # Give enough horsepower to build PC without taking all day
  # or several hours worth of swapping  Disable support we don't need
  config.vm.customize [
    "modifyvm", :id,
    "--memory", "1536",
    "--cpus", "2",
    "--usb", "off",
    "--usbehci", "off",
    "--audio", "none"
  ]

  config.omnibus.path = "~/omnibus-chef"
end
