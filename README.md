# TryHackMe-LianYu
A complete walkthrough for TryHackMe's Lian_Yu CTF machine. Includes web enumeration, FTP exploitation, steganography, and privilege escalation. All flags and credentials are documented.

## Introduction
LianYu is a beginner-friendly CTF machine from TryHackMe based on the Arrowverse. The main objective is to capture two flags: user.txt and root.txt.

Throughout this machine, we will learn:
- Web enumeration
- FTP exploitation
- Steganography techniques
- Privilege escalation

---

## Step 1: Initial Enumeration with Nmap

I started by scanning the target machine:

```nmap -sC -sV 10.82.164.105```

<img width="803" height="523" alt="nmap" src="https://github.com/user-attachments/assets/c3824e0d-08b6-472a-9c12-367f195c1580" />



Result:
- Port 21 - FTP
- Port 22 - SSH
- Port 80 - HTTP

---

## Step 2: Web Directory Enumeration with Gobuster

Scan main web dirictory:

```gobuster dir -u http://10.82.164.105 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 10```

<img width="970" height="294" alt="island" src="https://github.com/user-attachments/assets/d7a01d31-ede7-4c11-bdbc-12db7f44a2db" />


Found:
/island 

You will find /island. Visiting http://10.82.164.105/island/ reveals a hidden message. However, the "Code Word" isn't immediately visible on the page, suggesting we need to inspect the page source and found "vigilante"

<img width="624" height="251" alt="vigilante" src="https://github.com/user-attachments/assets/1dddc3a7-36cf-4ff7-8896-86039499a057" />

---

Scan inside /island:

```gobuster dir -u http://10.82.164.105/island -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 10```

<img width="962" height="295" alt="2100" src="https://github.com/user-attachments/assets/19e09fcc-fadb-46d8-95e8-3dbe95a9acc6" />


<img width="1224" height="112" alt="answ 2100" src="https://github.com/user-attachments/assets/2d3bbaad-b18a-42c5-b533-d2cff9e5fc35" />


Found:
/2100

Visiting http://10.82.164.105/island/2100/. While the video on the page is unavailable, the "inspector" tool reveals a crucial piece of information hidden in a comment. 

<img width="1081" height="734" alt="oliver queen" src="https://github.com/user-attachments/assets/6c6ce33b-f19e-4db5-87dd-370f7f7a1b0c" />

---

Scan inside /2100 for .ticket files:

```gobuster dir -u http://10.82.164.105/island/2100 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x .ticket -t 10```

<img width="969" height="308" alt="green" src="https://github.com/user-attachments/assets/ed00bb8f-dc1a-4b9d-add7-342093906cd8" />

Found:
green_arrow.ticket

<img width="1223" height="102" alt="answ green" src="https://github.com/user-attachments/assets/bbfce8bc-49e0-4943-aab0-1134955e8372" />

---

## Step 3: Decoding the FTP Password

I accessed the green_arrow.ticket file using the command curl http://10.82.164.105/island/2100/green_arrow.ticket. The output was an encoded string: RTy8yhBQdscX.

<img width="1029" height="249" alt="token green" src="https://github.com/user-attachments/assets/9147fb80-1e63-478d-8fd5-74756e5cb103" />

Based on research, this string was encoded using Base58. I decoded it using the Python command ```python3 -c "import base58; print(base58.b58decode('RTy8yhBQdscX').decode())". The result was !#th3h00d``` which turned out to be the FTP password.

<img width="694" height="64" alt="python" src="https://github.com/user-attachments/assets/7cdd7072-b8f6-4e19-ade8-bafe46f30eb1" />

<img width="1215" height="98" alt="answ ftp" src="https://github.com/user-attachments/assets/d1c5a9e9-8b0d-44d0-86fd-55776c5650f0" />


---

## Step 4: Logging into FTP

Using the code word "vigilante" as the username and !#th3h00d as the password, I logged into the FTP server with the command ```ftp 10.82.164.105```. Once inside, I listed all files using the ls command. I saw several image files: 

- aa.jpg
- Leave_me_alone.png
- Queen's_Gambit.png

I downloaded all of them using the get command for each file.

