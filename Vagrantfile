# -*- mode: ruby -*-
# vi: set ft=ruby :

############################################################
# SSH Public Key
############################################################

$ssh_public_key_filename = "#{Dir.home}/.ssh/id_rsa.pub" 

if ENV["SSH_PUBLIC_KEY"]
  $ssh_public_key_filename = ENV["SSH_PUBLIC_KEY"]
end

$ssh_public_key_content = File.readlines($ssh_public_key_filename).first.strip 

############################################################
# Cluster information
############################################################

$single_node = true
$os = "ubuntu"
$ip_prefix = "192.168.100"

$script = <<-SCRIPT
mkdir -p /root/.ssh
echo #{$ssh_public_key_content} >> /root/.ssh/authorized_keys

export HOSTNAME=$(hostname)
sed -i -e "s/127\.0\.0\.1.*$HOSTAME.*$HOSTNAME//" /etc/hosts
SCRIPT

############################################################
# Single node
############################################################

$single_node_ram = 8192
$single_node_cpus = 4

############################################################
# Controllers
############################################################

$controllers_count = 1
$controllers_ram = 2048
$controllers_cpus = 1

############################################################
# Workers
############################################################

$workers_count = 2
$workers_ram = 2048
$workers_cpus = 1

############################################################
# Environment variables
############################################################

if ENV["MULTI_NODE"]
    $single_node = false
end

if ENV["OS"] 
    if ENV["OS"] != "ubuntu" && ENV["OS"] != "centos"
        raise "Unsupported OS: '" + ENV["OS"] + "'"
    end

    $os = ENV["OS"]
end

############################################################
# Summary
############################################################

puts
puts "####################################################"

puts "SSH Public Key: #{$ssh_public_key_filename}"
puts "OS: #{$os}"

if $single_node 
  puts "Setup: Single Node"
  puts "Single Node RAM: #{$single_node_ram}"
  puts "Single Node CPUs: #{$single_node_cpus}"
else
  puts "Setup: Multi Node"
  puts "Controllers Count: #{$controllers_count}"
  puts "Controllers RAM: #{$controllers_ram}"
  puts "Controllers CPUs: #{$controllers_cpus}"
  puts "Workers Count: #{$workers_count}"
  puts "Workers RAM: #{$workers_ram}"
  puts "Workers CPUs: #{$workers_cpus}"
end

puts "####################################################"
puts

############################################################
# Setup
############################################################

if $os == "ubuntu"
  $box = "bento/ubuntu-18.04"
else
  $box = "centos/7"
end

if ENV["CONTROLLERS"]
    $controllers_count = Integer(ENV["CONTROLLERS"])
end

if ENV["WORKERS"]
    $workers_count = Integer(ENV["WORKERS"])
end

if ENV["IP_PREFIX"]
    $ip_prefix = ENV["IP_PREFIX"]
end

############################################################
# Routines
############################################################

def index_padding(index)
	return "%02d" % index
end

def single_node_name()
    return "single-node"
end

def controller_name(index)
    return "controller" + index_padding(index)
end

def worker_name(index)
    return "worker" + index_padding(index)
end

def single_node_ip()
    return $ip_prefix + ".50"
end

def controller_ip(index)
	return $ip_prefix + ".2" + index_padding(index)
end

def worker_ip(index)
	return $ip_prefix + ".1" + index_padding(index)
end

def add_machine(config, ram, cpus, name, ip)
    config.vm.define name do |machine|
        machine.vm.hostname = name
        machine.vm.network :private_network, ip: ip

        machine.vm.provider :virtualbox do |vb|
            vb.memory = ram
            vb.cpus = cpus
        end
    end
end

def add_hosts_entry(config, name, ip)
    config.vm.provision "shell", inline: "echo '#{ip} #{name}' >> /etc/hosts"
end

############################################################
# Create machines
############################################################

Vagrant.configure("2") do |config|
    config.vm.box = $box
    config.vm.synced_folder '.', '/vagrant', disabled: true
    config.vm.box_check_update = false

    config.vm.provider "virtualbox" do |vb|
        #vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
        vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
    end

    config.vm.provision "shell" do |s|
        s.inline = <<-SHELL
        SHELL
    end

    config.vm.provision "shell", inline: $script

    if $single_node
        add_hosts_entry(config, single_node_name(), single_node_ip())

        add_machine(config, $single_node_ram, $single_node_cpus, single_node_name(), single_node_ip())
    else
        (0..$controllers_count - 1).each do |j|
            add_hosts_entry(config, controller_name(j), controller_ip(j))
        end

        (0..$workers_count - 1).each do |j|
            add_hosts_entry(config, worker_name(j), worker_ip(j))
        end

        (0..$controllers_count - 1).each do |i|
            add_machine(config, $controllers_ram, $controllers_cpus, controller_name(i), controller_ip(i))
        end

        (0..$workers_count - 1).each do |i|
            add_machine(config, $workers_ram, $workers_cpus, worker_name(i), worker_ip(i))
        end
    end
end
