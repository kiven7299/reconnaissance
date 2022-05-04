common procedure
=======
1. find subdomain with amass

   ```sh
   #!/bin/bash
   amass enum -active -dir <output dir> -src -o <output filename> -d <main domain name> -blf <path to file providing blackisted domains>
   ```

   - extract hosts from amass-hosts-result

     ```sh
     #!/bin/bash
     cat amass_output/amass.txt | cut -d']' -f 2 | awk '{print $1}' | sort -u > hosts-amass.txt
     ```

     

2. Find valid active domain

   ```sh
   cat hosts-amass.txt | httpx -follow-redirects -ip -ports 80,443,8080,8081 -web-server -status-code -fc 400,404 -title -method -o httpx-hosts -x ALL
   ```



### Put it altogether

#### For common recon

Create new `.sh` file: 

- Take 1 paratemer which is the domain to recon.

```sh
#!/bin/bash
domain="$1"
echo "Reccon $domain"
mkdir -p $domain
echo "------- AMASS ---------"
echo "Find all subdomains"
amass enum -active -src -ip -o "$domain/amass-results" -d $domain
cat "$domain/amass-results" | cut -d']' -f 2 | awk '{print $1}' | sort -u > "$domain/hosts-amass"
echo 
echo "------- HTTPx ---------\n"
echo "HTTP probe to check if domain is active (remove result returning 404, 400)"
cat "$domain/hosts-amass" | httpx -follow-redirects -ip -ports 80,443,8080,8081 -web-server -status-code -fc 400,404 -title -method -x ALL -o "$domain/httpx-hosts" 
```



nmap
=======
main path: /usr/share/nmap/
scan openned service + find vulnerabilities

```powershell
nmap --script vulners -sV [--script-args mincvss=<arg_val>] <hostname|IP: target>
```



### Using  proxy with `proxychains`

- Find proxychains config files

  ```
  locate proxychains
  ```

  The main config file is `/etc/proxychains.conf`

- Edit config file to use socks5 proxy

  Add proxy config after `[ProxyLst]`

  ```
  [ProxyList]
  #socks5 proxy
  socks5 127.0.0.1 8888
  ```

- Using proxychain with `nmap`

  ```
  proxychains namp ...
  ```
  
  - Fix bug:
  
    ```
    yeah, nmap has some stupid (superfluous) code in it that tries to determine which network interface to use to send out packets, even if the user uses tcp options, so it could just unconditionally connect() and leave the decision to the OS.
    please file a bug against nmap.
    in the meantime there exist 3 options to workaround it on the proxychains side:
    
        1. do not use a dns name, but a raw ipv4 address
        2. or disable proxy_dns in the config
        3. or do not use proxychains at all but nmap's integrated options to use a SOCKS proxy.
    ```
  
    
  
  


### Usages

Update `vulners` script at  https://github.com/vulnersCom/nmap-vulners

Common use for redteam /pentest

```powershell
proxychains nmap -T4 -v -sS -sC -p- -oN <output file> <ip address range>
```



amass
=======
amass intel -- Discover targets for enumerations

amass enum -- Perform enumerations and network mapping

```powershell
common: amass enum --active -dir <output dir> -o <output filename> -d <main domain name>
```

amass viz -- Visualize enumeration results.

amass track -- Track differences between enumerations.

amass db -- Manipulate the Amass graph database.
	

# dirsearch

common use

```powershell
python dirsearch.py --header="Cookie: ..." -e sql,gz,bak,bk,php,txt,html,xml,asp,aspx,cgi,phtml,jsp,zip,rar,7z -r -R 3 --random-agents -b -t 5 --exclude-status=404,400
```

