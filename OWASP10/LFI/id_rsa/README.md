_____________

# Guide LFI (Directory traversal) / Escalation to RCE through robbing id_rsa

In this exercise, we explore escalating to RCE via LFI by stealing the `id_rsa` private key.

![[Captura de pantalla 2026-03-15 162058.png]]

First, as always, we confirm a Directory Traversal (LFI). By carefully analyzing `/etc/passwd`, we notice a listed user that uses `/bin/bash`.

![[Captura de pantalla 2026-03-15 162308.png]]

This is the key moment where a proper SSH service configuration protects us from an attacker accessing files such as `/home/user/.ssh/id_rsa`, as will be shown below.

![[Captura de pantalla 2026-03-15 162455.png]]

Having the `id_rsa` listed, we can now establish an SSH connection as the leaked user. First, we will create a file named `id_rsa` with the content we obtained, and then use `chmod` to grant it the appropriate permissions:

`chmod 600 id_rsa`

![[Captura de pantalla 2026-03-15 171109.png]]

When attempting the connection, we realize it requires a **passphrase**. To crack this security pass, we will use the **John the Ripper** tool. Since John the Ripper cannot work directly with the SSH key file format, we will use a utility called `ssh2john` to help us convert it to the correct format.

![[Captura de pantalla 2026-03-15 172121.png]]

With the cracked passphrase "**celtic**", we proceed to establish the SSH connection.

![[Captura de pantalla 2026-03-15 172737.png]]

The connection was successful; the system has been exploited and compromised. Due to a poorly sanitized SSH service configuration by the victim, we were allowed access to the user's `id_rsa`, gaining access to their account and demonstrating once again how an LFI can lead to RCE as pentesters.