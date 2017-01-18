# API Connectivity Issues

Appboy’s API endpoint use a CDN that routes traffic to the closest POP based on DNS information. If you are having issues connecting or notice that you are connecting to a POP that is not efficient please make sure to use your providers DNS servers or DNS servers that are setup in the same data center as your server and have proper IP location meta information associated with them.

We have noticed that a handful of firewalls attempt to modify or secure HTTPS/TLS traffic which interferes with connections to Appboy’s API endpoint. If your servers are behind any sort of physical firewall please disable any HTTPS/TLS acceleration or modifications that the firewall(s) and/or router(s) are performing. Additionally you can whitelist outbound traffic to our CDN providers (Fastly.com) to see if that resolves the issue.

Occasionally iptables setups that filter on syn/ack/rst packets can also cause issues, so if you are using iptables on your host you could also whitelist outbound traffic to our CDN providers (Fastly.com) to see if that resolves the issue.

If you are still having network issues with connecting to Appboy’s API Endpoint please provide a MTR MTR Test and the results from Fastly Debug while experiencing an issue and submit that with your support request.
The test results must be obtained from a server that is having issues connecting to Appboy’s API endpoint and not from a development laptop. A network capture (tcpdump or pcap file) will also be helpful if it can be obtained.

WHITELISTING APPBOY’S API ENDPOINT IP RANGES

To whitelist Appboy’s API endpoint through your firewall, our CDN provides access to the list of assigned IP ranges via a JSON dump. You can access that list at this [URL](https://api.fastly.com/public-ip-list).
