# -*- mode: ruby -*-
# vi: set ft=ruby :
#

NODES = 3
DISKS = 2

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider :libvirt do |v,override|
        override.vm.box = "generic/ubuntu2204"
        override.vm.synced_folder '.', '/home/vagrant/sync', disabled: true
    end

    (0..NODES-1).each do |i|
        config.vm.define "node#{i}" do |node|
            node.vm.network :private_network, ip: "192.168.10.10#{i}"

            (0..DISKS-1).each do |d|
                driverletters = ('b'..'z').to_a
                node.vm.host_name = "node#{i}"
                node.vm.provider :libvirt do  |lv|
                    lv.storage :file, :device => "vd#{driverletters[d]}", :path => "disk-#{i}-#{d}.disk", :size => '5G'
                    lv.memory = 8096
                    lv.cpus =8
                end
            end

            if i == (NODES-1)
                node.vm.provision :ansible do |ansible|
                    ansible.limit = "all"
                    ansible.playbook = "site.yml"
                    ansible.groups = {
                        "gluster" => (0..NODES-1).map {|j| "node#{j}"},
                        "docker" => (0..NODES-1).map {|j| "node#{j}"},
                        # "heketi" => ["node0"]
                    }
                end
            end
        end
    end
end