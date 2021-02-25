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
   cat hosts-amass.txt | httpx -follow-redirects -ip -ports 80,443,8080,8081 -web-server -status-code -fc 400,404 -title -method -o httpx-hosts 
   ```



### Put it altogether

#### For common recon

Create new `.sh` file: 

- Take 1 paratemer which is the domain to recon.

```sh
#!/bin/bash
domain="$1"
mkdir -p $domain
echo "------- AMASS ---------"
echo "Find add subdomains"
amass enum -active -dir "$domain" -src -ip -o amass-results -d $domain
cat "$doamin/amass-results" | cut -d']' -f 2 | awk '{print $1}' | sort -u > "$domain/hosts-amass"
echo 
echo "------- HTTPx ---------\n"
echo "HTTP probe to check if domain is active (remove result returning 404, 400)"
cat "$domain/hosts-amass" | httpx -follow-redirects -ip -ports 80,443,8080,8081 -web-server -status-code -fc 400,404 -title -method -o "$domain/httpx-hosts" 
```



nmap
=======
main path: /usr/share/nmap/
scan openned service + find vulnerabilities

```powershell
nmap --script vulners -sV [--script-args mincvss=<arg_val>] <hostname|IP: target>
```





amass
=======
amass intel -- Discover targets for enumerations

amass enum -- Perform enumerations and network mapping

```powershell
common: amass enum --active -dir <output dir> -o <output filename> -d <main domain name>
```

amass viz -- Visualize enumeration results

amass track -- Track differences between enumerations

amass db -- Manipulate the Amass graph database
	