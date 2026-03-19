_____________

# Guide LFI (Directory traversal) / Escalation to RCE through robbing id_rsa

In this exercise, we explore escalating to RCE via LFI by stealing the `id_rsa` private key.

<img width="805" height="240" alt="Captura de pantalla 2026-03-15 162058" src="https://github.com/user-attachments/assets/8ea9cf54-f0c9-40f3-9b98-004fd497cff6" />

First, as always, we confirm a Directory Traversal (LFI). By carefully analyzing `/etc/passwd`, we notice a listed user that uses `/bin/bash`.

<img width="1905" height="142" alt="Captura de pantalla 2026-03-15 162308" src="https://github.com/user-attachments/assets/d9f0dcce-80b8-4d2e-96bc-6bfbac8e6e3b" />

This is the key moment where a proper SSH service configuration protects us from an attacker accessing files such as `/home/user/.ssh/id_rsa`, as will be shown below.

<img width="1882" height="426" alt="Captura de pantalla 2026-03-15 162455" src="https://github.com/user-attachments/assets/707e0e7d-f9ca-4a45-8215-0fcae8a16cc4" />

Having the `id_rsa` listed, we can now establish an SSH connection as the leaked user. First, we will create a file named `id_rsa` with the content we obtained, and then use `chmod` to grant it the appropriate permissions:

`chmod 600 id_rsa`

<img width="694" height="143" alt="Captura de pantalla 2026-03-15 171109" src="https://github.com/user-attachments/assets/8f48a23c-cc66-4483-9317-90b2d7b41f27" />

When attempting the connection, we realize it requires a **passphrase**. To crack this security pass, we will use the **John the Ripper** tool. Since John the Ripper cannot work directly with the SSH key file format, we will use a utility called `ssh2john` to help us convert it to the correct format.

<img width="780" height="325" alt="Captura de pantalla 2026-03-15 172121" src="https://github.com/user-attachments/assets/7bdd873c-240b-4d11-9118-25dff8c36db0" />

With the cracked passphrase "**celtic**", we proceed to establish the SSH connection.

<img width="633" height="218" alt="Captura de pantalla 2026-03-15 172737" src="https://github.com/user-attachments/assets/fb81887c-c51f-48d3-a9f9-efd339f38710" />

The connection was successful; the system has been exploited and compromised. Due to a poorly sanitized SSH service configuration by the victim, we were allowed access to the user's `id_rsa`, gaining access to their account and demonstrating once again how an LFI can lead to RCE as pentesters.
