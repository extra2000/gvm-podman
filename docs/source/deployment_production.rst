Production Deployment
=====================

Example how to deploy GVM for single-instance production environment.

Prerequisites
-------------

* Redis container from https://github.com/extra2000/redis-podman with at least 500MB memory limit.

Clone repositories
------------------

.. code-block:: bash

    mkdir ~/extra2000
    cd ~/extra2000
    git clone https://github.com/extra2000/gvm-podman.git
    git clone https://github.com/extra2000/greenbone-gvm-libs.git -b v21.4.4 gvm-podman/src/gvm-libs
    git clone https://github.com/extra2000/greenbone-pg-gvm.git gvm-podman/src/pg-gvm
    git clone https://github.com/extra2000/greenbone-gvmd.git -b v21.4.4 gvm-podman/src/gvmd
    git clone https://github.com/extra2000/greenbone-openvas-scanner.git -b 21.4.4-rootless-podman-fix gvm-podman/src/openvas-scanner
    git clone https://github.com/extra2000/greenbone-ospd-openvas.git -b v21.4.4 gvm-podman/src/ospd-openvas
    git clone https://github.com/extra2000/greenbone-gsad.git -b v21.4.4 gvm-podman/src/gsad
    git clone https://github.com/extra2000/greenbone-gsa.git -b v21.4.4 gvm-podman/src/gsa
    git clone https://github.com/extra2000/greenbone-gvm-tools.git -b v21.10.0 gvm-podman/src/gvm-tools

.. note::

    The ``gvm-tools`` repository is optional, only used for remote access or through command-line.

Then, ``cd`` into project root directory:

.. code-block:: bash

    cd gvm-podman/

Build images
------------

From the project root directory, ``cd`` into ``src/``:

.. code-block:: bash

    cd src/

Build images:

.. code-block:: bash

    podman build -t extra2000/greenbone/gse-base -f Dockerfile.gse-base .
    podman build -t extra2000/greenbone/gvmd -f Dockerfile.gvmd .
    podman build -t extra2000/greenbone/pg-gvm -f Dockerfile.pg-gvm .
    podman build -t extra2000/greenbone/openvas-scanner -f Dockerfile.openvas-scanner .
    podman build -t extra2000/greenbone/ospd-openvas -f Dockerfile.ospd-openvas .
    podman build -t extra2000/greenbone/gsad -f Dockerfile.gsad .
    podman build -t extra2000/greenbone/gsa -f Dockerfile.gsa .
    podman build -t extra2000/greenbone/gvm-tools -f Dockerfile.gvm-tools .

Deploy pg-gvm
-------------

From the project root directory, ``cd`` into ``deployment/production/pg-gvm``:

.. code-block:: bash

    cd deployment/production/pg-gvm

Create config files and fix permissions:

.. code-block:: bash

    cp -v configmaps/pg-gvm.yaml{.example,}
    cp -v configs/postgresql.conf{.example,}
    cp -v configs/pg_hba.conf{.example,}
    chmod og+r ./configs/postgresql.conf ./configs/pg_hba.conf
    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v pg-gvm-pod.yaml{.example,}

Create SELinux security policy:

.. code-block:: bash

    cp -v selinux/pg_gvm_podman.cil{.example,}

Load SELinux security policy:

.. code-block:: bash

    sudo semodule -i selinux/pg_gvm_podman.cil /usr/share/udica/templates/base_container.cil

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "pg_gvm_podman"

Deploy pg-gvm:

.. code-block:: bash

    podman play kube --configmap ./configmaps/pg-gvm.yaml --seccomp-profile-root ./seccomp/ pg-gvm-pod.yaml

Create systemd files to run at startup:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name pg-gvm-pod-srv01
    systemctl --user enable container-pg-gvm-pod-srv01.service

Deploy gvmd
-----------

From the project root directory, ``cd`` into ``deployment/production/gvmd``:

.. code-block:: bash

    cd deployment/production/gvmd

Create config file:

.. code-block:: bash

    cp -v configmaps/gvmd.yaml{.example,}

Create pod file:

.. code-block:: bash

    cp -v gvmd-pod.yaml{.example,}

Create SELinux security policy:

.. code-block:: bash

    cp -v selinux/gvmd_podman.cil{.example,}

Load SELinux security policy:

.. code-block:: bash

    sudo semodule -i selinux/gvmd_podman.cil /usr/share/udica/templates/base_container.cil

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "gvmd_podman"

Setup database and ``data-objects`` directories:

.. code-block:: bash

    podman run -it --rm -v pg-gvm-postgresql-run:/var/run/postgresql:ro -v gvm-var-lib:/var/lib/gvm/:rw localhost/extra2000/greenbone/gvmd bash
    gvmd --verbose --migrate
    gvmd --verbose --create-user=admin --password=admin
    gvmd --verbose --get-users
    gvmd --modify-setting 78eceaec-3385-11ea-b237-28d24461215b --value <admin_uuid>
    mkdir -pv /var/lib/gvm/data-objects/gvmd/21.04/{configs,port_lists,report_formats}
    exit

