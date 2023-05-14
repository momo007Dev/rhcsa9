### Containers

* Container images adhere to a standard naming convention for identification. This is referred to as fully qualified image name (FQIN). An FQIN is comprised of four components:
  * (1) the storage location (registry_name)
  * (2) the owner or organization name (user_name)
  * (3) a unique repository name (repo_name)
  * (4) an optional version (tag).
* There are several registries available on the Internet. These include Red Hat Container Catalog at ***registry.redhat.io*** (or ***registry.access.redhat.com***), Red Hat Quay at ***quay.io***, and Docker Hub at ***hub.docker.com***.
* RHEL offers two commands—**podman** and **skopeo**— to manage and interact with images, registries, and containers
* The system-wide configuration file for image registries is the registries.conf file and it resides in the /etc/containers directory. 

---

### Podman Command

* Image Management
  * `podman images` => Lists images on local storage
  * `podman inspect` => Display image details
  * `podman (login/logout)` => Login to a container registry
  * `podman pull` => Downloads an image
  * ``podman rmi` => Removes an image
  * `podman search` => Searches for an image
  * `podman tag` => Adds a name to an image
* Container Management
  * `podman attach` => Attaches to a running container
  * `podman exec` => Runs a process in a running container
  * `podman generate` => Generates a systemd unit configuration file
  * `podman info` => Reveals system information
  * `podman ps` => Lists running containers
  * `podman rm` => Removes a container
  * `podman run` => Launches a new container
  * `podman (start/stop/restart)` => Start/stops/restarts a container

---

### Exercices

* **Search, Examine, Download and Remove an Image**

  * In this exercise, you will look for an image called **mysql** in the **quay.io** registry, examine its details, pull it to your system, confirm the retrieval, and finally erase it from the local storage. You will use the podman and skopeo commands as required for these operations.

  * ```bash
    podman search quay.io/rhel8/mysql
    skopeo inspect docker://quay.io/app-sre/mysql
    podman pull docker://quay.io/app-sre/mysql
    podman images
    podman inspect mysql
    podman rmi mysql
    ```

* **Run, Interact with and Remove a Named Container**

  * In this exercise, you will run a container based on the latest version of a universal base image (ubi) for RHEL 8 available in the Red Hat Container Registry. You will assign this container a name and run a few native Linux commands in a terminal window interactively. You will exit out of the container and invoke a command inside the container from server20. Finally, you will reconnect to the container, exit out, and delete it to mark the completion of the exercise.

  * ```bash
    sudo rm -rf /etc/docker
    podman run -ti --name rhel8-base-os ubi8 # Make a container RUNNING
    [root@2d77bduej /]# exit
    podman exec rhel8-base-os cat /etc/redhat-release
    # Red Hat Enterprise Linux release 8.2 (Ootpa)
    podman attach rhel8-base-os # To connect to a RUNNING container
    podman rm rhel8-base-os
    ```

* **Run a Nameless Container and Auto-Remove it after entry point command execution**

  * ```bash
    podman run --rm ubi7 ls
    # Will run the "ls" command and then remove the container
    ```

* **Configure Port Mapping**

  * In this exercise, you will launch a container called **rhel7-port-map in detached mode (as a daemon)** with host port **10000** mapped to port **8000** inside the container. You will use a version of the RHEL 7 image with Apache web server software pre-installed. This image is available in the Red Hat Container Registry. You will list the running container and confirm the port mapping.

  * ```bash
    podman search registry.redhat.io/rhel7/httpd
    podman login registry.redhat.io # username/password prompted
    podman pull registry.redhat.io/rhescl/httpd-24-rhel7
    podman images
    
    podman run -dp 10000:8000 --name rhel7-port-map httpd-24-rhel7
    podman ps
    podman port rhel7-port-map
    ```
  
* **Stop, Restart and Remove a Container**

  * ```bash
    podman (start/stop) rhel7-port-map
    podman ps -a # to view stopped containers too
    
    podman rm rhel7-port-map # Can only remove stopped containers
    ```

* **Pass and Set Environment Variables**

  * In this exercise, you will launch a container using the latest version of a **ubi for RHEL 8 available in the Red Hat Container Registry**. You will inject the **HISTSIZE** environment variable and a variable called **SECRET** with the value **“secret123”**. You will name this container **rhel8-env-vars** and have a shell terminal opened to check the variable settings. Finally, you will remove this container.

  * ```bash
    podman run -it -e HISTSIZE -e SECRET="secret123" --name rhel8-env-vars
    ```

* **Attach Persistent Storage and Access Data Across Containers**

  * In this exercise, you will set up a directory on server20 and attach it to a new container. You will write some data to the directory while in the container. You will delete the container and launch another container with the same directory attached. You will observe that the saved data is available in the new container and is accessible. You will remove the container to mark the completion of this exercise.

  * ```bash
    sudo mkdir /host_data
    sudo chmod 777 /host_data
    ll -d /host_data
    
    sudo podman run --name rhel7-persistent-data -v /host_data:/container_data:Z -it ubi7
    
    # -v = Volume | Z=>SELinux File context
    echo "This is persistent storage" > /container_data/testfile
    ls -lZ /container_data/
    ```

* **Configure a Root Container as a Systemd Service**

  * In this exercise, you will create a systemd unit configuration file for managing the state of your root containers. You will launch a new container and use it as a template to generate a service unit file. You will stop and remove the launched container to avoid conflicts with new containers that will start. You will use the systemctl command to verify the automatic container start, stop, and deletion.

  * ```bash
    sudo podman run -dt --name root-container ubi8
    sudo podman generate systemd --new --name root-container | sudo tee /etc/systemd/system/root-container.service
    ```

  * ```bash
    sudo podman stop root-container
    sudo podman rm root-container
    sudo systemctl daemon-reload
    ```

  * ```bash
    sudo systemctl status root-container
    sudo systemctl restart root-container
    ```

* **Configure a Rootless Container as a systemd Service**

  * In this exercise, you will create a systemd unit configuration file for managing the state of your rootless containers. You will launch a new container as conuser1 (create this user) and use it as a template to generate a service unit file. You will stop and remove the launched container to avoid conflicts with new containers that will start. You will use the systemctl command as conuser1 to verify the automatic container start, stop, and deletion.

  * ```bash
    sudo useradd conuser1
    sudo passwd conuser1
    mkdir ~/.config/systemd/user -p
    
    podman run -dt --name rootless-container ubi8
    podman generate systemd --new --name rootless-container > ~/.config/systemd/user/rootless-containers.service
    
    podman stop rootless-container
    podman rm rootless-containers
    
    systemctl --user daemon-reload
    systemctl --user enable --now rootless-container.service
    loginctl enable-linger # Makes the service stop/start when user logs in
    systemctl --user restart rootless-container
    ```
