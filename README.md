# Build a Docker Image and Push the Image to Dockerhub

An Ansible playbook which builds a docker image from a DockerFile fetched from GitHub and there after ansible pushes the image to DockerHub. Simply its an easy method to build a docker image and push it to the repository.

## Architecture

![image](https://user-images.githubusercontent.com/94472101/149037381-ca31ba23-55af-4c8f-8ebd-d5d21f069ca6.png)


## Synopsis

 - Install the required packages such as docker, git, python2-pip.
 - Clone the git repository to the client machine (build server).
 - Login to the DockerHub account, build the image and push the built image.
 - Finally remove the docker images from the build server.

## Features

 - Easy to build a docker image from the playbook and push it to DockerHub.
 - The play is using inbuilt ansible module
 - Image will be only there is a change in the git repo contents

## Prerequisites
 
 - [Install Ansible](https://devopsmyway.com/how-to-install-ansible-on-amazon-linuxec2/) on your master machine
 - [DockerHub account](https://hub.docker.com/)
 
## Packages Required

 - [docker](https://docs.docker.com/engine/install/)
 - [git](https://git-scm.com/downloads)
 - [python2-pip](https://pip.pypa.io/en/stable/installation/)
 - [docker-py](https://pypi.org/project/docker-py/)

## Modules Used
 - [yum[(())
 - [git](https://docs.ansible.com/ansible/2.5/modules/git_module.html)
 - [service](https://docs.ansible.com/ansible/2.5/modules/service_module.html)
 - [pip](https://docs.ansible.com/ansible/2.4/pip_module.html)
 - [docker_login](https://docs.ansible.com/ansible/2.9/modules/docker_login_module.html)
 - [docker_image](https://docs.ansible.com/ansible/2.4/docker_image_module.html)

## Variables Used

 - git_url                                           ### Enter GitHub repo url ###
 - docker_user                                       ### Enter dockerhub account username ###
 - docker_password                                   ### Enter dockerhub account password ###
 - Image_name                                        ### Enter Image name ###
 

## Provisioning

- Install the packages required to run the playbook and start/enable the docker service in the client machine.

```
    - name: "Build: installing Packages required for buiding Docker Image"
      yum:
        name:
          - docker
          - python2-pip
          - git
        state: present
     
    - name: "Build: Starting/Enabling Docker service"
      service:
        name: docker
        state: restarted
        enabled: true
        
    - name: "Build: Installing Docker client for python"
      pip:
        name: docker-py
        state: present

        
```

- Now clone the git repo of FlaskApplication in the client machine.

```
    - name: "Build: Clonning the git repo of flaskapp"
      git:
        repo: "{{ git_url }}"
        dest: "/var/flaskapp/"
      register: git_status

```

- Login to the DockerHub account

```

    - name: "Build: Login to Dockerhub"
      when: git_status.changed == true
      docker_login:
        username: "{{ docker_user }}"
        password: "{{ docker_password }}"
        
```
- Build the Docker image from the DockerFile which is clonned from the git repo.

```
    - name: "Build: Building a dockerimage"
      when: git_status.changed == true
      docker_image:
        build:
          path: "/var/flaskapp/"
        name: "{{ Image_name }}"
        tag: "{{ item }}"
        state: present
        push: yes
        source: build
      with_items:    
        - "latest"
        - "{{ git_status.after }}"

```

- Also define the variables 

```
  vars:
    git_url: "https://github.com/Fujikomalan/devops-flask.git"
    docker_user: "swathikarun"
    docker_password: "Sw@th!k@4"
    Image_name: "swathikarun/devops-flaskapp"
```

- Now run the playbook using the below command.

```
ansible-playbook -i inventory dockerimage-build.yml
```

## Result

A docker image of the Flask Application with a unique tag is built and finally pushed the image to DockerHub.
