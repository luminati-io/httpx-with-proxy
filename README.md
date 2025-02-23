# Using Proxies with HTTPX

[![Promo](https://github.com/luminati-io/Rotating-Residential-Proxies/blob/main/50%25%20off%20promo.png)](https://brightdata.com/proxy-types/residential-proxies) 

This guide explains how to use proxies with HTTPX, with examples for unauthenticated, authenticated, rotating, and fallback proxies.

## Using Unauthenticated Proxies

With an unauthenticated proxy, we’re not using a username or password, and all requests go to a `proxy_url`. Below is a code snippet that uses an unauthenticated proxy:

```python
import httpx

proxy_url = "http://localhost:8030"


with httpx.Client(proxy=proxy_url) as client:
    ip_info = client.get("https://geo.brdtest.com/mygeo.json")
    print(ip_info.text)
```

## Using Authenticated Proxies

Authenticated proxies require a username and password. Once you submit correct credentials, you are granted connection to the proxy.

With authentication, the `proxy_url` looks like this: `http://<username>:<password>@<proxy_url>:<port_number>`. The following example demonstrates how to construct the user portion of the authentication string using both `zone` and `username`. It also utilizes [Bright Data's datacenter proxies](https://brightdata.com/proxy-types/datacenter-proxies) as the base connection.

```python
import httpx

username = "your-username"
zone = "your-zone-name"
password = "your-password"

proxy_url = "http://brd-customer-{username}-zone-{zone}:{password}@brd.superproxy.io:33335"

ip_info = httpx.get("https://geo.brdtest.com/mygeo.json", proxy=proxy_url)

print(ip_info.text)
```

Here is the breakdown of the above code:

- We start with creating config variables: `username`, `zone`, and `password`.
- We use those to create our `proxy_url`: `"http://brd-customer-{username}-zone-{zone}:{password}@brd.superproxy.io:33335"`.
- We send a request to the API to retrieve general information about our proxy connection.

The response should look like this.

```json
{"country":"US","asn":{"asnum":20473,"org_name":"AS-VULTR"},"geo":{"city":"","region":"","region_name":"","postal_code":"","latitude":37.751,"longitude":-97.822,"tz":"America/Chicago"}}

```

## Using Rotating Proxies

Rotating proxies means creating a list of proxies and choosing from them randomly. The code snippet below creates a list of `countries`, then uses `random.choice()` on each request to pick a random country from the list. Our `proxy_url` gets formatted to fit the country. The list of [rotating proxies](https://brightdata.com/solutions/rotating-proxies) in the code is from Bright Data.

```python
import httpx
import asyncio
import random


countries = ["us", "gb", "au", "ca"]
username = "your-username"
proxy_url = "brd.superproxy.io:33335"

datacenter_zone = "your-zone"
datacenter_pass = "your-password"


for random_proxy in countries:
    print("----------connection info-------------")
    datacenter_proxy = f"http://brd-customer-{username}-zone-{datacenter_zone}-country-{random.choice(countries)}:{datacenter_pass}@{proxy_url}"

    ip_info = httpx.get("https://geo.brdtest.com/mygeo.json", proxy=datacenter_proxy)

    print(ip_info.text)
```

The code is very similar to the one for authenticated proxies. The key differences are:

- We create an array of countries: `["us", "gb", "au", "ca"]`.
- We make multiple requsts instead of a single one. For each request, we use `random.choice(countries)` to choose a random country every time we create the `proxy_url`.

## Creating a Fallback Proxy Connection

All examples above use datacenter and free proxies. The former often get blocked by websites, the latter isn't very reliable. For all this to work, there should be a fallback to residential proxies.

To do that, let's create a function called `safe_get()`. When we call it, we first try to get the url using a datacenter connection. When this fails, we _fall back_ to our residential connection.

```python
import httpx
from bs4 import BeautifulSoup
import asyncio


country = "us"
username = "your-username"
proxy_url = "brd.superproxy.io:33335"

datacenter_zone = "datacenter_proxy1"
datacenter_pass = "datacenter-password"

residential_zone = "residential_proxy1"
residential_pass = "residential-password"

cert_path = "/home/path/to/brightdata_proxy_ca/New SSL certifcate - MUST BE USED WITH PORT 33335/BrightData SSL certificate (port 33335).crt"


datacenter_proxy = f"http://brd-customer-{username}-zone-{datacenter_zone}-country-{country}:{datacenter_pass}@{proxy_url}"

residential_proxy = f"http://brd-customer-{username}-zone-{residential_zone}-country-{country}:{residential_pass}@{proxy_url}"

async def safe_get(url: str):
    async with httpx.AsyncClient(proxy=datacenter_proxy) as client:
        print("trying with datacenter")
        response = await client.get(url)
        if response.status_code == 200:
            soup = BeautifulSoup(response.text, "html.parser")
            if not soup.select_one("form[action='/errors/validateCaptcha']"):
                print("response successful")
                return response
    print("response failed")
    async with httpx.AsyncClient(proxy=residential_proxy, verify=cert_path) as client:
        print("trying with residential")
        response = await client.get(url)
        print("response successful")
        return response

async def main():
    url = "https://www.amazon.com"
    response = await safe_get(url)
    with open("out.html", "w") as file:
        file.write(response.text)

asyncio.run(main())
```

Here is the code breakdown:

- We now have two sets of configuration variables: one for our datacenter connection and another for our residential connection.
- This time, we use an `AsyncClient()` session to introduce some of the more advanced functionality of HTTPX.
- First, we attempt to make our request with the `datacenter_proxy`.
- If we fail to get a proper response, we retry the request using the `residential_proxy`. Also note the `verify` flag in the code. When using Bright Data's residential proxies, you need to download and use the [SSL certificate](https://docs.brightdata.com/general/account/ssl-certificate).
- Once we’ve got a solid response, we write the page to an HTML file. We can open this page up in a browser and see what the proxy actually accessed and sent back to us.

After running the code above your output and resulting HTML file should look like this.

```
trying with datacenter
response failed
trying with residential
response successful
```

![Screenshot of the Amazon homepage](https://github.com/luminati-io/httpx-with-proxy/blob/main/Images/image.png)

## Conclusion

When you combine HTTPX with [Bright Data's top-tier proxy services](https://brightdata.com/proxy-types), you get a private, efficient, and reliable way to scrape the web. Start your free trial with Bright Data’s proxies today!
