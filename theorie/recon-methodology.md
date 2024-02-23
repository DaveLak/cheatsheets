# Definition

In order to successfully compromise a company in the context of a red team assessment, the first step is to identify the existing attack surface.
The process of information gathering is called Reconnaissance.
After a good Reconaissance, it is often possible to perform password spraying and powerfull phishing attacks.
In this context, information about the corporate structure, the internal IT landscape, the publicly accessible systems and the current employees are important.

## Determine corporate structure

In the first instance, it is necessary to identify the companies that are included in the scope of the assessment.
The following sources of information can be used to identify the relationships between different companies:

* https://www.crunchbase.com
* https://www.handelsregister.de
* https://www.northdata.de
* https://www.unternehmensregister.de

The search for company names, which according to northdata are related to each other, can be automated with the help of the tool `north-scraper` (see https://github.com/r1cksec/autorec/blob/master/scripts/north-scraper).

Depending on the target company it is necessary to remove some false positives afterwards, because northdata also shows past relations between companies.

Furthermore, a look at the company website or wikipedia often helps to better understand the structure of the company.
Information on subsidiaries and company locations may be listed there.

## Public infrastructure

The public infrastructure of a company includes domains and IP addresses.

### Get public IP address ranges

The easiest way to find public IP address ranges of a company is to search the RIPE database offline.
The database can be downloaded with the following command (unpacked about 4 GB):

```
wget ftp://ftp.ripe.net/ripe/dbase/ripe.db.gz
```

Afterwards it is possible to search the database with a simple `grep` command.
The name of the target company or acronyms provide a good starting point.
Since there is no established standard for specifying information in the database, the results should be completed or checked with further ressources like https://www.shodan.io or https://bgp.he.net/cc.

The search on bgp.he.net can be automated using the tool `get-netblocks` (see https://github.com/r1cksec/autorec/blob/master/scripts/get-netblocks):

```
get-netblocks '<companyName>'
```

There is additionally also the tool `spk` (see https://github.com/dhn/spk) which queries various databases like https://apps.db.ripe.net, https://wq.apnic.net or https://bgp.tools directly.

```
spk -silent -s <companyName>
```

Sometimes it is necessary to quickly get the name of the hoster for a given IP address.
The tool `get-whois-hoster` automatically queries lookip.net and provides the hoster for the system in question (see https://github.com/r1cksec/autorec/blob/master/scripts/get-whois-hoster):

```
get-whois-hoster <domainFile>
```

### Get rootdomains

In order to perform a subdomain enumeration, rootdomains are needed.
Rootdomains can be identified by simply searching for the company name using any search engine.
It is also possible that the rootdomains differ based on top-level domains.
However a domain with a completely different name could also belong to the target company.

One option to find rootdomains is the use of the Intel module from Amass (see https://github.com/OWASP/Amass).

The Intel module collects various root domains from different sources of the Open Source Intelligence (OSINT) sector.
These sources include, for example https://viewdns.info.

```
amass intel -d <domain> -whois
```

If the target company is using O365, root domains can also be identified via requesting the Autodiscover service.
The process has been automated by the tool `letItGo` (see https://github.com/SecurityRiskAdvisors/letitgo).

```
letItGo <domain>
```

However, the Intel module from amass can also be started with an ASN or an IP address or range.
A suitable ASN is often listed in the RIPE database under the `origin` field:

```
ip=$(host <domain> | awk -F 'has address' '{print $2}' | sed 's/ //g' | head -n 1); whois $ip | grep -i origin | awk -F ' ' '{print $2}'
```

Additionally, it is often useful to enumerate different top-level domains for already known root domains.
Again this process can be automated using the tool `tld-discovery` (https://github.com/r1cksec/autorec/blob/master/scripts/tld-discovery):

```
python3 tld-discovery.py <domain>
```

If IP address ranges of the target company are already known, it is also possible to examine the Subject Alternate Name (SAN) value of the existing TLS certificates.
The IP range in question could for example be scanned with nmap (see https://github.com/nmap/nmap):

```
nmap --script /usr/share/nmap/scripts/ssl-cert.nse <rhost> -p 443 
```

The scan process and extraction of SAN values can be automated using the tool `get-cert-domains-from-ip-range` (see https://github.com/r1cksec/autorec/blob/master/scripts/get-cert-domains-from-ip-range):

```
python3 get-cert-domains-from-ip-range.py <ipRange>
```

Of course, it is worthwhile to perform a reverse dns lookup for all known IP addresses.

```
nmap -sL <ipRange> | awk -F 'Nmap scan report for ' '{print $2}' | grep '('
```

Furthermore, it is also possible to find other root domains by searching for the start date of certificates, because many certificates are renewed automatically at a fixed time.
The search can be performed using https://search.censys.io:

```
parsed.validity.start:<year>-<month>-<day>T<hour>\:<minutes>\:<second>Z
```

Some Certificate Authority like DigiCert use 00:00:00 as start time.
Therefore, when a CA does not include the exact issuing time to certificates, the certificates issued close in time can be discovered on https://crt.sh by their positions in the logs.

Moreover, it is possible to crawl the website of a company with a web spider.
Among the extracted domains and subdomains, further root domains can be identified in this way.
The tool `hakrawler` allows to extract all URLs from a given website (see https://github.com/hakluke/hakrawler):

```
cat "<urlFile>" | hakrawler -s -u -d 1
```

### Classify rootdomains

The list of domains resulting from this procedure must then be sorted and classified.

In the most cases websites can be assigned to a specific company based on the copyright or the imprint.
In this way a long list of root domains can be sorted and classified accordingly.
Again this process can be automated using the tool `get-page-owner` (see https://github.com/r1cksec/autorec/blob/master/scripts/get-page-owner):

```
python3 get-page-owner <domainFile>
```

Alternatively the copyright can be used to find further root domains.
A simple Google search with `"copyright"` is often sufficient to list other root domains.

Another way to assign rootdomains to a target company is the comparision of favicons.
The tool favfreak is suitable for this purpsoe (see https://github.com/devanshbatham/FavFreak).

Favfreak calculates the MD5 hash of a favicon and then groups equal results:

```
cat <urlFile> | favfreak
```

It is also possible to sort the domains using screenshots of websites using the tool `aquatone`.
Aquatone takes screenshots and then groups equal results (see https://github.com/michenriksen/aquatone):

```
cat <rhostFile> | aquatone -out <outputDirectory>
```

Alternatively rootdomains can also be classified based on the `title` tag.
The tool `get-title` automates this process (see https://github.com/r1cksec/autorec/blob/master/scripts/get-title):

### Subdomain enumeration

Some subdomains are easy to guess.
The tool `massdns` is particularly well suited for this, since several DNS servers are used and thus many queries can be parallelized (see https://github.com/blechschmidt/massdns):

```
cat <subdomainFile> | awk -F " " '{print $1".<domain>"}' | massdns -r <dnsResolversFile> -t A -o S -w <outputFile>
```

Subdomain wordlists can be obtained here https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS.

Additionally, subdomains can be enumerated using public OSINT resources like https://crt.sh or https://dnsdumpster.com. 

Again the tool `amass` automates this process.

```
amass enum -passive -d <domain> -src
```

Alternatively, there are also tools like `subfinder` (see https://github.com/projectdiscovery/subfinder):

```
subfinder -d <domain>
```

## Internal informations

Internal informations are necessary to send trust-building phishing messages or to perform targeted password spraying attacks.

### Internal domain

`Nmap` can be used to scan available systems and search for NTLM authentications (example OWA, Skype for Business, Lyncdiscover etc):

```
nmap -p 80,443 --script http-ntlm-info --script-args http-ntlm-info.root=/root/ <target>
```

To cover a larger set of web paths it is also possible to use the tool `ntlmrecon` (see https://github.com/pwnfoo/NTLMRecon). 

```
ntlmrecon --input <url>
```

The internal domain thus determined can then be used for a Google search for `"internal.domain"` to obtain further interesting information about the internal infrastructure.

### Metadata

Often interesting information are hidden in metadata of published documents like PDF, Excel or Word files.

Examining the `producer` and `author` fields of different files, will likely reveal conclusions about the internal Active Directory user syntax.
Using `metagoofil` several documents can be downloaded in the first step (see https://github.com/kurobeats/metagoofil):

```
metagoofil -d <ipOrDomain> -t pdf -l 100 -n 25 -o <outputDir> -w
```

Afterwards the metadata can be extracted using `exiftool` (see https://github.com/exiftool/exiftool):

```
exiftool * | grep -i "Producer\|Author" | sort -u
```

The metadata can also contain other informations like domains or e-mail addresses:

```
strings * | grep -i "@"
```

Searching for PDF documents and extracting the metadata can also be done at once with the script `get-pdf-metadata` (see https://github.com/r1cksec/corptrace/blob/master/ressources/modules/startpage_get_pdf_metadata.py)

```
get-pdf-Metadata <domain>
```

### DNS records

It is also advisable to check all DNS records of a domain.
In this way it is possible to find additional information about internally used software.
The tool `get-dns-records` will retrieve all kind of DNS records (see https://github.com/r1cksec/autorec/blob/master/scripts/get-dns-records):

```
bash get-dns-records.sh <domain>
```

### Github

If the target company runs one or more Github repositories, it is often worthwhile to search this repositories for additional informations as well as credentials.
The following tools facilitate this search:

* https://github.com/r1cksec/autorec/blob/master/scripts/get-github-repos
* https://github.com/r1cksec/autorec/blob/master/scripts/grep-through-commits
* https://github.com/r1cksec/autorec/blob/master/scripts/get-grep-app
* https://github.com/zricethezav/gitleaks
* https://github.com/techjacker/repo-security-scanner
* https://github.com/tillson/git-hound
* https://github.com/trufflesecurity/truffleHog

## Employees and E-mail addresses

In many cases, attackers are interested in e-mail addresses of a target company.
OSINT resources resources provide a good starting point.
The following two services offers a good collection of e-mail addresses for a given domain:

* https://phonebook.cz
* https://www.skymem.info

It is also possible to find e-mail addresses using search engines.
The tool `EmailAll` combines the scrape of OSINT resources and the use of search engines (see https://github.com/Taonn/EmailAll).

```
python3 emailall.py --domain <domain> run
```

Once the syntax of the target company is known, e-mail addresses or user names can also be derived based on employee names.
A suitable list of employees can be obatined using social media platforms like Xing or LinkedIn.

Xing offers the advantage that all employees of a company can be listed by sending a few HTTP GET requests.
After pressing the "more Employes" button, it is possible to intercept the request and change the value of the variable "first" to a new value (for example "1000").

```
POST /xing-one/api HTTP/1.1
Host: www.xing.com
[...]
{
    "operationName":"Employees",
    "variables":{
        "consumer":"",
        "includeTotalQuery":false,
        "id":"targetCompanyIdNumber",
        "first":1000,
[...]
```

LinkedIn is more restrictive regarding lists of employees.
But it is still possible to collect employees using Google Dorks.

```
intitle:"companyName" inurl:"linkedin.com/in/" site:linkedin.com
```

Alternativly, you can pay for rekruting solutions like PhantomBuster (see https://phantombuster.com) and thus obtain additional information.

Another source to get more email addresses are known database leaks.
Besides a list of mail addresses and maybe even passwords, this source can also be used to discover new root domains.
Once a new rootdomain has been found, the enumeration of subdomains and internal informations can be repeated.

