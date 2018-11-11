## ansible-molecule-demo

### Introduction

[molecule](https://github.com/ansible/molecule) is a project for testing Ansible roles.  It was announced in [September 2018](https://groups.google.com/forum/#!topic/ansible-project/ehrb6AEptzA) that molecule and [ansible-lint](https://github.com/ansible/ansible-lint) that upstream maintenance would be adopted into the ansible project.

Myself and colleagues had looked at molecule in the past, especially after watching the [Infrastructure Testing with Molecule](https://www.ansible.com/infrastructure-testing-with-molecule) session at AnsibleFest 2017 by Elan Hashman shortly after the San Francisco event.  However, after some initialy testing I never really introduced it as a standard part of my Ansible role development process.  So now molecule has kind of got a more official stamp of approval this repo is aimed at capturing getting started (again) with the current versions.

Contents

- [Platform](#platform)
- [Docker](#docker)
- [Environment](#environment)
- [molecule init](#molecule-init)
- [molecule test](#molecule-test)

### Platform  
For testing I will be using CentOS 7.5 patched to date (`sudo yum -y update`).  In my case this is running as a VM in VMware Workstation Pro 12.5 on my work laptop (not that should have any baring on what is described in this repo).

### Docker  
Initially I will be working with the [Docker driver](https://molecule.readthedocs.io/en/latest/configuration.html#docker).  So the first steps are to get [Docker CE](https://www.docker.com/products/docker-engine) installed on my CentOS Linux installation and running at startup:  

```
sudo yum -y install yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum -y install docker-ce
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker tony
```

The last line adds my user account to the docker group so after I log back in again I can use docker commands without needing to use sudo.  

To test all is ok I can logoff/logon using my account and test all is ok by running:  

<pre>
<b>docker run hello-world</b>
</pre>
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
d1725b59e92d: Pull complete
Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```
### Environment

I will be running ansible and molecule in a Python virtual environment using [virtualenv](https://virtualenv.pypa.io/en/latest/).  This is what I used to get my environment configured ready to play with molecule:  
```
sudo yum install -y epel-release
sudo yum install -y gcc python-pip python-devel openssl-devel libselinux-python tree git
sudo pip install --upgrade pip
sudo pip install virtualenv
mkdir ~/venv
mkdir ~/ansible
virtualenv ~/venv/ansible-molecule-demo
source ~/venv/ansible-molecule-demo/bin/activate
pip install ansible
pip install molecule
pip install docker-py
```

### molecule init

So now we have created a virtualenv for using molecule we can create a new ansible role to test with.  The commands below will activate the virtualenv and create a new molecule enabled ansible role template:  

```
source ~/venv/ansible-molecule-demo/bin/activate
cd ~/ansible/
molecule init role --role-name ansible-molecule-demo
cd ansible-molecule-demo
```

We can view the created structure using the `tree` command:  
```
.
├── defaults
│   └── main.yml
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── molecule
│   └── default
│       ├── Dockerfile.j2
│       ├── INSTALL.rst
│       ├── molecule.yml
│       ├── playbook.yml
│       └── tests
│           ├── test_default.py
│           └── test_default.pyc
├── README.md
├── tasks
│   └── main.yml
└── vars
    └── main.yml
```

As you can see the structure looks like you might expect for a typical ansible role with the addition of the molecule directory.  

*Note*: The contents of this repo have this structure at the repo root i.e. the repo is a molecule enabled ansible role  

### molecule test

To perform a test of our new blank skeleton role we can run `molecule test`.  This should pass through all of the test sequences without error (bearing in mind the role is not actually doing anything yet).  

*Note*: When first running this command in my test environment I got an error on the task `TASK [Create Dockerfiles from image names]`.  If you get the same you can result then review the [Create Dockerfiles from image names](https://github.com/tonyskidmore/ansible-molecule-demo/wiki/Create-Dockerfiles-from-image-names) section from the Wiki of this repo.  

*Note*: After fixing the above issue I also encountered a 'Config' object has no attribute 'cache' error during the verifier stage.  If you encounter the same issue refer to ['Config' object has no attribute 'cache'](https://github.com/tonyskidmore/ansible-molecule-demo/wiki/'Config'-object-has-no-attribute-'cache') also from the Wiki of this repo.

### molecule configuration  

When initializing a molecule enabled repo with `molecule init` a default configuration is created in the `molecule/default/molecule.yml` file.  Reading the blog article by Jeff Geerling [Testing your Ansible roles with Molecule](https://www.jeffgeerling.com/blog/2018/testing-your-ansible-roles-molecule) you will discover that you can use pre-built Docker images and he provides a set of these images for various Linux distributions.  If you review the [molecule.yml](https://github.com/tonyskidmore/ansible-molecule-demo/blob/master/molecule/default/molecule.yml) file in this repo you will see that we have taken this approach and also commented out some of the sequences that are skipped by default.  






### References
[molecule](https://github.com/ansible/molecule)  

[molecule documentation](https://readthedocs.org/projects/molecule/)  

[Testing your Ansible roles with Molecule](https://www.jeffgeerling.com/blog/2018/testing-your-ansible-roles-molecule)  


### Versions
CentOS 7.5  
Python 2.7.5  
ansible 2.7.1  
molecule  2.19
