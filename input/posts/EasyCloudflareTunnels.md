Title: Quick temp fileshare
Published: 2/15/2026
Tags: [cloudflare] 
---

# Short filesharing across the net

Sometimes you just need to move a file - and nothing works apart from HTTP. So the easiest is to spin up an http server and make the server accessible via a cloudflare tunnel.

This would share the current folder via the python http server on cloudflare via a temp url. 
```
python3 -m http.server 8888 &
cloudflared tunnel --url http://localhost:8888 &
```

You get the URL out of cloudflare output:

```
026-02-15T17:35:57Z INF Thank you for trying Cloudflare Tunnel. Doing so, without a Cloudflare account, is a quick way to experiment and try it out. However, be aware that these account-less Tunnels have no uptime guarantee, are subject to the Cloudflare Online Services Terms of Use (https://www.cloudflare.com/website-terms/), and Cloudflare reserves the right to investigate your use of Tunnels for violations of such terms. If you intend to use Tunnels in production you should use a pre-created named tunnel by following: https://developers.cloudflare.com/cloudflare-one/connections/connect-apps
2026-02-15T17:35:57Z INF Requesting new quick Tunnel on trycloudflare.com...
2026-02-15T17:36:01Z INF +--------------------------------------------------------------------------------------------+
2026-02-15T17:36:01Z INF |  Your quick Tunnel has been created! Visit it at (it may take some time to be reachable):  |
2026-02-15T17:36:01Z INF |  https://antarctica-medicare-prairie-four.trycloudflare.com                                |
2026-02-15T17:36:01Z INF +--------------------------------------------------------------------------------------------+
```

Then on the target machine you can simply navigate to the url / filename you wanted to share. Done. Kill the commands. 