Sync NVT:

.. code-block:: bash

    podman run -it --rm -v openvas-var-lib:/var/lib/openvas:rw --user ospd-openvas localhost/extra2000/greenbone/ospd-openvas bash
    greenbone-nvt-sync
    exit

Sync feeds:

.. code-block:: bash

    podman run -it --rm -v pg-gvm-postgresql-run:/var/run/postgresql:ro -v openvas-var-lib:/var/lib/openvas:ro -v ./secrets/server.crt:/var/lib/gvm/CA/servercert.pem:ro -v ./secrets/server.key:/var/lib/gvm/private/CA/serverkey.pem:ro -v ./secrets/ca.crt:/var/lib/gvm/CA/cacert.pem:ro -v gvm-var-lib:/var/lib/gvm/:rw localhost/extra2000/greenbone/gvmd bash
    greenbone-feed-sync --type SCAP
    greenbone-feed-sync --type CERT
    greenbone-feed-sync --type GVMD_DATA
    exit

Deploy gvmd:

.. code-block:: bash

    podman play kube --configmap ./configmaps/gvmd.yaml --seccomp-profile-root ./seccomp/ gvmd-pod.yaml

Create systemd files to run at startup:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name gvmd-pod-srv01
    systemctl --user enable container-gvmd-pod-srv01.service

Deploy ospd-openvas
-------------------

From the project root directory, ``cd`` into ``deployment/production/ospd-openvas``:

.. code-block:: bash

    cd deployment/production/ospd-openvas

Create config files and fix permissions:

.. code-block:: bash

    cp -v configmaps/ospd-openvas.yaml{.example,}
    cp -v configs/ospd-openvas.conf{.example,}
    chcon -v -t container_file_t configs/ospd-openvas.conf

Create pod file:

.. code-block:: bash

    cp -v ospd-openvas-pod.yaml{.example,}

Create SELinux security policy:

.. code-block:: bash

    cp -v selinux/ospd_openvas_podman.cil{.example,}

Load SELinux security policy:

.. code-block:: bash

    sudo semodule -i selinux/ospd_openvas_podman.cil /usr/share/udica/templates/base_container.cil

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "ospd_openvas_podman"

Deploy ospd-openvas:

.. warning::

    Please wait for ``gvmd-pod`` to become ready and idle before deploying ``ospd-openvas-pod``.

.. code-block:: bash

    podman play kube --configmap configmaps/ospd-openvas.yaml --seccomp-profile-root ./seccomp ospd-openvas-pod.yaml

Create systemd files to run at startup:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name ospd-openvas-pod-srv01
    systemctl --user enable container-ospd-openvas-pod-srv01.service

Deploy gsa
----------

From the project root directory, ``cd`` into ``deployment/production/gsa``:

.. code-block:: bash

    cd deployment/production/gsa

Create config file:

.. code-block:: bash

    cp -v configmaps/gsa.yaml{.example,}

Create pod file:

.. code-block:: bash

    cp -v gsa-pod.yaml{.example,}

Create SELinux security policy:

.. code-block:: bash

    cp -v selinux/gsa_podman.cil{.example,}

Load SELinux security policy:

.. code-block:: bash

    sudo semodule -i selinux/gsa_podman.cil /usr/share/udica/templates/base_container.cil

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "gsa_podman"

Deploy gsa:

.. code-block:: bash

    podman play kube --configmap ./configmaps/gsa.yaml --seccomp-profile-root ./seccomp/ gsa-pod.yaml

Create systemd files to run at startup:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name gsa-pod-srv01
    systemctl --user enable container-gsa-pod-srv01.service

The GSA web page can be accessed via http://127.0.0.1:8080.

Testing with gvm-tools
----------------------

.. code-block:: bash

    podman run -it --rm extra2000/greenbone/gvm-tools gvm-cli --log debug --gmp-username admin --gmp-password admin --protocol GMP tls --hostname [GVMD_SERVER_IP] --port 4000 --xml "<get_version/>"


Change user password
--------------------

For example, to change password for ``admin`` user:

.. code-block:: bash

    podman exec -it gvmd-pod-srv01 gvmd --user=admin --new-password="NEW_PASSWORD"

Updating feeds
--------------

Execute the following commands:

.. code-block:: bash

    podman exec -it --user ospd-openvas ospd-openvas-pod-srv01 greenbone-nvt-sync
    podman exec -it gvmd-pod-srv01 greenbone-feed-sync --type SCAP
    podman exec -it gvmd-pod-srv01 greenbone-feed-sync --type CERT
    podman exec -it gvmd-pod-srv01 greenbone-feed-sync --type GVMD_DATA

