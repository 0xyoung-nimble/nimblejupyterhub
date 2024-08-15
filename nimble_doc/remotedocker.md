### Prerequisites

- **Remote Host**: The remote machine where Docker will run. Docker should be installed on this machine.
- **JupyterHub Server**: The server where JupyterHub is running. This machine will communicate with the remote Docker host.

### Step 1: Configure Docker on the Remote Host

#### 1.1 Enable Remote Access to Docker Daemon

By default, Docker listens for commands on a Unix socket (only accessible locally). To allow remote access, you need to configure Docker to listen on a TCP port.

1. **Edit the Docker daemon configuration file**:

   On the remote machine, open the Docker configuration file (usually located at `/etc/docker/daemon.json`):

   ```bash
   sudo vi /etc/docker/daemon.json
   ```

2. **Add the following configuration**:

   This configuration enables Docker to listen on all network interfaces on port `2376` (you can specify a different port if needed).

   ```json
   {
     "hosts": ["tcp://0.0.0.0:2376", "unix:///var/run/docker.sock"]
   }
   ```

3. **Restart Docker**:

   After saving the configuration, restart the Docker service:

   ```bash
   sudo systemctl restart docker
   ```

#### 1.2 Secure Docker Daemon with TLS (Optional)


1. **USE nimble domain TLS Certificates**:

   Check the nimble domain TLS

2. **Configure Docker Daemon to Use TLS**:

   Edit the Docker configuration file to include paths to the TLS certificates:

   ```json
   {
     "hosts": ["tcp://0.0.0.0:2376", "unix:///var/run/docker.sock"],
     "tls": true,
     "tlscacert": "/path/to/ca.pem",
     "tlscert": "/path/to/server-cert.pem",
     "tlskey": "/path/to/server-key.pem",
     "tlsverify": true
   }
   ```

   Replace `"/path/to/"` with the actual path to the certificate files.

3. **Restart Docker**:

   Restart the Docker service to apply the changes:

   ```bash
   sudo systemctl restart docker
   ```

#### 1.3 Open Firewall Port If Needed

Ensure that the port Docker is listening on (e.g., `2376`) is open on the remote machine's firewall:

```bash
sudo ufw allow 2376/tcp
```

### Step 2: Configure JupyterHub to Use the Remote Docker Host

On the JupyterHub server, configure DockerSpawner to connect to the remote Docker daemon.



#### 2.2 Configure JupyterHub

1. Install packages and jupyterHub in development

   ```bash
   pip install -r requirements.txt
   pip install -e .
   ```

Edit the `jupyterhub_config.py` file to configure DockerSpawner to use the remote Docker host:



   [Jupyter Base Notebook on Docker Hub](https://hub.docker.com/r/jupyter/base-notebook/)
   
2. **Configuration with TLS**:

   If you are using TLS, configure the paths to the TLS certificates:

   ```python
   c.JupyterHub.spawner_class = 'dockerspawner.DockerSpawner'

   c.DockerSpawner.host_ip = '<REMOTE_HOST_IP_OR_HOST>'
   c.DockerSpawner.port = 2376
   c.DockerSpawner.tls_verify = True
   c.DockerSpawner.tls_ca = '/path/to/ca.pem'
   c.DockerSpawner.tls_cert = '/path/to/client-cert.pem'
   c.DockerSpawner.tls_key = '/path/to/client-key.pem'

   c.DockerSpawner.image = 'jupyter/base-notebook:latest'

   c.DockerSpawner.extra_host_config = {
       'network_mode': 'bridge',
   }
   ```


3. **Restart JupyterHub**:

   Restart JupyterHub to apply the configuration changes:

   ```bash
   jupyterhub -f jupyterhub_config.py 
   ```

### Step 3: Testing the Configuration

1. **Access JupyterHub**:

   Access your JupyterHub instance through the web browser. When users log in and spawn a notebook server, the container should be created on the remote Docker host.

2. **Verify**:

   On the remote Docker host, you can verify that containers are being created by running:

   ```bash
   docker ps
   ```

   You should see the containers corresponding to the users logged into JupyterHub.
