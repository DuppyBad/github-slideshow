## Hack The Box Writeup
# RouterSpace


### Step 1

Step 1 as always is to run an nmap scan
```
sudo nmap -sV -sC {targetip} -vv -oN nmapinital.txt
```
From this we see that port 22(SSH) and port 80(HTTP) are open, though there could be more ports as we only scanned the most common 1024. Putting a full -p- scan on in the background, we move to using Gobuster to try and enumerate the directories on the webserver.

This returns an error however, as the routerspace.htb domain is configured to stop these attempts at directory fuzzing, and returns a code ```200``` every time we attempt any directory that is invalid, instead pointing at a page giving a notice to our suspicious behaviour

Channging tack, we check the website manually, and find that upon clicking the download button, we can download the .apk of this routerspace app, and save it to our kali machine. .apk files are installable by android devices, but we can install them ourselves with the use of anbox, and adb, the android debug bridge. reading the anbox docs to ensure that it is installed correctly, we next use adb to install the .apk to anbox, letting us run the app on our linux system. ifconfig shows that the anbox has its own interface anbox0, pointing at some {ipaddress}. we can use adb to point the global proxy for the emulator for this at port 8080, and add the anbox0 ip to burpsuite proxy rules so we can intercept the request

Looking at the request we can see that the app sends a string containing ip to routerspace.htb, so the host must ensure that their /etc/hosts points routerspace.htb at {targetip} to ensure the app is functional. Messing with the string, we can use | to separate and start executing arbitrary code, from whoami. Being able to execute this code allows us to generate an ssh keypair, and echo the public key into the users authorized_keys, and then using ```chmod 700``` on the authorized_keys to mark it as acceptable by ssh. Finally we take our private key and mark it similarly, before using it to login the machine with ```ssh {username}@{targetip} -i {private.key}```

Inside our users home directory, we can ```cat user.txt``` for the first flag, though we will have to escalate our privilges in order to get the root.txt
We don't know our users password, so using sudo -l to look for SUID binaries is out of the question, but we can use linPEAS in order to enumerate possible exploits. Running it on our target, we see that it is vulnerable to a sudo exploit in this particular version of sudo, which you can look up once you obtain the sudo version yourself.

This CVE-XXX-XX-XX, can be leveraged to run as root, and after creating our exploit.sh, we can mark it as executable and run it, giving us a root shell. From here we simply have to ```cat /root/root.txt``` , and we have the final flag

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/DuppyBad/github-slideshow/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
