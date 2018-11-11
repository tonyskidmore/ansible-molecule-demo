## ansible-molecule-demo

[![Build Status](https://travis-ci.org/tonyskidmore/ansible-molecule-demo.svg?branch=master)](https://travis-ci.org/tonyskidmore/ansible-molecule-demo)

### Introduction

[molecule](https://github.com/ansible/molecule) is a project for testing Ansible roles.  It was announced in [September 2018](https://groups.google.com/forum/#!topic/ansible-project/ehrb6AEptzA) that molecule and [ansible-lint](https://github.com/ansible/ansible-lint) that upstream maintenance would be adopted into the ansible project.

Myself and colleagues had looked at molecule in the past, especially after watching the [Infrastructure Testing with Molecule](https://www.ansible.com/infrastructure-testing-with-molecule) session at AnsibleFest 2017 by Elan Hashman shortly after the San Francisco event.  However, after some initialy testing I never really introduced it as a standard part of my Ansible role development process.  So now molecule has kind of got a more official stamp of approval this repo is aimed at capturing getting started (again) with the current versions.

Contents

- [Platform](#platform)
- [Docker](#docker)
- [Environment](#environment)
- [molecule init](#molecule-init)
- [molecule test](#molecule-test)
- [Basic Update](#basic-update)
- [Travis CI](#travis-ci)

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

### Basic Update  

Ok so we have a blank role but is doesn't do anything.  So we are going to make a very basic update with a TDD approach to add some functionality to our demo role.  If you want to follow along you can try following the steps below after reviewing all of the above sections to make sure you have a similar environment and are aware of some of the issues you might hit:  

```
git clone https://github.com/tonyskidmore/ansible-molecule-demo.git
cd ansible-molecule-demo/
git checkout test
```

We now have a version of ansible-molecule-demo without the updates to the role.  So the first thing we will do is add the test for our role requirements.  We want a file called `/tmp/test` to exist with the content of `ansible-molecule-demo`.  Update the `molecule/default/tests/test_default.py` to add an additional test to test our required functionality:  

```
import os

import testinfra.utils.ansible_runner

testinfra_hosts = testinfra.utils.ansible_runner.AnsibleRunner(
    os.environ['MOLECULE_INVENTORY_FILE']).get_hosts('all')


def test_hosts_file(host):
    f = host.file('/etc/hosts')

    assert f.exists
    assert f.user == 'root'
    assert f.group == 'root'


def test_test_file(host):
    f = host.file('/tmp/test')

    assert f.exists

    content = f.content
    assert b'ansible-molecule-demo' in content
```

Now if we run `molecule test` now we will get the following error during the verify task because the file does not exist:  
```
--> Scenario: 'default'
--> Action: 'verify'
--> Executing Testinfra tests found in /home/tony/ansible/ansible-molecule-demo/molecule/default/tests/...
    ============================= test session starts ==============================
    platform linux2 -- Python 2.7.5, pytest-3.10.0, py-1.7.0, pluggy-0.8.0
    rootdir: /home/tony/ansible/ansible-molecule-demo/molecule/default, inifile: pytest.ini
    plugins: testinfra-1.16.0
collected 2 items

    tests/test_default.py .F                                                 [100%]

    =================================== FAILURES ===================================
    ______________________ test_test_file[ansible://instance] ______________________

    host = <testinfra.host.Host object at 0x7f604ea22c90>

        def test_test_file(host):
            f = host.file('/tmp/test')

    >       assert f.exists
    E       assert False
    E        +  where False = <file /tmp/test>.exists

    tests/test_default.py:20: AssertionError
    ===================== 1 failed, 1 passed in 20.06 seconds ======================
An error occurred during the test sequence action: 'verify'. Cleaning up.
```

We will update `defaults/main.yml` and `tasks/main.yml` with the below content:  
```
---
# defaults file for ansible-molecule-demo
test_file: /tmp/test
test_file_content: "ansible-molecule-demo"
```

```
---
# tasks file for ansible-molecule-demo

- name: create test file with contents
  copy:
    path: "{{ test_file }}"
    content: "{{ test_file_content }}"
```

Now if we run `molecule test` we should have made the test pass because of the functionality we added to our role:  

```
--> Scenario: 'default'
--> Action: 'verify'
--> Executing Testinfra tests found in /home/tony/ansible/ansible-molecule-demo/molecule/default/tests/...
    ============================= test session starts ==============================
    platform linux2 -- Python 2.7.5, pytest-3.10.0, py-1.7.0, pluggy-0.8.0
    rootdir: /home/tony/ansible/ansible-molecule-demo/molecule/default, inifile: pytest.ini
    plugins: testinfra-1.16.0
collected 2 items

    tests/test_default.py ..                                                 [100%]

    ========================== 2 passed in 21.75 seconds ===========================
Verifier completed successfully.
```

### Travis CI  

[Travis CI](https://docs.travis-ci.com/) is a cloud based Continuous Integration product that allows for automated testing.  As detailed in the Jeff Geerling blog post [Testing your Ansible roles with Molecule](https://www.jeffgeerling.com/blog/2018/testing-your-ansible-roles-molecule) we can link Travis CI to our GitHub repo and get Travis CI to do test builds of our code on different platforms based on commits and/or on a schedule.  

To enable this functionality we create the [.travis.yml](https://github.com/tonyskidmore/ansible-molecule-demo/blob/master/.travis.yml) file and enable the repo in our Travis account.  


### References
[molecule](https://github.com/ansible/molecule)  

[molecule documentation](https://readthedocs.org/projects/molecule/)  

[Testing your Ansible roles with Molecule](https://www.jeffgeerling.com/blog/2018/testing-your-ansible-roles-molecule)  


### Versions
CentOS 7.5  
Python 2.7.5  
ansible 2.7.1  
molecule  2.19
