devilapi@krasserserver:~$ wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
-bash: wget: command not found
[sudo] password for devilapi:
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
gpg: no valid OpenPGP data found.
devilapi@krasserserver:~$ sudo apt install wget -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  wget
0 upgraded, 1 newly installed, 0 to remove and 95 not upgraded.
Need to get 984 kB of archives.
After this operation, 3,692 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bookworm/main amd64 wget amd64 1.21.3-1+b2 [984 kB]
Fetched 984 kB in 0s (5,615 kB/s)
Selecting previously unselected package wget.
(Reading database ... 241950 files and directories currently installed.)
Preparing to unpack .../wget_1.21.3-1+b2_amd64.deb ...
Unpacking wget (1.21.3-1+b2) ...
Setting up wget (1.21.3-1+b2) ...
Processing triggers for man-db (2.11.2-2) ...
devilapi@krasserserver:~$ wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
OK
devilapi@krasserserver:~$ sudo apt-key export "$(apt-key list | grep -oP '(?<=pub.*[ /])[0-9A-F]+(?= \[SC\])')" | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/mongodb.gpg > /dev/null
grep: lookbehind assertion is not fixed length
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
gpg: key "" not found: Invalid user ID
gpg: WARNING: nothing exported
gpg: no valid OpenPGP data found.
devilapi@krasserserver:~$ wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/mongodb.gpg > /dev/null
devilapi@krasserserver:~$ ls /etc/apt/trusted.gpg.d/mongodb.gpg
/etc/apt/trusted.gpg.d/mongodb.gpg
devilapi@krasserserver:~$ echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0 main
devilapi@krasserserver:~$ sudo apt update
Get:1 http://security.debian.org/debian-security bookworm-security InRelease [48.0 kB]
Get:2 http://deb.debian.org/debian bookworm InRelease [151 kB]
Get:3 http://deb.debian.org/debian bookworm-updates InRelease [55.4 kB]
Hit:4 https://download.docker.com/linux/debian bookworm InRelease
Get:5 https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0 InRelease [2,906 B]
Get:7 https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0/main amd64 Packages [28.7 kB]
Hit:6 https://packagecloud.io/ookla/speedtest-cli/debian bookworm InRelease
Get:8 https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0/main arm64 Packages [26.3 kB]
Fetched 312 kB in 1s (209 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
95 packages can be upgraded. Run 'apt list --upgradable' to see them.
N: Repository 'Debian bookworm' changed its 'firmware component' value from 'non-free' to 'non-free-firmware'
N: More information about this can be found online in the Release notes at: https://www.debian.org/releases/bookworm/amd64/release-notes/ch-information.html#non-free-split
devilapi@krasserserver:~$ sudo apt install -y mongodb-org
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package mongodb-org
devilapi@krasserserver:~$ devilapi@krasserserver:~$ wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
OK
devilapi@krasserserver:~$ sudo apt-key export "$(apt-key list | grep -oP '(?<=pub.*[ /])[0-9A-F]+(?= \[SC\])')" | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/mongodb.gpg > /dev/null
grep: lookbehind assertion is not fixed length
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
gpg: key "" not found: Invalid user ID
gpg: WARNING: nothing exported
gpg: no valid OpenPGP data found.
devilapi@krasserserver:~$ wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/mongodb.gpg > /dev/null
devilapi@krasserserver:~$ ls /etc/apt/trusted.gpg.d/mongodb.gpg
/etc/apt/trusted.gpg.d/mongodb.gpg
devilapi@krasserserver:~$ echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0 main
devilapi@krasserserver:~$ sudo apt update
Get:1 http://security.debian.org/debian-security bookworm-security InRelease [48.0 kB]
Get:2 http://deb.debian.org/debian bookworm InRelease [151 kB]
Get:3 http://deb.debian.org/debian bookworm-updates InRelease [55.4 kB]
Hit:4 https://download.docker.com/linux/debian bookworm InRelease
Get:5 https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0 InRelease [2,906 B]
Get:7 https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0/main amd64 Packages [28.7 kB]
Hit:6 https://packagecloud.io/ookla/speedtest-cli/debian bookworm InRelease
Get:8 https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0/main arm64 Packages [26.3 kB]
Fetched 312 kB in 1s (209 kB/s)
Reading package lists... Done
devilapi@krasserserver:~$ge mongodb-orgall -y mongodb-org Release notes at: https://www.debian.org/releases/bookworm/amd-bash: devilapi@krasserserver:~$: command not found
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
gpg: no valid OpenPGP data found.
-bash: syntax error near unexpected token `('
-bash: OK: command not found
grep: lookbehind assertion is not fixed length
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
-bash: devilapi@krasserserver:~$: command not found
gpg: no valid OpenPGP data found.
-bash: grep:: command not found
-bash: syntax error near unexpected token `('
-bash: syntax error near unexpected token `('
-bash: gpg:: command not found
-bash: gpg:: command not found
-bash: gpg:: command not found
-bash: devilapi@krasserserver:~$: command not found
gpg: no valid OpenPGP data found.
-bash: devilapi@krasserserver:~$: command not found
-bash: /etc/apt/trusted.gpg.d/mongodb.gpg: Permission denied
-bash: devilapi@krasserserver:~$: command not found
-bash: deb: command not found
-bash: devilapi@krasserserver:~$: command not found
-bash: Get:1: command not found
-bash: Get:2: command not found
-bash: Get:3: command not found
-bash: Hit:4: command not found
-bash: Get:5: command not found
-bash: Get:7: command not found
-bash: Hit:6: command not found
-bash: Get:8: command not found
-bash: syntax error near unexpected token `('
-bash: Reading: command not found
-bash: Building: command not found
-bash: Reading: command not found
-bash: 95: command not found
-bash: N:: command not found
-bash: N:: command not found
-bash: devilapi@krasserserver:~$: command not found
-bash: Reading: command not found
-bash: Building: command not found
-bash: Reading: command not found
-bash: E:: command not found
-bash: devilapi@krasserserver:~$: command not found
devilapi@krasserserver:~$ cat /etc/apt/sources.list.d/mongodb-org-6.0.list
devilapi@krasserserver:~$ echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0 main
devilapi@krasserserver:~$ sudo apt update
Get:1 http://deb.debian.org/debian bookworm InRelease [151 kB]
Hit:2 http://security.debian.org/debian-security bookworm-security InRelease
Hit:3 https://download.docker.com/linux/debian bookworm InRelease
Hit:4 http://deb.debian.org/debian bookworm-updates InRelease
Hit:5 https://repo.mongodb.org/apt/debian bookworm/mongodb-org/6.0 InRelease
Hit:6 https://packagecloud.io/ookla/speedtest-cli/debian bookworm InRelease
Fetched 151 kB in 1s (129 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
95 packages can be upgraded. Run 'apt list --upgradable' to see them.
W: https://repo.mongodb.org/apt/debian/dists/bookworm/mongodb-org/6.0/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
N: Repository 'Debian bookworm' changed its 'firmware component' value from 'non-free' to 'non-free-firmware'
N: More information about this can be found online in the Release notes at: https://www.debian.org/releases/bookworm/amd64/release-notes/ch-information.html#non-free-split
devilapi@krasserserver:~$ sudo apt install -y mongodb-org
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package mongodb-org
devilapi@krasserserver:~$ echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/debian bullseye/mongodb-org/6.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/debian bullseye/mongodb-org/6.0 main
devilapi@krasserserver:~$ sudo apt update
Get:1 http://deb.debian.org/debian bookworm InRelease [151 kB]
Hit:2 http://security.debian.org/debian-security bookworm-security InRelease
Hit:3 http://deb.debian.org/debian bookworm-updates InRelease
Hit:4 https://download.docker.com/linux/debian bookworm InRelease
Get:5 https://repo.mongodb.org/apt/debian bullseye/mongodb-org/6.0 InRelease [2,951 B]
Hit:6 https://packagecloud.io/ookla/speedtest-cli/debian bookworm InRelease
Get:7 https://repo.mongodb.org/apt/debian bullseye/mongodb-org/6.0/main amd64 Packages [82.4 kB]
Get:8 https://repo.mongodb.org/apt/debian bullseye/mongodb-org/6.0/main arm64 Packages [32.9 kB]
Fetched 269 kB in 1s (203 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
95 packages can be upgraded. Run 'apt list --upgradable' to see them.
W: https://repo.mongodb.org/apt/debian/dists/bullseye/mongodb-org/6.0/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
N: Repository 'Debian bookworm' changed its 'firmware component' value from 'non-free' to 'non-free-firmware'
N: More information about this can be found online in the Release notes at: https://www.debian.org/releases/bookworm/amd64/release-notes/ch-information.html#non-free-split
devilapi@krasserserver:~$ sudo apt install -y mongodb-org
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 mongodb-org-mongos : Depends: libssl1.1 (>= 1.1.1) but it is not installable
 mongodb-org-server : Depends: libssl1.1 (>= 1.1.1) but it is not installable
E: Unable to correct problems, you have held broken packages.
devilapi@krasserserver:~$ wget http://ftp.de.debian.org/debian/pool/main/o/openssl/libssl1.1_1.1.1n-1_amd64.deb
--2024-12-27 10:53:46--  http://ftp.de.debian.org/debian/pool/main/o/openssl/libssl1.1_1.1.1n-1_amd64.deb
Resolving ftp.de.debian.org (ftp.de.debian.org)... 2a13:dd80:deb::deb, 141.76.2.4
Connecting to ftp.de.debian.org (ftp.de.debian.org)|2a13:dd80:deb::deb|:80... connected.
HTTP request sent, awaiting response... 404 Not Found
2024-12-27 10:53:46 ERROR 404: Not Found.

devilapi@krasserserver:~$ wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb
--2024-12-27 10:54:45--  http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb
Resolving archive.ubuntu.com (archive.ubuntu.com)... 2620:2d:4000:1::102, 2620:2d:4002:1::103, 2620:2d:4002:1::101, ...
Connecting to archive.ubuntu.com (archive.ubuntu.com)|2620:2d:4000:1::102|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1128092 (1.1M) [application/vnd.debian.binary-package]
Saving to: ‘libssl1.1_1.1.0g-2ubuntu4_amd64.deb’

libssl1.1_1.1.0g-2ubuntu4 100%[===================================>]   1.08M  5.90MB/s    in 0.2s

2024-12-27 10:54:45 (5.90 MB/s) - ‘libssl1.1_1.1.0g-2ubuntu4_amd64.deb’ saved [1128092/1128092]

devilapi@krasserserver:~$ sudo dpkg -i libssl1.1_1.1.0g-2ubuntu4_amd64.deb
Selecting previously unselected package libssl1.1:amd64.
(Reading database ... 242041 files and directories currently installed.)
Preparing to unpack libssl1.1_1.1.0g-2ubuntu4_amd64.deb ...
Unpacking libssl1.1:amd64 (1.1.0g-2ubuntu4) ...
Setting up libssl1.1:amd64 (1.1.0g-2ubuntu4) ...
Processing triggers for libc-bin (2.36-9+deb12u9) ...
devilapi@krasserserver:~$ sudo apt install -y mongodb-org
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 mongodb-org-mongos : Depends: libssl1.1 (>= 1.1.1) but 1.1.0g-2ubuntu4 is to be installed
 mongodb-org-server : Depends: libssl1.1 (>= 1.1.1) but 1.1.0g-2ubuntu4 is to be installed
E: Unable to correct problems, you have held broken packages.
devilapi@krasserserver:~$ sudo apt update
Get:1 http://deb.debian.org/debian bookworm InRelease [151 kB]
Hit:2 http://security.debian.org/debian-security bookworm-security InRelease
Hit:3 https://download.docker.com/linux/debian bookworm InRelease
Hit:4 http://deb.debian.org/debian bookworm-updates InRelease
Hit:5 https://repo.mongodb.org/apt/debian bullseye/mongodb-org/6.0 InRelease
Hit:6 https://packagecloud.io/ookla/speedtest-cli/debian bookworm InRelease
Fetched 151 kB in 1s (101 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
95 packages can be upgraded. Run 'apt list --upgradable' to see them.
W: https://repo.mongodb.org/apt/debian/dists/bullseye/mongodb-org/6.0/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
N: Repository 'Debian bookworm' changed its 'firmware component' value from 'non-free' to 'non-free-firmware'
N: More information about this can be found online in the Release notes at: https://www.debian.org/releases/bookworm/amd64/release-notes/ch-information.html#non-free-split
devilapi@krasserserver:~$ sudo apt install -y mongodb-org
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 mongodb-org-mongos : Depends: libssl1.1 (>= 1.1.1) but 1.1.0g-2ubuntu4 is to be installed
 mongodb-org-server : Depends: libssl1.1 (>= 1.1.1) but 1.1.0g-2ubuntu4 is to be installed
E: Unable to correct problems, you have held broken packages.
devilapi@krasserserver:~$ sudo apt --fix-broken install
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
0 upgraded, 0 newly installed, 0 to remove and 95 not upgraded.
devilapi@krasserserver:~$ sudo apt install -y mongodb-org
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 mongodb-org-mongos : Depends: libssl1.1 (>= 1.1.1) but 1.1.0g-2ubuntu4 is to be installed
 mongodb-org-server : Depends: libssl1.1 (>= 1.1.1) but 1.1.0g-2ubuntu4 is to be installed
E: Unable to correct problems, you have held broken packages.
devilapi@krasserserver:~$ devilapi@krasserserver:~$ wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2sudo apt install -y mongodb-org
                          devilapi@krasserserver:~$ wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb
--2024-12-27 10:54:45--  http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb
Resolving archive.ubuntu.com (archive.ubuntu.com)... 2620:2d:4000:1::102, 2620:2d:4002:1::103, 2620:2d:4002:1::101, ...
Connecting to archive.ubuntu.com (archive.ubuntu.com)|2620:2d:4000:1::102|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1128092 (1.1M) [application/vnd.debian.binary-package]
Saving to: ‘libssl1.1_1.1.0g-2ubuntu4_amd64.deb’

libssl1.1_1.1.0g-2ubuntu4 100%[===================================>]   1.08M  5.90MB/s    in 0.2s

2024-12-27 10:54:45 (5.90 MB/s) - ‘libssl1.1_1.1.0g-2ubuntu4_amd64.deb’ saved [1128092/1128092]

devilapi@krasserserver:~$ sudo dpkg -i libssl1.1_1.1.0g-2ubuntu4_amd64.deb
Selecting previously unselected package libssl1.1:amd64.
(Reading database ... 242041 files and directories currently installed.)
Preparing to unpack libssl1.1_1.1.0g-2ubuntu4_amd64.deb ...
Unpacking libssl1.1:amd64 (1.1.0g-2ubuntu4) ...
Setting up libssl1.1:amd64 (1.1.0g-2ubuntu4) ...
Processing triggers for libc-bin (2.36-9+deb12u9) ...
devilapi@krasserserver:~$ sudo apt install -y mongodb-org
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 mongodb-org-mongos : Depends: libssl1.1 (>= 1.1.1) but 1.1.0g-2ubuntu4 is to be installed
 mongodb-org-server : Depends: libssl1.1 (>= 1.1.1) but 1.1.0g-2ubuntu4 is to be installed
E: Unable to correct problems, you have held broken packages.
devilapi@krasserserver:~$ sudo apt update
Get:1 http://deb.debian.org/debian bookworm InRelease [151 kB]
Hit:2 http://security.debian.org/debian-security bookworm-security InRelease
Hit:3 https://download.docker.com/linux/debian bookworm InRelease
Hit:4 http://deb.debian.org/debian bookworm-updates InRelease
Hit:5 https://repo.mongodb.org/apt/debian bullseye/mongodb-org/6.0 InRelease
Hit:6 https://packagecloud.io/ookla/speedtest-cli/debian bookworm InRelease
Fetched 151 kB in 1s (101 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
95 packages can be upgraded. Run 'apt list --upgradable' to see them.
W: https://repo.mongodb.org/apt/debian/dists/bullseye/mongodb-org/6.0/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
N: Repository 'Debian bookworm' changed its 'firmware component' value from 'non-free' to 'non-free-fi
-bash: devilapi@krasserserver:~$: command not found
-bash: --2024-12-27: command not found
-bash: syntax error near unexpected token `('
-bash: syntax error near unexpected token `('
-bash: HTTP: command not found
-bash: syntax error near unexpected token `('
-bash: Saving: command not found
-bash: libssl1.1_1.1.0g-2ubuntu4: command not found
-bash: syntax error near unexpected token `('
-bash: devilapi@krasserserver:~$: command not found
-bash: Selecting: command not found
-bash: Reading: command not found
-bash: Preparing: command not found
-bash: syntax error near unexpected token `('
-bash: syntax error near unexpected token `('
-bash: syntax error near unexpected token `('
-bash: devilapi@krasserserver:~$: command not found
-bash: Reading: command not found
-bash: Building: command not found
-bash: Reading: command not found
-bash: Some: command not found
-bash: requested: command not found
-bash: distribution: command not found
-bash: or: command not found
-bash: The: command not found
-bash: The: command not found
-bash: Hit:3: command not found
-bash: Hit:4: command not found
-bash: Hit:5: command not found
-bash: Hit:6: command not found
-bash: syntax error near unexpected token `('
-bash: Reading: command not found
-bash: Building: command not found
-bash: Reading: command not found
-bash: 95: command not found
-bash: syntax error near unexpected token `('
-bash: N:: command not found
-bash: N:: command not found
-bash: devilapi@krasserserver:~$: command not found
-bash: Reading: command not found
-bash: Building: command not found
-bash: Reading: command not found
-bash: Some: command not found
-bash: requested: command not found
-bash: distribution: command not found
-bash: or: command not found
-bash: The: command not found
-bash: The: command not found
-bash: syntax error near unexpected token `('
-bash: syntax error near unexpected token `('
-bash: E:: command not found
-bash: devilapi@krasserserver:~$: command not found
-bash: Reading: command not found
-bash: Building: command not found
-bash: Reading: command not found
-bash: 0: command not found
-bash: devilapi@krasserserver:~$: command not found
-bash: Reading: command not found
-bash: Building: command not found
-bash: Reading: command not found
devilapi@krasserserver:~$ sudo apt remove --purge libssl1.1
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be REMOVED:
  libssl1.1*
0 upgraded, 0 newly installed, 1 to remove and 95 not upgraded.
After this operation, 3,493 kB disk space will be freed.
Do you want to continue? [Y/n] y
(Reading database ... 242050 files and directories currently installed.)
Removing libssl1.1:amd64 (1.1.0g-2ubuntu4) ...
Processing triggers for libc-bin (2.36-9+deb12u9) ...
(Reading database ... 242041 files and directories currently installed.)
Purging configuration files for libssl1.1:amd64 (1.1.0g-2ubuntu4) ...
devilapi@krasserserver:~$ wget http://security.debian.org/debian-security/pool/updates/main/o/openssl/libssl1.1_1.1.1n-1+deb11u5_amd64.deb
--2024-12-27 10:58:57--  http://security.debian.org/debian-security/pool/updates/main/o/openssl/libssl1.1_1.1.1n-1+deb11u5_amd64.deb
Resolving security.debian.org (security.debian.org)... 2a04:4e42:400::644, 2a04:4e42:200::644, 2a04:4e42:600::644, ...
Connecting to security.debian.org (security.debian.org)|2a04:4e42:400::644|:80... connected.
HTTP request sent, awaiting response... 404 Not Found
2024-12-27 10:58:57 ERROR 404: Not Found.

devilapi@krasserserver:~$ client_loop: send disconnect: Connection reset

C:\Users\vau8>ssh devilapi@node03.krasserserver.com
devilapi@node03.krasserserver.com's password:
Preparing to unpack .../04-cgroupfs-mount_1.4_all.deb ...
Unpacking cgroupfs-mount (1.4) ...
Selecting previously unselected package python3-protobuf.
Preparing to unpack .../05-python3-protobuf_3.21.12-3_amd64.deb ...
Unpacking python3-protobuf (3.21.12-3) ...
Selecting previously unselected package libnet1:amd64.
Preparing to unpack .../06-libnet1_1.1.6+dfsg-3.2_amd64.deb ...
Unpacking libnet1:amd64 (1.1.6+dfsg-3.2) ...
Selecting previously unselected package criu.
Preparing to unpack .../07-criu_3.17.1-2_amd64.deb ...
Unpacking criu (3.17.1-2) ...
Selecting previously unselected package libintl-perl.
Preparing to unpack .../08-libintl-perl_1.33-1_all.deb ...
Unpacking libintl-perl (1.33-1) ...
Selecting previously unselected package libintl-xs-perl.
Preparing to unpack .../09-libintl-xs-perl_1.33-1_amd64.deb ...
Unpacking libintl-xs-perl (1.33-1) ...
Selecting previously unselected package libmodule-find-perl.
Preparing to unpack .../10-libmodule-find-perl_0.16-2_all.deb ...
Unpacking libmodule-find-perl (0.16-2) ...
Selecting previously unselected package libproc-processtable-perl:amd64.
Preparing to unpack .../11-libproc-processtable-perl_0.634-1+b2_amd64.deb ...
Unpacking libproc-processtable-perl:amd64 (0.634-1+b2) ...
Selecting previously unselected package libsort-naturally-perl.
Preparing to unpack .../12-libsort-naturally-perl_1.03-4_all.deb ...
Unpacking libsort-naturally-perl (1.03-4) ...
Selecting previously unselected package needrestart.
Preparing to unpack .../13-needrestart_3.6-4+deb12u3_all.deb ...
Unpacking needrestart (3.6-4+deb12u3) ...
Setting up libnet1:amd64 (1.1.6+dfsg-3.2) ...
Setting up runc (1.1.5+ds1-1+deb12u1) ...
Setting up libmodule-find-perl (0.16-2) ...
Setting up tini (0.19.0-1) ...
Setting up libproc-processtable-perl:amd64 (0.634-1+b2) ...
Setting up libintl-perl (1.33-1) ...
Setting up cgroupfs-mount (1.4) ...
Setting up python3-protobuf (3.21.12-3) ...
Setting up containerd (1.6.20~ds1-1+b1) ...
Installing new version of config file /etc/containerd/config.toml ...
Setting up libsort-naturally-perl (1.03-4) ...
Setting up needrestart (3.6-4+deb12u3) ...
Setting up docker.io (20.10.24+dfsg1-1+deb12u1) ...
Installing new version of config file /etc/default/docker ...
Installing new version of config file /etc/init.d/docker ...
Setting up libintl-xs-perl (1.33-1) ...
Setting up criu (3.17.1-2) ...
Processing triggers for man-db (2.11.2-2) ...
Processing triggers for libc-bin (2.36-9+deb12u9) ...
devilapi@krasserserver:~$ docker --version
Docker version 20.10.24+dfsg1, build 297e128
devilapi@krasserserver:~$ docker pull mongo:6.0
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/images/create?fromImage=mongo&tag=6.0": dial unix /var/run/docker.sock: connect: permission denied
devilapi@krasserserver:~$ sudo docker pull mongo:6.0
6.0: Pulling from library/mongo
6414378b6477: Pull complete
e72d05fdcad4: Pull complete
e74f3b867e89: Pull complete
8dd75d4e021c: Pull complete
ee8b12b4d97c: Pull complete
686e3d514bb8: Pull complete
15eba7714a6a: Pull complete
77ad40620929: Pull complete
Digest: sha256:b250e27576511407ed59de2c308bbcbae34bf0d5d7df946e2ca7a618899b4cf0
Status: Downloaded newer image for mongo:6.0
docker.io/library/mongo:6.0