Title: Receiving compressed data from an http(s) endpoint
Published: 20/03/2024 11:00:00
Tags: [CSharp, R, Python, JavaScript] 
---
With the amount of data that we are passing around across services, it is often beneficial to use compression on the data to reduce the transmission time. Modern platforms and algorithms are now very efficient at compressing regular data, particularly if that data is text or json data. 

If the developer of the endpoint has prepared their service for compression, the client must still indicate that they are ready to receive the compressed data. Luckily, most implementations of modern http clients in R, Python, JavaScript and Dotnet support compression / decompression and are seamless for the client. This means that you can set the compression headers on and simply benefit from compressed data being received. 

We can also check in the Content-Encoding header which compression was used. I've found that example.com is sending responses compressed with gzip.

## Python
```python
import requests

url = "http://example.com"  # Replace with the actual URL you want to request

# Specify the accepted encoding methods in the headers
headers = {
    'Accept-Encoding': 'gzip, br',
}

response = requests.get(url, headers=headers)
print(response.text)

# In case you want to see if it was compressed, you can check via the headers
#if 'Content-Encoding' in response.headers:
#    print(f"Response was compressed using: {response.headers['Content-Encoding']}")
#else:
#    print("Response was not compressed.")
```

## R
```R
library(httr)

# The URL to which you're sending the request
url <- "http://example.com"

# Setting the Accept-Encoding header
response <- GET(url, add_headers(`Accept-Encoding` = 'gzip, br'))

# The content of the response will be automatically decompressed by httr, so you can access it directly.
content(response, "text")
```

## C#
In C#, for some ungodly strange reason, the standard HTTP endpoint doesn't decompress for you automatically unless you add a decompression handler - see handler [HttpClientHandler.AutomaticDecompression](https://learn.microsoft.com/en-us/dotnet/api/system.net.http.httpclienthandler.automaticdecompression?view=net-8.0#system-net-http-httpclienthandler-automaticdecompression)

```Csharp
using System;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using System.Text;

class Program
{
    static async Task Main(string[] args)
    {
        HttpClientHandler handler = new HttpClientHandler();
        handler.AutomaticDecompression = System.Net.DecompressionMethods.GZip; //Adding automatic Decompression means that the accept headers are added automatically

        using (var client = new HttpClient(handler))
        {
            string url = "http://example.com";
            HttpResponseMessage response = await client.GetAsync(url);
            response.EnsureSuccessStatusCode();

            Console.WriteLine(await response.Content.ReadAsStringAsync());
		}
	}
}
```

## JavaScript

it is so easy that you don't even need to do anything else than setting the gzip: true for the support

```JS
const request = require('request');
const requestOptions = {
  url: 'http://example.com',
  gzip: true, // This is all that is required
};
request(requestOptions, (error, response, body) => {
  // Handle the response here
});
```
