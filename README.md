# Provisioning a container image using Ansible.

[This repo contains files for this blog post.](https://blog.tomecek.net/post/building-containers-with-buildah-and-ansible/)

The `./provision-container.yml` Ansible playbook is able to create a container
according to a specified base image and execute an Ansible role in it, thus
provisioning it.


## Requirements

A recent version of [Ansible](https://github.com/ansible/ansible), and a container execution engine: either
[buildah](https://github.com/ansible/ansible-container/pull/790) or [docker](https://github.com/moby/moby).


## Usage

By default, the playbook uses buildah. If you want to use docker, just change
the `container_engine` variable:
```
container_engine: docker
```

Then just simply run the playbook as root (buildah requires root):

```console
$ sudo ansible-playbook provision-container.yml

PLAY [localhost] ***********************************************************************

TASK [Gathering Facts] *****************************************************************
ok: [localhost]

TASK [Obtain base image and create a container out of it] ******************************
changed: [localhost]

TASK [Make the base image available locally] *******************************************
skipping: [localhost]

TASK [Create the container] ************************************************************
skipping: [localhost]

TASK [add the newly created container to the inventory] ********************************
changed: [localhost]

TASK [run the role in the container] ***************************************************

TASK [sample-nginx : install nginx] ****************************************************
changed: [localhost -> build_container]

TASK [sample-nginx : clean dnf metadata] ***********************************************
 [WARNING]: Consider using dnf module rather than running dnf

changed: [localhost -> build_container]

TASK [Change default command of the container image] ***********************************
changed: [localhost]

TASK [Commit the container and make it an image] ***************************************
changed: [localhost]

TASK [Commit the container and make it an image] ***************************************
skipping: [localhost]

TASK [remove the container] ************************************************************
skipping: [localhost]

PLAY RECAP *****************************************************************************
localhost                  : ok=7    changed=6    unreachable=0    failed=0
```

By default, the container image is exported into dockerd:
```console
$ docker images docker.io/nginx
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/nginx     latest              8f1aaab79770        25 seconds ago      268 MB
```

You can just run it and check if nginx serves the default page:
```console
$ docker run -d docker.io/nginx
3165ec03253bae24951d20ab7a4a3905f824b67304eca16ae0ce9ca01504c411

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                  PORTS               NAMES
3165ec03253b        docker.io/nginx     "nginx -g 'daemon ..."   2 seconds ago       Up Less than a second                       kind_mayer

$ curl -s 172.17.0.2 | grep title
        <title>Test Page for the Nginx HTTP Server on Fedora</title>
```