<img width="1004" height="678" alt="ftp" src="https://github.com/user-attachments/assets/a4115b55-6609-464d-8307-319e1228fad6" />


When I try open the file one by one, I noticed that ```Leave_me_along.png``` cannot be open.

<img width="695" height="886" alt="image 1" src="https://github.com/user-attachments/assets/7d93999c-180a-4bbb-932d-e78f2e4eec4a" />


<img width="716" height="805" alt="image 2" src="https://github.com/user-attachments/assets/8069e2a4-f5b3-4203-9284-dc79503e6668" />


<img width="1151" height="717" alt="image 3" src="https://github.com/user-attachments/assets/23aefba0-5fae-4530-8fc4-67aa2368f05e" />


---

## Step 5: Repairing the Corrupted PNG Header

I examined the downloaded files and ran the file ```Leave_me_alone.png``` command. The output showed data instead of PNG image data, indicating that the file header was corrupted. I needed to repair it.

I used the command ```hexedit Leave_me_alone.png``` to open the file in a hex editor. The correct PNG header should be ```89 50 4E 47 OD 01 1A 0A```. I replaced the corrupted bytes with these correct values. After saving, I ran file Leave_me_alone.png again and this time it showed PNG image data.

<img width="933" height="614" alt="PNG" src="https://github.com/user-attachments/assets/28be6b97-1bbf-4889-a4a2-4c546fdb61dd" />


<img width="454" height="365" alt="PASSWORD" src="https://github.com/user-attachments/assets/a7d7235a-06dd-4ea6-b918-fc94e47ed0b8" />

---

## Step 6: Extracting Hidden Data with Steghide

Now that I had the password, I used it to extract hidden data from aa.jpg. I ran the command ```steghide extract -sf aa.jpg -p password```. This successfully extracted a file named ```ss.zip```. I then unzipped ss.zip using the command ```unzip ss.zip```. Inside, I found two files: passwd.txt and shado.

<img width="443" height="163" alt="unzipp" src="https://github.com/user-attachments/assets/73e7e149-8900-4b02-adb4-c96c961070d9" />

---

## Step 7: Obtaining the SSH Password

I read the contents of the shado file using the command ```cat shado```. The output was ```M3tahuman```, which was the SSH password I needed.

<img width="262" height="64" alt="shado" src="https://github.com/user-attachments/assets/ca76ab07-c3e1-4405-a34b-6d5f5c3ce526" />

---

## Step 8: Logging into SSH

With the username slade and the password ```M3tahuman```, I logged into the SSH server using the command ```ssh slade@10.82.164.105```. I was successfully connected to the target machine.

<img width="585" height="357" alt="welcoem lianyu" src="https://github.com/user-attachments/assets/aa752abd-93d4-4482-9e01-e6997b105674" />


---

## Step 9: Capturing the User Flag

Once inside the SSH session, I listed the files using the ```ls``` command. I saw the file user.txt. I read its contents using ```cat user.txt``` and obtained the first flag: ```THM{P30P7E_K33P_53CRET5__COMPUT3R5_DON'T}```.

<img width="585" height="85" alt="user txt" src="https://github.com/user-attachments/assets/87cd9611-cd5c-4df4-877a-df8c1a8c874f" />

---

## Step 10: Privilege Escalation to Root

I checked what sudo permissions the user slade had by running the command ```sudo -l```. I was prompted for a password, so I entered ```M3tahuman```. The output showed that slade could run ```/usr/bin/pkexec``` as root with a password. I then escalated to root using the command ```sudo pkexec/bin/sh```. This gave me a root shell, indicated by the ```#prompt```. With root access, I navigated to the root directory and read the root.txt file using the command ```cat /root/root.txt```. The final flag was: ```THM{MY_WORD_I5_MY_BOND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}```.

<img width="884" height="381" alt="root" src="https://github.com/user-attachments/assets/722bdc80-1b3a-4cd1-89b7-df99d57df530" />

<img width="1226" height="105" alt="answ root" src="https://github.com/user-attachments/assets/9950ff8b-e6bc-4df1-b0c3-ddbaf72c2bbd" />

<img width="1003" height="588" alt="hoorayyy" src="https://github.com/user-attachments/assets/d6d15d06-42da-4914-9ac8-e04c478a548c" />




















