--- 
layout: post
title: Bug Bounties Recon With Bash
subtitle:
date: 2020-04-26
author: D
header-img:
catalog: true
tags: [Recon,ShellBash,web hacking]
---

# Some Core Utils

- grep - search for patterns in files or stdin
- sed - edit the input stream
- awk - general purpose text-processing language
- cat - concatenate files
- find - list files recursively and apply filters
- sort - sort the lines from stdin
- uniq - remove duplicate lines from stdin
- xargs - run a command using each line from stdin as an argument
- tee - copy stdin to a file and to the screen

# Subshell Tricks

- <(cmd) - returns the output of `cmd` as a file descriptor
	- Handy if you want to diff the output of two commands...
	- diff<(cmd-one)<(cmd-two)
- $(cmd) - returns the output text of `cmd`
	- Handy if you want to store the command output in a variable
	- myvar=$(cmd)

# Enumerating Subdomains

- We could use external services
	- hackertarget.com
	- crt.sh
	- certspotter.com
- But it's nice to complement that with good-old brute force
- You will need:
	- A target
	- A wordlist
	- Bash :)

For example, the content of subdomain.txt as following:
```
admin
test
qa
dev
www
m
forums
invalid
blog
shop
sports
news
mail
ftp
```
Brute force subdomains. brute.sh as following:
```
#!/usr/bin/bash

domain=$1
while read sub; do
        if host "$sub.$domain" &> /dev/null; then
                echo "$sub.$domain";
        fi
done
```
Run on terminal:
```
cat subdomains.txt | ./brute.sh yahoo.com
```
Generate urls:
```
cat subdomains.txt | ./brute.sh yahoo.com | awk '{print "https://" $1}' > urls
```

# Dangling CNAMEs

cnames.sh as following:
```
#!/usr/bin/bash

domain=$1
while read sub; do
        cname=$(host -t CNAME $sub.$domain | grep 'an alias' | awk '{print $NF}')

        if [ -z "$cname" ]; then
                continue
        fi

        if ! host $cname &> /dev/null; then
                echo "$cname did not resolv ($sub.$domain)";
        fi
done
```
type command as following:
```
cat subdomains.txt | ./cnames.sh yahoo.com
```

# Fetch All the Things

fetch.sh as following:
```
#!/usr/bin/bash

mkdir -p out

while read url; do
        filename=$(echo $url | md5sum | awk '{print $1}')
        filename="out/$filename"
        echo "$filename $url" | tee -a index
        curl -sk -v "$url" &> $filename
done
```
Type command on terminal:
```
cat urls | ./fetch.sh
```

# Some Things To Grep For

- Titles
- Server headers
- Known `subdomain takeover` strings
- URLs(and then go and fetch the URLs!)
	- JavaScript files are nice (:
- Secrets
- Error message
- File upload forms
- Interesting Base64 encoded strings :) 

Example1:
```
cd out
grep -oiE '<title>(.*)</title>' *
```
Example2:
```
cd out
grep -hE '^> ' * | sort -u
```

# Speeding Things Up

- Pipes give you some parallelisation for free
	- It's not enough though, is it?
- xargs can run things in parallel...

sub.sh as following:
```
#!/usr/bin/bash

domain=$1

if host $domain &> /dev/null; then
        echo $domain
fi
```
parsub.sh as following:
```
#!/usr/bin/bash

domain=$1
while read sub; do
        echo $sub.$domain
done | xargs -n1 -P16 ./sub.sh
```
Terminal type as following:
```
cat subdomains.txt | ./parsub.sh yahoo.com
```

