# esviewer.js

**tl;dr: This lets you search an Elasticsearch index without having to install anything but Elasticsearch. You just point your browser to an HTML file and off you go. It's not secure, not one bit.**

This tool addresses a frequent issue I have. I usually have a dataset in Elasticsearch and I want to search through it. No Kibana, no nothing. I would usually just read the JSON of `http://localhost:9200/index/_search?q=my_query`, but that's not a terribly friendly interface. Also, I hate dependencies, so I came up with this.

There is a single HTML file that talks to Elasticsearch's REST API, that's pretty much it. There is just one caveat: modern browsers don't let websites talk to APIs that have a different hostname (or even a different port of the same hostname), unless said server is set up with [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

There are at least two ways you can CORSify your Elasticsearch server. You can turn on CORS in your ES settings (usually `/etc/elasticsearch/elasticsearch.yml` on Unix systems):

```
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-methods: GET, POST
```

Or if you're using something like nginx, you can put a proxy in front of your ES cluster and set it to add CORS while it's passing requests through (you'll also move your cluster to a different port by doing this).

```
server {
    listen 8080;
    location / {

    add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
    proxy_pass http://localhost:9200;
    }
}
```

**You should never use either of these solutions in production.** This is just a toy script that I use on my laptop to browse my data.

I've also used this on a VPS and because I usually don't want to deal with nginx, I use the first method and then allow my home IP to access ports 80 and 9200 of my remove server (those being the webserver and Elasticsearch, respectively). I use Ubuntu, so I can tweak [ufw](https://en.wikipedia.org/wiki/Uncomplicated_Firewall) in the following way. Note that using SSH tunnels is probably better.

```
sudo ufw allow 22;
sudo ufw allow from my.ip.address to any port 9200;
sudo ufw allow from my.ip.address to any port 80;
sudo ufw enable;
```

In case you go for the nginx option, you can filter IPs in the CORS rule above, so you get both CORS and basic security in one, whereas here I have to modify both ES and ufw. It's a matter of preference.

I really enjoy using this script, because it requires very little setup - and I usually have CORS enabled locally anyway, so I just launch the HTML file and I'm done. The file has a few options you can modify.

```
const endpoint = 'http://possibly.a.remote.address:9200/_search'; // set your Elasticsearch endpoint
const pglen = 5; // set number of items per page
const render_keys = ['restrict', 'to', 'just', 'some', 'keys']
```

As always, it comes with no guarantees, pull requests and issue submissions are welcome. I'll leave you with a screenshot:

![screenshot](https://raw.githubusercontent.com/kokes/esviewer.js/master/screenshot.png)
