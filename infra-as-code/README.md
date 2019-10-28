# Infrastructure as Code

The goal of this lab are to:
- Create a [CentOS 7](https://wiki.centos.org/) VM using [Vagrant](https://www.vagrantup.com/)
- Install [GitLab](https://about.gitlab.com/stages-devops-lifecycle/) on this VM using [Ansible](https://www.ansible.com/)
- Perform health checks on the GitLab installation using Ansible

## Usefull links

- [Vagrant documentation](https://www.vagrantup.com/docs/)
- [Ansible documentation](https://docs.ansible.com/ansible/latest/index.html)
- [Ansible module index](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)
- [GitLab installation doc for CentOS 7](https://about.gitlab.com/install/#centos-7)
- [GitLab Health Check doc](https://docs.gitlab.com/ee/user/admin_area/monitoring/health_check.html)

## Prerequisites

Before you can start the lab, you have to:
1. Install Virtualbox: https://www.virtualbox.org/wiki/Downloads
2. Install Vagrant on your computer: https://www.vagrantup.com/downloads.html
3. (Optional) **On Windows**, ensure that Hyper-V is disabled:
   1. Open a new Powershell
   2. Run the following command:
      ```powershell
      Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      ```
4. Download the `centos/7` Vagrant box for the **Virtualbox provider**:
   ```shell
   $ vagrant box add centos/7
   ==> box: Loading metadata for box 'centos/7'
       box: URL: https://vagrantcloud.com/centos/7
   This box can work with multiple providers! The providers that it
   can work with are listed below. Please review the list and   choose
   the provider you will be working with.
   
   1) hyperv
   2) libvirt
   3) virtualbox
   4) vmware_desktop
   
   Enter your choice: 3
   ```
5. Clone the lab repository to your computer:
   ```
   git clone https://github.com/adaltas/ece-devops.git
   ```
6. Go to the `infra-as-code` directory:
   ```
   cd ece-devops/infra-as-code
   ```

## Lab

### Useful information

In the lab, we will use the `ansible_local` provisioner, so Ansible will be automatically installed on the VM by Vagrant. You don't need it on your computer!

### Resources

In the `infra-as-code` directory, you will find:
- A `Vagrantfile` that defines the VMs to be managed by Vagrant (1 CentOS 7 VM named `gitlab_server` in our case)
- A `playbooks/` directory that contains Ansible playbooks to install GitLab and run health checks

### Lab steps

1. Take a look at the [`Vagrantfile`](Vagrantfile) and at the YAML files [`playbooks/run.yml`](playbooks/run.yml) and [`playbooks/gitlab/install/tasks/main.yml`](playbooks/roles/gitlab/install/tasks/main.yml)
2. Create and provision the VM
   1. Run the `vagrant up` command
   2. You should end up with the following error (this is planned): 
      ```shell
      TASK [gitlab/install : Install GitLab] *****************************************
      fatal: [gitlab_server]: FAILED! => {"changed": false, "msg": "No package matching 'gitlab-ee' found available, installed or updated", "rc": 126, "results": ["No package matching 'gitlab-ee' found available, installed or updated"]}
      ```
   3. Check that everything is ok by connecting to the VM through SSH:
      ```
      vagrant ssh gitlab_server
      ```
3. Complete the GitLab installation
   1. Fill the blanks in [`gitlab/install/tasks/main.yml`](playbooks/roles/gitlab/install/tasks/main.yml) based on the steps 1. and 2. of the [GitLab installation doc for CentOS 7](https://about.gitlab.com/install/#centos-7)
   2. Update the playbooks on the VM using `vagrant upload`:
      ```
      vagrant upload playbooks /vagrant/playbooks gitlab_server
      ```
   3. Rerun provisioning with the command `vagrant provision`
4. Test your installation by connecting to http://20.20.20.2 (step 3 of the [GitLab installation doc](https://about.gitlab.com/install/#centos-7)):
   1. Choose a password
   2. Login as the user `root` using the password

### Homework

1. Read the [GitLab Health Check doc](https://docs.gitlab.com/ee/user/admin_area/monitoring/health_check.html)
2. Run a healthcheck using `curl`:
   1. Connect to the VM using `vagrant ssh`
   2. Run the command:
      ```shell
      $ curl http://127.0.0.1/-/health
      GitLab OK
      ```
3. Read [`playbooks/roles/gitlab/healthcheck/tasks/main.yml`](playbooks/roles/gitlab/healthcheck/tasks/main.yml) to understand how it relates
4. Run the `gitlab/healthcheck` role
   1. Connect to the VM using `vagrant ssh`
   2. Run the playbooks using the right tag (replace `TAG`):
      ```
      ansible-playbook /vagrant/playbooks/run.yml --tags TAG -i /tmp/vagrant-ansible/inventory/vagrant_ansible_local_inventory
      ```
5. Run the 2 other kind of health checks in the playbook (using the [uri module](https://docs.ansible.com/ansible/latest/modules/uri_module.html)):
   1. [Readiness check](https://docs.gitlab.com/ee/user/admin_area/monitoring/health_check.html#readiness).
   2. [Liveness check](https://docs.gitlab.com/ee/user/admin_area/monitoring/health_check.html#liveness)
6. Print the results of the health checks in the console
7. (Bonus) Print a custom message with only the disfunctional(s) services in the Readiness check if there are some. To test the printing, stop `redis` using the command `sudo gitlab-ctl stop redis` on the node before running the playbook again. Tip: use the `json` attribute of the response.