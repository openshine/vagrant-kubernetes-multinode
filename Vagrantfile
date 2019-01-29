VAGRANTFILE_API_VERSION = "2"


# Define constants
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Use vagrant-env plugin if available
  if Vagrant.has_plugin?("vagrant-env")
    config.env.load('.env') # enable the plugin
  end

  # ENV variables defaults
  K8S_IMAGE_BOX = ENV.fetch('K8S_IMAGE_BOX', "bento/fedora-28").to_s
  K8S_MASTERS_CPUS = ENV.fetch('K8S_MASTERS_CPUS', 2).to_i
  K8S_MASTERS_MEM = ENV.fetch('K8S_MASTERS_MEM', 2048).to_i
  K8S_WORKERS = ENV.fetch('K8S_WORKERS', 3).to_i
  K8S_WORKERS_CPUS = ENV.fetch('K8S_WORKERS_CPUS',1).to_i
  K8S_WORKERS_MEM = ENV.fetch('K8S_WORKERS_MEM', 1024).to_i


  config.vm.define "master", primary: true do |master|
    master.vm.box = K8S_IMAGE_BOX
    master.vm.hostname = "master.vagrant.vm"
    if Vagrant.has_plugin?("vagrant-hosts")
      master.vm.provision :hosts, :sync_hosts => true
    end
    master.vm.network "private_network", ip: "192.168.99.100"

    master.vm.provider :virtualbox do |v|
      v.memory = K8S_MASTERS_MEM
      v.cpus = K8S_MASTERS_CPUS
    end
  end

  (0..K8S_WORKERS-1).each do |i|
    config.vm.define "worker#{i}" do |worker|
      worker.vm.box = K8S_IMAGE_BOX
      worker.vm.hostname = "worker#{i}.vagrant.vm"
      if Vagrant.has_plugin?("vagrant-hosts")
        worker.vm.provision :hosts, :sync_hosts => true
      end
      ip = 109 + i
      worker.vm.network "private_network", ip: "192.168.99.#{ip}"

      worker.vm.provider :virtualbox do |v|
        v.memory = K8S_WORKERS_MEM
        v.cpus = K8S_WORKERS_CPUS

        gluster_disk = "worker#{i}.vdi"
        if not File.exists?(gluster_disk)
          v.customize ['createhd', '--filename', gluster_disk, '--size', 25 * 1024]
        end
        v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', gluster_disk]
      end
    end
  end

  config.vm.define "controller", primary: true do |controller|
    controller.vm.box = "bento/fedora-28"
    controller.vm.hostname = "controller.vagrant.vm"
    controller.vm.network "private_network", ip: "192.168.99.99"
    controller.vm.provision :hosts, :sync_hosts => true

    controller.vm.provision "ansible_local" do |ansible|
      ansible.config_file = "ansible/ansible.cfg"
      ansible.playbook = "ansible/main.yml"
      ansible.limit = "all"
      ansible.galaxy_role_file = "ansible/requirements.yml"
      ansible.galaxy_roles_path = "/home/vagrant/.ansible/roles"
      ansible.extra_vars = {
        ansible_ssh_user: 'vagrant',
        ansible_ssh_pass: 'vagrant',
        ansible_python_interpreter: '/usr/bin/python3'
      }
      ansible.groups = {
        "k8s-masters" => ["master"],
        "k8s-masters:vars" => {
          kubernetes_role: "master",
          kubernetes_apiserver_advertise_address: "192.168.99.100",
          kubernetes_allow_pods_on_master: false,
          kubernetes_enable_web_ui: false,
          kubernetes_pod_network_cidr: "10.244.0.0/17",
          kubernetes_kubeadm_init_extra_opts: "--service-cidr=10.244.128.0/17",
          kubernetes_kubelet_extra_args: "--cluster-dns=10.244.128.10 --cluster-domain=cluster.local",
          kubernetes_flannel_manifest_file_rbac: "/vagrant/k8s/flannel/kube-flannel-rbac.yml",
          kubernetes_flannel_manifest_file: "/vagrant/k8s/flannel/kube-flannel.yml"
        },
        "k8s-workers" => ["worker[0:#{K8S_WORKERS-1}]"],
        "k8s-workers:vars" => {
          kubernetes_role: "node",
          kubernetes_allow_pods_on_master: false,
          kubernetes_kubelet_extra_args: "--cluster-dns=10.244.128.10 --cluster-domain=cluster.local"
        }
      }
      ansible.verbose = true
    end
  end
end
