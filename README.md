# haphash: Anti-scraper for haproxy

This is a simple anti-scraper solution for [haproxy](https://www.haproxy.org),
using a similar "hashcash" challenge as
[anubis](https://xeiaso.net/blog/2025/anubis/) uses. The goal is to be as
simple as possible, so this can be implemented alongside other haproxy rules to
control traffic.

## Overview

AI crawlers keep [breaking the
web](https://thelibre.news/foss-infrastructure-is-under-attack-by-ai-companies/).
I mostly have avoided this problem by having very lightweight pages, but lately
I've noticed some scrapers are being particularly obnoxious. Many solutions to
this problem involve adding another proxy component, but I'm already running
haproxy in most places, which is a perfectly fine reverse proxy and I don't
want to make things more complex if I can avoid it.

This uses a haproxy ["stick
table"](https://www.haproxy.com/blog/introduction-to-haproxy-stick-tables) to
store details of IP addresses. It is based on simply allowing IP addresses,
rather than cookies. As the IP address is stored in memory and there's no
cookie, this likely does not add to any GDPR obligations (this is not legal
advice).

It is expected this will be combined with haproxy IP based [rate
limiting](https://www.haproxy.com/blog/four-examples-of-haproxy-rate-limiting),
with the benefit that this doesn't add another component to the system.

If you want to try it out, my [contact](https://dgl.cx/contact) page is always
protected by it.

## The moving parts

[`challenge.html`](challenge.html) is the HTML served to clients, templated via
haproxy. (Because this is templated you can't just open it in your browser --
note the double percent signs.)

[`haproxy.conf`](haproxy.conf) is a haproxy config snippet that makes use of
this. It's expected you adjust this for your implementation. The "challenge"
backend is where the majority of the logic lives and should only need tiny
changes.

This is small:

```console
$ wc -l haproxy.conf challenge.html
      38 haproxy.conf
      94 challenge.html
     132 total
```

## Set-up

Copy [`challenge.html`](challenge.html) to `/etc/haproxy/challenge.html` (or
other suitable location).

From [haproxy.conf](haproxy.conf) add the `challenge` backend to your haproxy
configuration. Add the relevant lines from `frontend www` to your frontend
section.

To start with it is recommended you protect a single path for testing purposes.
Restarting haproxy will clear the stick table (configure
[peers](https://www.haproxy.com/documentation/haproxy-configuration-tutorials/proxying-essentials/custom-rules/stick-tables/#synchronize-stick-tables-across-peers)
to make the allowed IP addresses persist).

The difficulty is set in both the HTML and the haproxy config, it defaults to 4
(which is pretty fast).

## License

©[David Leadbeater](https://🖐.st) 2025; [0BSD](https://dgl.cx/0bsd), see
[COPYING](COPYING).

## Alternatives

* [haproxy enterprise](https://www.haproxy.com/documentation/haproxy-configuration-tutorials/security/enterprise-features/)
* [tedu's anticrawl](https://flak.tedunangst.com/post/anticrawl)
* [anubis](https://anubis.techaro.lol/)
* [go-away](https://git.gammaspectra.live/git/go-away)

## Credits

* [Xe Iaso](https://xeiaso.net/) for anubis.
