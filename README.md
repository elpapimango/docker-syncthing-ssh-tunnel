# Docker SSH Tunnel

This Docker creates a simple SSH tunnel to a remote server. This is taken from Syncthings documents.

https://docs.syncthing.net/v1.29.3/users/tunneling.html#create-the-ssh-tunnel

## Usage

### Docker

1. First you should create a config file in your local directory. For simplicity you can create this file in `~/.ssh` in your local machine.

2. Inside `~/.ssh/config` put these lines:

```
    Host syncthing-tunnel                     # You can use any name
            HostName ssh-tunnel.corporate.tld # Tunnel 
            IdentityFile ~/.ssh/id_rsa        # Private key location
            User syncguy                      # Username to connect to SSH service
            ForwardAgent yes
            TCPKeepAlive yes
            ConnectTimeout 5
            ServerAliveCountMax 10
            ServerAliveInterval 15
```

3. Don't forget to put your private key (`id_rsa`) to `~/.ssh` folder.

4. Now in `docker-compose.yml` you can define the tunnel as follows:

```
    version: '2'
    services:
      mysql:
        image: ghcr.io/elpapimango/docker-syncthing-ssh-tunnel
        volumes:
          - $HOME/.ssh:/root/ssh:ro
        environment:
	        SSH_DEBUG: "-v"
          TUNNEL_HOST: syncthing-tunnel
          REMOTE_HOST: 127.0.0.1
          LOCAL_PORT: 22001
          REMOTE_PORT: 22000
```

5. Run `docker-compose up -d`


Create the SSH Tunnel

First open a tunnel from hosta to hostb by running the SSH client on hosta, such that localhost:22001/TCP on each machine redirects to localhost:22000/TCP on the other (for syncthing to use):

127.0.0.1 is explicitly used throughout the example so the tunnels and Syncthing do NOT listen on externally exposed interfaces, for better security.
Listen on localhost

Now in Syncthing on both sides of the tunnel (hosta and hostb) in Settings | Connections, you can disable/uncheck all options: Enable NAT Traversal, Local Discovery, Global Discovery, and Enable Relaying. Also configure Syncthing to listen only on localhost by setting Sync Protocol Listen Addresses to:

tcp://127.0.0.1:22000

Provide Address for Remote Device

Next add the remote device and use Edit | Advanced to assign the Addresses of:

tcp://127.0.0.1:22001

Port 22001/TCP is the SSH tunnel that will redirect to localhost port 22000/TCP on the other machine. This same configuration is done on both hosta and hostb, and then they can find each other through the tunnel.
