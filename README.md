# redi
Automated redirector setup compatible with HTTP RATs (CobaltStrike Beacon, meterpreter, etc), and CobaltStrike DNS Beacon. The script can either set up nginx reverse proxy, or DNS proxy/forwarder using dnsmasq. If HTTPS was selected, it will automatically setup letsencrypt certbot and obtain valid letsencrypt SSL certificates for your redirector domain name, and start nginx using the generated configuration. 

The reverse proxy method allows for setting up more than one redirector each with its own valid certificate all pointing to the same CobaltStrike HTTPS stager/listener even if CobaltStrike is using self-signed/untrusted certificate.

It is also possible to modify the nginx configuration generated by the script to add extra features. For instance, you can choose to proxy only the traffic that matches your CobaltStrike malleable c2 profile and serve a static page or a different proxy otherwise. The configuration modifies the user-agent header to add the original source IP so that you can see it directly in your CobaltStrike web logs (see picture below). 

With some configuration tweaking you can even SSL offload beacon's HTTPS traffic to a teamserver's HTTP listener!.


## Advantages
- Auto SSL setup for HTTPS using letsencrypt certbot.
- Auto nginx and dnsmasq configuration.
- Access logs for HTTP redirector (default nginx logs).
- Fine control over HTTP headers by customizing nginx configuration. 
- Allows for multiple valid HTTPS redirectors setup
- Adds original source ip to user-agent header for easy tracking. 
- No port bending needed.
- SSL offloading possible, so you can have SSL beacon delivered to a backend HTTP listener !! (needs special setup).

![alt tag](https://github.com/taherio/random/raw/38641d74f0628a26142b121e62b393e96cac156a/image.png)

## How to use

```
git clone https://github.com/taherio/redi.git
cd redi
chmod u+x redi.sh
./redi.sh <redirector domain> <teamserver ip/domain> <http/https>

#alternatively for dns redirector
./redi.sh <teamserver ip> dns
```

### Example For setting up HTTPS redirector with multiple domains 
```
./redi.sh first.myredirector.ca,second.myredirector.ca,third.myredirector.ca myteamserver.com https
```

### Example For setting up HTTPS redirector
```
./redi.sh myredirector.ca myteamserver.com https
```

### Example For setting up DNS redirector
```
./redi.sh 192.0.0.1 dns
```

## Sample of HTTPS config generated
```
server {
    listen 443 ssl;
    server_name myredirector.ca;

    ssl on;
    ssl_certificate 	/etc/letsencrypt/live/myredirector.ca/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myredirector.ca/privkey.pem;

    location / {
        proxy_pass         https://myteamserver.com:443/;
        proxy_redirect     off;
        proxy_set_header   Host             $host;
        proxy_set_header   "User-Agent" "${http_user_agent} - Original IP ${remote_addr}";
    }
}
```
