Asinble

coppy phaỉ cài đặt thêm gói libselinux-python
	http://mirror.centos.org/centos/6/os/x86_64/Packages/libselinux-python-2.0.94-7.el6.x86_64.rpm
	http://118.69.190.70/packages/libselinux-python/libselinux-python-2.0.94-7.el6.x86_64.rpm

Cài đặt ansinble trên server deploy
	sudo yum install ansible

Thêm ip các remote host vào file /etc/ansible/hosts, ansible sẽ dùng list các host trong file này để chạy

cài đặt gói "libselinux-python" trên các remote host
	 ansible all -m yum -a 'state=present name=libselinux-python' --ask-pass

add public key xuống remote host, chuyển sang sử dụng public key thay vì dùng uer password
	tạo mới key
	ssh-keygen -t rsa
	coppy key xuống các remote host
	ansible all -m copy -a "src=/root/.ssh/id_rsa.pub dest=/tmp/id_rsa.pud" --ask-pass -c paramiko
	add key vào các remote host
	ansible all -m shell -a "cat /tmp/id_rsa.pud >> /root/.ssh/authorized_keys" --ask-pass -c paramik
	ansible all -m authorized_key -a "user=root key='{{ lookup('file', '/root/.ssh/id_rsa.pub') }}' path=/root/.
ssh/authorized_keys manage_dir=no" --ask-pass -c paramiko

-a 'ARGUMENTS', --args='ARGUMENTS'
           The ARGUMENTS to pass to the module.
	
-m NAME, --module-name=NAME
           Execute the module called NAME.

-u USERNAME, --user=USERNAME
           Use this USERNAME to login to the target host, instead of the current user.

-U SUDO_USERNAME, --sudo-user=SUDO_USERNAME
           Sudo to SUDO_USERNAME default is root. (deprecated, use become).

-k, --ask-pass
           Prompt for the connection password, if it is needed for the transport used. For example,
           using ssh and not having a key-based authentication with ssh-agent.

ping đến các host
	ansible all -m ping --ask-pass

Run bash shell
	ansible all -m shell -a "hostname" -u root

copy file đến các remote host
	ansible all -m copy -a "src=/root/.ssh/id_rsa.pub dest=/tmp/id_rsa.pud" --ask-pass -c paramiko
	

Install YUM package
	ansible all -m yum -a "name=httpd state=present" -u root
		present: install
		latest:	last version
		removed: remove
		installes: install
		absent: remove

authorized_key
	 ansible all -m authorized_key -a "user=root key='{{ lookup('file', '/root/.ssh/id_rsa.pub') }}' path=/root/.
ssh/authorized_keys manage_dir=no" --ask-pass -c paramiko

start service
	ansible HNIMPOP -m service -a "name=iptables state=running" -u root --ask-pass
		started: start service
		stopped: stop service
		restarted: restart service
		reloaded: reload service

auto 1 kịch bản

test.yml
---

- hosts: ElasticGroup
  remote_user: root
  tasks:

    - name: Install nginx
      yum: pkg=nginx state=installed
    - name: Copy ansible inventory file to client
      copy: src=/etc/ansible/hosts dest=/root/
              owner=root group=root mode=0644


thực thi:
ansible-playbook test.yml -f 10



ansible TEST -m shell -a "cd ~ && curl -O http://118.69.190.70/packages/libselinux-python/libselinux-python-2.0.94-7.el6.x86_64.rpm" -u root

ansible TEST -m shell -a "cd ~ && rpm -ivh libselinux-python-2.0.94-7.el6.x86_64.rpm && rm -rf libselinux-python-2.0.94-7.el6.x86_64.rpm" -u root --ask-pass

ssh-keygen -t rsa

ansible TEST -m copy -a "src=/home/thuonghh/.ssh/id_rsa.pub dest=/tmp/id_rsa.pud" -u root --ask-pass -c paramiko


ansible TEST -m shell -a "cd ~ && mkdir .ssh" -u root --ask-pass -c paramiko

ansible TEST -m shell -a "cd ~ && mkdir .ssh && cat /tmp/id_rsa.pub >> /root/.ssh/authorized_keys" -u root --ask-pass -c paramiko

ansible TEST -m authorized_key -a "user=root key='{{ lookup('file', '/home/thuonghh/.ssh/id_rsa.pub') }}' path=/root/.ssh/authorized_keys manage_dir=no" -u root --ask-pass -c paramiko


deploy-ansible.yml
---

- hosts: ['172.28.0.86']
  remote_user: root
  tasks:

    - name: download libselinux-python package
      shell: cd ~ && curl -O http://118.69.190.70/libselinux-python/libselinux-python-2.0.94-7.el6.x86_64.rpm 
      ignore_errors: yes

    - name: install libselinux-python package
      shell: cd ~ && rpm -ivh libselinux-python-2.0.94-7.el6.x86_64.rpm && rm -rf libselinux-python-2.0.94-7.el6.x86_64.rpm
      ignore_errors: yes

    - name: clear libselinux-python file
      file:
        path: /root/.ssh
        state: absent

    - name: copy publickey to remote host
      copy: src=/home/thuonghh/.ssh/id_rsa.pub dest=/tmp/id_rsa.pud

    - name: create .ssh directory if it doesn't exist
      file:
        path: /root/.ssh
        state: directory
        mode: 0755

    - name: create authorized_keys
      shell: cd ~ && cat /tmp/id_rsa.pud >> /root/.ssh/authorized_keys
      ignore_errors: yes

    - name: Set authorized key in alternate location
      authorized_key:
        user: root
        state: present
        key: "{{ lookup('file', '/home/thuonghh/.ssh/id_rsa.pub') }}"
        path: /root/.ssh/authorized_keys
        manage_dir: no

rsyslog.yml
---
- hosts: ['172.28.0.86']
  remote_user: root
  tasks:

    - name: restart rsyslog services
      service: 
        name: rsyslog
        state: restarted
