# -*- mode: ruby -*-
# vi: set ft=ruby :

# Elastic Agent Setup
agent_url         = 'https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-7.12.0-linux-x86_64.tar.gz'

# YOU MUST REPLACE THESE VALUES WITH YOUR OWN. THESE AREN'T EVEN REAL
kibana_url        = 'https://abcdef1234567890abcdef0123456789.us-central1.gcp.cloud.es.io:443'  # Replace this sample with your Kibana server
enrollment_token  = 'XUyAzGtlSymvOEEQCqGv2cYoztLMeoiGYFzlrp5E5JPOEydHR+ERycb44A==' # Replace this random token with the one given by Kibana Fleet

require 'ipaddr'

# to make sure the nodes are created in order, we
# have to force a --no-parallel execution.
ENV['VAGRANT_NO_PARALLEL'] = 'yes'

ubuntu_number_of_nodes      = 2
ubuntu_first_node_ip        = '192.168.53.101'
ubuntu_node_ip_address      = IPAddr.new ubuntu_first_node_ip
ubuntu_first_node_vpn_ip    = '10.2.0.101'
ubuntu_node_vpn_ip_address  = IPAddr.new ubuntu_first_node_vpn_ip

Vagrant.configure(2) do |config|
  config.vm.box = "generic/ubuntu2004"

  config.vm.provider 'libvirt' do |lv, config|
    lv.cpus = 2
    lv.cpu_mode = 'host-passthrough'
    lv.nested = true
    lv.keymap = 'pt'
    config.vm.synced_folder '.', '/vagrant', type: 'nfs'
  end

  config.vm.provider 'virtualbox' do |vb|
    vb.linked_clone = true
    vb.cpus = 2
    config.vm.synced_folder ".", "/vagrant", disabled: false
  end

  (1..ubuntu_number_of_nodes).each do |n|
    name = "u#{n}"
    fqdn = "#{name}.example.test"
    ip_address = ubuntu_node_ip_address.to_s; ubuntu_node_ip_address = ubuntu_node_ip_address.succ
    vpn_ip_address = ubuntu_node_vpn_ip_address.to_s; ubuntu_node_vpn_ip_address = ubuntu_node_vpn_ip_address.succ

    config.vm.define name do |config|
      config.vm.provider 'libvirt' do |lv, config|
        lv.memory = 512
      end
      config.vm.provider 'virtualbox' do |vb|
        vb.memory = 512
      end
      config.vm.hostname = fqdn
      config.vm.network :private_network, ip: ip_address, libvirt__forward_mode: 'none', libvirt__dhcp_enabled: false
      config.vm.provision 'shell',
        path: 'provision-base.sh',
        env: {
          'AGENT_URL' => agent_url,
          'KIBANA_URL' => kibana_url,
          'ENROLLMENT_TOKEN' => enrollment_token
        }

      config.vm.provision 'shell', path: 'provision-wireguard.sh', args: [ip_address, vpn_ip_address]
      config.trigger.before :destroy do |trigger|
        trigger.ruby do |env, machine|
          FileUtils.rm_f("tmp/wg-peer-#{name}.conf")
        end
      end
      # update all the machines wireguard configuration with all the other peers.
      config.trigger.after :up do |trigger|
        trigger.ruby do |env, machine|
          # see https://github.com/hashicorp/vagrant/blob/v2.2.7/lib/vagrant/plugin/v2/trigger.rb
          # see https://github.com/hashicorp/vagrant/blob/v2.2.7/lib/vagrant/environment.rb
          # see https://github.com/hashicorp/vagrant/blob/v2.2.7/lib/vagrant/machine.rb
          # see https://github.com/hashicorp/vagrant/blob/v2.2.7/lib/vagrant/machine_state.rb
          # see https://github.com/hashicorp/vagrant/blob/v2.2.7/lib/vagrant/plugin/v2/communicator.rb
          env.active_machines.each do |machine_name, machine_provider|
            m = env.machine(machine_name, machine_provider)
            if m.state.id == :running
              m.ui.info('Updating WireGuard peers...')
              m.communicate.sudo('/vagrant/wg-update-peers.sh') do |type, data|
                m.ui.info(data.chomp)
              end
            end
          end
        end
      end
    end
  end
end

