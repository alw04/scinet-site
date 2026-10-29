---
title: SSH Tunneling on SCINet
description: Using SSH local port forwarding to access services on SCINet
categories: [Access]

subnav:
  - title: Before You Begin
    url: '#before-you-begin'
  - title: SSH Port Forwarding
    url: '#ssh-port-forwarding'
  - title: Creating an SSH Tunnel
    url: '#creating-an-ssh-tunnel'
    subnav:
      - title: Find the Remote Host and Port
        url: '#find-the-remote-host-and-port'
      - title: Connect Using SSH
        url: '#connect-using-ssh'
      - title: Access the Application
        url: '#access-the-application'
      - title: Forwarding Multiple Ports
        url: '#forwarding-multiple-ports'
  - title: Closing an SSH Tunnel
    url: '#closing-an-ssh-tunnel'
  - title: Troubleshooting
    url: '#troubleshooting'
---

SSH tunneling (also called SSH port forwarding) allows you to securely access a network service running on a remote SCINet system from your local computer. For example, an SSH tunnel can be used to access a web-based application running on a Ceres compute node through your local web browser.

This guide describes **local port forwarding** using the SSH command-line client. If the application you want to use is available through [Open OnDemand](/guides/use/open-ondemand), that interface is generally easier to use and does not require an SSH tunnel.

## Before You Begin

Before creating an SSH tunnel, you will need:

1. An active SCINet account with access to the system you want to use.
2. SSH access configured on your local computer. Follow the [SSH access to SCINet](/guides/access/ssh-login) guide to install and configure SmallStepCLI and complete the initial SSH setup.
3. A terminal application with an SSH client. The commands below use the OpenSSH `ssh` command.
4. The hostname and port number of the service you want to access. If the application runs on a compute node, start it through an appropriate compute allocation and use the hostname and port reported by that application or job.

**Note:** Applications and other compute-intensive tasks should run on the compute nodes, not on cluster login nodes. See the [SLURM Resource Manager](/guides/use/slurm) guide for information about requesting computing resources on Ceres.

## SSH Port Forwarding

Local port forwarding creates a connection between a port on your computer and a port on the remote system through an SSH login host. Once the tunnel is established, you can connect to the remote service using a local address such as `http://localhost:8888`.

The general SSH command for local port forwarding is:

```bash
ssh -N -L LOCAL_PORT:REMOTE_HOST:REMOTE_PORT USERNAME@SSH_HOST
```

The components of this command are:

| Option | Description |
| --- | --- |
| `-N` | Establishes the SSH connection without opening a remote shell. |
| `-L` | Enables local port forwarding. |
| `LOCAL_PORT` | The port on your local computer that you will connect to. |
| `REMOTE_HOST` | The hostname of the system running the application, as reachable from the login host. |
| `REMOTE_PORT` | The port on which the remote application is running. |
| `USERNAME` | Your SCINet username. |
| `SSH_HOST` | The SSH login hostname for the SCINet system. |

The local and remote ports do not have to be the same.

## Creating an SSH Tunnel

The following example shows how to access a web-based application running on a **Ceres compute node**. It assumes the application is already running on an allocated compute node and is accessible from the Ceres SSH login host. The values shown below are examples; replace them with the values for your application.

### Find the Remote Host and Port

1. Start your application by following its instructions and requesting any required compute resources.
2. Identify the hostname of the compute node running the application and the port on which the application is listening. In this example, the hostname is represented by `COMPUTE_NODE` and the application uses port `8888`.
3. Keep the application and its compute job running while using the tunnel.

### Connect Using SSH

1. Open a **new terminal window on your local computer**.
2. Run the following command, replacing `COMPUTE_NODE` with the actual compute-node hostname and `user.name` with your SCINet username:

    ```bash
    ssh -N -L 8888:COMPUTE_NODE:8888 user.name@ceres.scinet.usda.gov
    ```

3. If prompted, complete the SCINet authentication process.
4. Keep the terminal window open while you use the application. After authentication, there may be no additional terminal output. This is normal when using the `-N` option.

**Note:** Run the SSH tunnel command **on your local computer**, not from a terminal already logged into Ceres. The hostname and port must match the running application's settings.

### Access the Application

1. Open the web browser on your local computer.
2. Enter the following address:

    ```text
    http://localhost:8888
    ```

The browser connects to the local port, and SSH forwards the connection to the remote application through Ceres.

If port `8888` is already in use on your local computer, you can select a different local port. For example:

```bash
ssh -N -L 8890:COMPUTE_NODE:8888 user.name@ceres.scinet.usda.gov
```

In this case, access the application at `http://localhost:8890`. The application is still using port `8888` on the remote compute node.

### Forwarding Multiple Ports

If you need to access multiple applications or services running on different ports, you can forward multiple ports through a single SSH connection by including an additional `-L` option for each port.

For example, if one service is running on port `8888` and another is running on port `9000` on the same compute node, you can use:

```bash
ssh -N -L 8888:COMPUTE_NODE:8888 -L 9000:COMPUTE_NODE:9000 user.name@ceres.scinet.usda.gov
```

This forwards both ports through the same SSH connection. You can then access the services at `http://localhost:8888` and `http://localhost:9000`, assuming both services are running and reachable on the compute node.

You can add more `-L` options as needed, using a different available local port for each service.

## Closing an SSH Tunnel

When you are done using the application:

1. Save your work and close the application in your browser, if appropriate.
2. Return to the local terminal window in which you started the SSH tunnel.
3. Press **Ctrl+C** to close the SSH connection and stop port forwarding.
4. If you started the application using a SLURM job, stop that job separately according to the application's instructions. For example, a job can be cancelled using `scancel JOB_ID`, replacing `JOB_ID` with the actual SLURM job ID.

**Important:** Closing the browser tab or SSH tunnel does **not** automatically stop a running SLURM job. Remember to release computing resources when you are finished.

## Troubleshooting

If you cannot connect to an application through the SSH tunnel, check the following:

1. **SSH authentication:** Verify that SmallStepCLI is configured and you can log in using the instructions in the [SSH access guide](/guides/access/ssh-login).
2. **Local port:** If SSH reports that the address is already in use, select another available local port and update the address in your browser.
3. **Remote hostname and port:** Confirm that you used the hostname and address reported by the application or job. The application must be running and reachable from the SSH login host.
4. **Running application:** Verify that the application and its compute job have not stopped. An open SSH tunnel does not mean the remote service is still running.
5. **Terminal window:** Verify the local terminal window containing the SSH command is still open. If you have closed it, create the tunnel again.
