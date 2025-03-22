# SETUP FOR SPARK ON UBUNTU 22.04

Installs `Ubuntu 22.04.5 LTS 20241002.0.0` with the necessary configuration to run `Spark` with `MySQL` and `PyTorch` on any system using Ansible, thereby laying the foundation for any `Machine Learning` project with a consistent configuration.

## Installation with VirtualBox and Vagrant
1. Install Oracle VM VirtualBox (https://www.oracle.com/virtualization/technologies/vm/downloads/virtualbox-downloads.html) and Vagrant (https://developer.hashicorp.com/vagrant/install).
2. Add `C:\Program Files\Oracle\VirtualBox\` and `C:\Program Files\Vagrant\bin` to the system Path (on Windows) if they are not already added.
3. Run `vagrant up` (or `vagrant provision` for subsequent runs).

- There is a bug in this virtual machine that causes it to hang during creation at the step `default: SSH auth method: private key`. To fix this, go to Settings > Network > Advanced in `Oracle VM VirtualBox`, uncheck and re-check the `Cable Connected` option. For more information: https://stackoverflow.com/questions/38463579/vagrant-hangs-at-ssh-auth-method-private-key.


- The username and password for the created virtual machine are `vagrant` and `vagrant`, respectively.
- To connect to the virtual machine with Visual Studio Code, create `C:\Users\[User]\.ssh\` with the following content:

```ssh
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile "[path_to_project]/.vagrant/machines/default/virtualbox/private_key"
  IdentitiesOnly yes
  LogLevel FATAL
  PubkeyAcceptedKeyTypes +ssh-rsa
  HostKeyAlgorithms +ssh-rsa
```

This is simply the output of the `vagrant ssh-config` command in the terminal (once the virtual machine is up).

- To run `PySpark` in the virtual machine terminal, use:

```bash
  /opt/spark-3.5.4-bin-hadoop3/bin/pyspark \
  --driver-memory 16000m \
  --conf "spark.executor.memory=15g" \
  --jars /usr/share/java/mysql-connector-java-9.1.0.jar
```

or in a single line:

```bash
  /opt/spark-3.5.4-bin-hadoop3/bin/pyspark --jars /usr/share/java/mysql-connector-java-9.1.0.jar
```

Replace the Spark and connector versions with the ones installed.


## Installation on `Ubuntu Terminal` (22.04.5 LTS) for Windows
1. Install Ansible in the Ubuntu terminal on Windows:
```bash
  sudo apt update && sudo apt upgrade -y
  sudo apt-add-repository --yes --update ppa:ansible/ansible
  sudo apt install ansible -y
```

2. Install `openssh` (in the Windows Ubuntu terminal) with:
```bash
  sudo apt update
  sudo apt install openssh-client openssh-server -y
```

3. Verify the service and start it if not running:
```bash
  sudo service ssh status
  sudo service ssh start

4. Connect from PowerShell (as administrator) in Windows with:
```bash
  ssh [usuario]@localhost
```

5. To connect from any terminal (e.g., Visual Studio Code), run the command `wsl`. You can also run `wsl` followed by a specific command. For example, to run Ansible (from the Ubuntu terminal on Windows):
```bash
  wsl ansible-playbook -i 'localhost,' provisioning/playbook_local.yml
```

To verify that everything is installed correctly, run:

```bash
  echo $JAVA_HOME
  echo $SPARK_HOME
  echo $PYSPARK_DRIVER_PYTHON
  echo $PYSPARK_DRIVER_PYTHON_OPTS
  echo $PATH
  echo $LD_LIBRARY_PATH 
  $SPARK_HOME/bin/pyspark \
  --driver-memory 16000m \
  --conf "spark.executor.memory=15g" \
  --jars /usr/share/java/mysql-connector-java-9.1.0.jar
```

Then, for testing that everything is correctly installed, execute the files `spark-test.ipynb`, `connect_spark_to_sql-test.ipynb`, and `torch-test.ipynb`.

To run Spark in `yarn` mode:

```bash
  $SPARK_HOME/bin/pyspark --master yarn 
```

And to confirm the `Hadoop` application is running, go to http://localhost:8088.

6. In a file named `inventory` (is a file without extension), the hosts should be:
```ssh
  192.168.33.10 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/default/virtualbox/private_key
  localhost ansible_user=fernando ansible_become=true ansible_become_password=fernando
```

The Ubuntu terminal on Windows with everything installed should occupy `~23,0 GB`.

## Using Ansible with an SSH connection only

Ansible can be run to connect to a machine via SSH. Specify the port and password in the inventory file and run:

```bash
  wsl ansible-playbook -i inventory provisioning/playbook_ssh.yml
```

Note that we are running Ansible from the Ubuntu terminal for an `SSH` port that differs from Ubuntu's terminal itself. Ansible simply does not exist for Windows, so we must use it with the Ubuntu terminal. Previously, it just so happened that the host running Ansible was the same as the target host, but this is not mandatory. All that is needed is an `SSH` connection to run Ansible.

In my case, to connect from the Windows Ubuntu terminal to a VirtualBox virtual machine via `SSH`, I had to right-click on the virtual machine, then `Settings` > `Network`. In the `Attached to` section, select the `Bridge Adapter` option, and in the Name field, choose the `Wi-Fi` adapter (in my case, Intel(R) Wi-Fi 6E AX211 160MHz). Additionally, ensure that `Cable Connected` is selected under the Advanced section.

Next, log in to the virtual machine and run `ip a`, then note the `IP` address `192.168.x.x`. This allows the Ubuntu terminal on Windows to establish a connection with the virtual machine using:

```bash
  ssh [vbox_user]@192.168.x.x
```

The corresponding line in the `inventory` file should then be:

```bash
  192.168.x.x ansible_user=[vbox_user] ansible_become_password=[vbox_pass] ansible_ssh_port=22
```

Finally, note that you can add comments in the `inventory` file using `#`.

## Commands for using Spark:
```bash
  sudo service mysql start
  export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
  export SPARK_HOME=/opt/spark-3.5.4-bin-hadoop3
  export PYSPARK_DRIVER_PYTHON=jupyter
  export PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port=8889'
  /opt/spark-3.5.4-bin-hadoop3/bin/pyspark \
  --driver-memory 16000m \
  --conf "spark.executor.memory=15g" \
  --jars /usr/share/java/mysql-connector-java-9.1.0.jar
```

With GPU Acceleration:
```bash
  /home/fernando/spark-3.5.4-bin-hadoop3/bin/pyspark \
  --driver-memory 16000m \
  --conf "spark.executor.memory=15g" \
  --conf spark.executor.resource.gpu.amount=1 \
  --conf spark.rapids.sql.explain=NONE \
  --jars /usr/share/java/mysql-connector-java-8.2.0.jar,/opt/jars/cudf-24.04.0-cuda12,/home/fernando/jars/rapids-4-spark_2.12-24.04.0-cuda12-arm64.jar
```

## Setting up a Spark cluster

Visit https://chatgpt.com/share/6776c47d-41d0-8005-bab8-585cf5fad2d8

To stop and restart all processes:
```bash
  $HADOOP_HOME/sbin/stop-all.sh
  $HADOOP_HOME/sbin/start-all.sh
```


## Useful commands in Ubuntu
Check the status of the `SSH` connection::
```bash
  sudo service ssh status
```

Start the `SSH` connection:
```bash
  sudo service ssh start
```

Check `SSH` ports:
```bash
  sudo ip a
```

Connect to the `SSH` port:
```bash
  ssh -p 22 fernando@localhost
```

Here, `22` is the port number. If you're installing an Ubuntu virtual machine with VirtualBox, you need to forward the port to another one (e.g., `2222`) to avoid conflicts with the Ubuntu terminal connection. Additionally, it may be necessary to delete `C:\Users\[User]\.ssh\known_hosts` if Windows only detects the Ubuntu terminal.

To create a new role, run the following in the `provisioning/roles/` directory:

```bash
  wsl ansible-galaxy init provisioning/roles/[role_name]
```

## References
- `Ansible: From Beginner to Pro`, Heap, Michaels
- `Big Data Analytics with Spark: A Practitioner's Guide to Using Spark for Large Scale Data Analysis`, Guller, Mohammed