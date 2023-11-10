---
layout: distill
title: host and remote
description: data Synchronization between host and remote machines
giscus_comments: false
date: 2023-07-21

toc:
  - name: SSH
  - name: Synchronization tools
  - name: Reference
---

## SSH
### Port forwarding
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/ssh-port-forwarding-case-1.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

<div class="caption">
    Fig 1. LocalForward
</div>


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/ssh-port-forwarding-case-2.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

<div class="caption">
    Fig 2. RemoteForward
</div>

### Trouble shooting
If the message doesn’t include the authentication method you want to use, take a look at the `/etc/ssh/sshd_config` configuration file. There can be a `AuthenticationMethods publickey password` to list all allowed methods if not using default setting. In addition, It’s a common error to accidentally set the `PasswordAuthentication` value to `yes` but `PermitRootLogin` to `no` or `without-password` when logging in as root.

Ensure that the appropriate configuration for your login method is set, then restart the service.

```bash
sudo restart ssh service
```

Another common error for authentication issue is error in Key Permissions And Ownership. 

## Synchronization tools

### scp

```bash
scp [OPTION] [[user@]SRC_HOST:]file1 [[user@]DEST_HOST:]file2
# OPTION:
# -P: Specifies the remote host ssh port.
# -p: Preserves files modification and access times.
# -r: This option tells scp to copy directories recursively.
# -v: verbose.
# -q: suppress the progress meter and non-error messages.
# -C: forces scp to compresses the data as it is sent to the destination machine.
```

The `scp` command relies on `ssh` for data transfer, so it requires an `ssh`key or password to authenticate on the remote systems.

The colon (`:`) is how `scp` distinguish between local and remote locations.

Examples:
```bash
# From local host to a remote host:
scp -v -rp /some/local/path/ <DEVBOX_ID>:/path/at/dest_host

# If SSH on the remote host is listening on a port other than the default 22 
# then you can specify the port using the -P argument:
scp -P 2322 <file.txt> <remote_username>@10.10.0.2:/remote/directory

# From a remote host to local host
scp -v -rp <DEVBOX_ID>:/path/at/dest_host /some/local/path/

# Between two remote hosts
scp user1@host1.com:/files/file.txt user2@host2.com:/files

# To route the traffic through the machine on which the command is issued, 
# use the -3 option:
scp -3 user1@host1.com:/files/file.txt user2@host2.com:/files
```

Unlike `rsync` , when using `scp` you don’t have to log in to one of the servers to transfer files from one to another remote machine.

### rsync

```bash
rsync [OPTION] SRC_HOST DEST_HOST

# pull from remote to local
rsync [OPTION] USER@HOST:SRC DEST

# push from local to remote
rsync [OPTION] SRC USER@HOST:DEST

# OPTION:
# -a: archive mode
# -v: verbose
# -n or --dry-run: execute a test operation without making real changes.
# -u or --update: skip files that are still new in the destination directory
# --ignore-existing: only sync new files.
# --existing: only update existing files, no new files are pulled.
```

Command `rsync` need to be used in one of the machines involved in the syncing operation.

One imperative differential of `rsync` in comparison to other file-coying commands in Linux is its use of the remote-update protocol, to transfer *only the difference* between files or directory content. By default, `rsync` only copies new or changed files from a source to destination.

Always do a `dry-run` before making real changes:
The `--update` or `-u` option allows `rsync` to skip files that are still new in the destination directory, and one important option, `--dry-run` or `-n` enables us to execute a test operation without making any changes. It shows us what files are to be copied.

Examples:
```bash 
# push (dry-run)
rsync -nav --ignore-existing Documents/* aaronkilik@10.42.1.5:~/all/

# pull (dry-run)
rsync -nav <DEVBOX_ID>:/path/at/dest_host /some/local/path/ 

# push (dry-run)
rsync -nav /some/local/path/ <DEVBOX_ID>:/path/at/dest_host
```

## Reference
<a href="https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot">Difference between ssh local and remote port forwarding</a>

<a href="https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/">How to Use SCP Command to Securely Transfer Files</a>

<a href="https://www.tecmint.com/sync-new-changed-modified-files-rsync-linux/">How to Use Rsync to Sync New or Changed/Modified Files in Linux</a>

<a href="https://www.hostinger.com/tutorials/how-to-use-rsync">How to Use the Linux rsync Command for Remote Directory Synchronization</a>

<a href="https://linuxize.com/post/how-to-setup-passwordless-ssh-login/">SSH key-based authentication</a> and <a href="https://linuxize.com/post/using-the-ssh-config-file/">SSH config file configuration</a>

<a href="https://docs.digitalocean.com/support/how-to-troubleshoot-ssh-authentication-issues/">How to troubleshoot ssh authentication issues</a>

<a href="https://rabexc.org/posts/pitfalls-of-ssh-agents">The pitfalls of using ssh-agent, or how to use an agent safely</a>

<a href="https://github.com/ccontavalli/ssh-ident">ssh-ident:
different agents and different keys for different projects, with ssh.</a>
