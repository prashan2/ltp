# LTP Network Tests

## Pre-requisites
Enable all the networking services on both client and server machines:
rshd, nfsd, fingerd.

## Server Configuration
* Verify that the below daemon services are running. If not, please install
and start them:
rsh-server, telnet-server, finger-server, rdist, rsync, dhcp-server, http-server.

Note: If any of the above daemon is not running on server, the test related to
that service running from client will fail.

* Edit the "/root/.rhosts" file. Please note that the file may not exist,
so you must create one if it does not. You must add the fully qualified
hostname of the machine you are testing on to this file. By adding the test
machine's hostname to this file, you will be allowing the machine to rsh to itself,
as root, without the requirement of a password.

```sh
echo $client_hostname >> /root/.rhosts
```

You may need to re-label '.rhost' file to make sure rlogind will have access to it:

```sh
/sbin/restorecon -v /root/.rhosts
```

* Add rlogin, rsh, rexec into /etc/securetty file:

```sh
for i in rlogin rsh rexec; do echo $i >> /etc/securetty; done
```

## FTP setup
* In “/etc/ftpusers” [or vi /etc/vsftpd.ftpusers], comment the line containing
“root” string. This file lists all those users who are not given access to do ftp
on the current system.

* If you don’t want to do the previous step, put following entry into /root/.netrc
machine <remote_server_name> login root password <remote_root_password>.
Otherwise, ftp,rlogin & telnet fails for ‘root’ user & hence needs to be
executed using ‘test’ user to get successful results.

## NFS setup
* In “/etc/exports“, add the following:

```
/ <local_machine_ip>/255.255.255.0(rw,no_root_squash,sync)
```

* Then run “exportfs” to get a list of exported file system.

## LTP setup
Install LTP testsuite on both client and server machines.
Make sure testcases and network tools are in PATH, e.g.:

```sh
export PATH=/opt/ltp/testcases/bin:/usr/bin:$PATH
```

The RHOST variable name should be set to the hostname of the server
(test management link) and PASSWD should be set to the root password
of the remote server.

Default values for all network variables are set in testcases/lib/test_net.sh.
If you need to override some parameters please export them before test run or
specify them when running ltp-pan or testscripts/network.sh.

## Running the tests
To run the test type the following:

```sh
TEST_VARS ./network.sh OPTIONS
```
Where
* TEST_VARS - non-default network parameters (see testcases/lib/test_net.sh), they
  could be exported before test run;
* OPTIONS - test group(s), use '-h' to see available ones.

## Analyzing the results
Generally this test must be run more than 24 hours. When you want to stop the test
press CTRL+C to stop ./network.sh.

Search failed tests in LTP logfile using grep FAIL <logfile>. For any failures,
run the individual tests and then try to come to the conclusion.
