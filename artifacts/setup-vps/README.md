# SSH configuration

1. Save the private key provided in https://sec23fallae.usenix.hotcrp.com/paper/24 as `~/.ssh/usenix-ae-reviewer` to your local machine.

2. Make the private key read-only: `chmod 600 ~/.ssh/usenix-ae-reviewer`.

3. Append the following text to `~/.ssh/config`:

```txt
Host usenix-ae-*
     Port 22
     User root
     IdentityFile ~/.ssh/usenix-ae-reviewer

Host usenix-ae-server-us
     HostName 198.199.93.138

Host usenix-ae-client-china
     HostName 59.110.233.201
```

4. Now you should be able to log in to the VPSes with:

```sh
ssh usenix-ae-server-us
ssh usenix-ae-client-china
```

## Basic test

To quickly test you have properly set up a pair of VPSes that can trigger the blocking:

Send some random probes from `usenix-ae-client-china` to port `2` of `usenix-ae-server-us` by repetitively executing the following command:

```sh
head -c200 /dev/urandom | nc -vn 198.199.93.138 2
```

After executing the command a few times (1 time to 15 times), you will notice the `nc` cannot connect to `198.199.93.138:2` anymore. Congratulations! The blocking is triggered. You should still be able to connect to other ports of the same server, for example, `198.199.93.138:3`.

Alternatively, you can use the triggering tools:

```sh
echo 198.199.93.138 | ./utils/affected-norand -p 2 -log /dev/null
```

This tool will take a list of IPs on stdin, and perform (default 25) repeated connections to
the specified port, sending the
same (configurable) random payload in each connection. If the tool is unable to connect for
(default 5) consecutive connections in a row, the tool labels the IP as `affected` by
blocking (`true` in the `affected` column):

```txt
endTime,addr,countSuccess,totalTimeout,consecutiveTimeout,code,affected
1678218390,198.199.93.138:2,2,3,3,timeout,true
```


## (Optional) Set up client and server yourself

Be cautious that running the set-up scripts below will disrupt other reviewers' ongoing experiments.
AE reviewers don't have to do anything to set up the client or server because we've already prepared the server for you.

In case you need to set them up yourself:

* run this script to compile a sink server, rsync the binary to the server, and make the server listen on all ports from 0-65535 for the specific client IP address:

```sh
./setup-server/to_digitalocean_server.sh
```

* run this script to compile all necessary tools, and rsync the binaries to the server:

```sh
./setup-client/to_alibaba_server.sh
```
