# USENIX Security 2023 AE

This repo includes the artifacts for the [USENIX Security 2023](https://www.usenix.org/conference/usenixsecurity23) paper "How China Detects and Blocks Fully Encrypted Traffic".

## Overview of the Repo Structure

```txt
.
├── ae-appendix
├── artifacts
│   ├── common
│   ├── setup-vps
│   ├── sink-server
│   ├── test-entropy
│   ├── test-printable-fraction
│   ├── test-printable-longest-run
│   ├── test-printable-prefixes
│   ├── test-protocol-fingerprints
│   └── utils
├── CHECKLIST
├── LICENSE
└── README.md
```

* `ae-appendix` contains the source code and Makefile to generate the artifact appendix.
* `artifacts/setup-vps` contains the source code to set up remote VPSes.
* `artifacts/sink-server` contains the source code for [a sink server](./artifacts/sink-server/), which runs on the server side.
* `artifacts/utils` contains [client-side testing tools](./artifacts/utils/).
* `artifacts/test-*` contain five different tests. Each of them corresponds to a claim.
* `artifacts/common-*` is a module that contains code on which measurement tools are built.

## VPS Information

| Location                              | ASN     | CPU Model                | # Core(s) | RAM  | OS                                                      |
|---------------------------------------|---------|--------------------------|-----------|------|---------------------------------------------------------|
| AlibabaCloud Beijing Datacenter       | AS37963 | Intel Xeon Platinum 8163 | 1         | 1GB  | Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-56-generic x86_64)   |
| DigitalOcean San Francisco Datacenter | AS14061 | Intel DO-Regular         | 1         | 1GB  | Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-88-generic x86_64) |

## Configuration

1. We refer reviewers to this [README](./artifacts/setup-vps/README.md) for detailed instructions on how to access our VPSes.

2. (Optional as we've done it for you) To set up the client (VPS in China):

```sh
./artifacts/setup-client/to_alibaba_server.sh
```

3. (Optional as we've done it for you) To set up the server (VPS in the US):

```sh
./artifacts/setup-server/to_digitalocean_server.sh
```

## Minimal Working Example

1. First login to the VPS in China:

`ssh usenix-ae-client-china`.

2. Send some random probes from `usenix-ae-client-china` to the port `2` of `usenix-ae-server-us` by repetitively executing the following command:

```sh
head -c200 /dev/urandom | nc -vn 198.199.93.138 2
```

3. After executing the command a few times (1 time to 15 times), you will notice the `nc` cannot connect to `198.199.93.138:2` anymore. Congratulations! The blocking is triggered (and will residually last for up to a few minutes).
You should still be able to connect to other ports of the same server, for example, `198.199.93.138:3`.

4. (Optional) Alternatively, one can use the triggering tools:

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
1678258922,198.199.93.138:2,2,5,5,timeout,true
```

This output means that connecting to the endpoint (198.199.93.138:2)
succeeded in 2 connections, but then
had 5 consecutive connections timeout in a row (and a total of 5 failed).
Because there was at least 5 consecutive timeouts, our tool labels this
endpoint/payload combination as affected (true).

## Reproduce Experiments

| Experiments                     | Human Time (minutes) | Compute Time (minutes) |
|---------------------------------|------------|--------------|
| Ex1: test-random                |      5      |        5      |
| [Ex2: test-entropy](./artifacts/test-entropy/)                |      30      |       30       |
| [Ex3: test-printable-prefixes](./artifacts/test-printable-prefixes/)     |    15        |     30         |
| [Ex4: test-printable-fraction](./artifacts/test-printable-fraction/)     |     15       |     30         |
| [Ex5: test-printable-longest-run](./artifacts/test-printable-longest-run/) |     15       |   15           |
| [Ex6: test-protocol-fingerprints](./artifacts/test-protocol-fingerprints/) |      15      |       240       |
