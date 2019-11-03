# cfresearch
A repository containing my research from CloudFlare's AntiDDoS, JSChallenge, Challenges, and WAF.
This was built for educational purposes such as learning how CloudFlare works, how to bypass CloudFlare, and how to prevent L7 DDoS Attacks.
> Contact Me: https://twitter.com/FuckBinary | https://t.me/trespassed | SmallDoink#0666

> Other Research: https://github.com/scaredos/l7research (Not Finished, Finishing CloudFlare research first)


## Layer7 DDoS Introduction
The common Layer7 DDoS attack are quite easy to pull off and perfect. Layer7 Attacks are far easier for the attacker to do since it can be done from a server with 1Gbps and a large proxy list. L7 attacks are more efficient in some cases for downing servers and making services unavailable. CloudFlare was built to stop these attacks, but like everything, it's not perfect. This was built to go over some of the known bypasses to bypass CloudFlare's WAF and DDoS prevention methods. CloudFlare is easy to bypass because their prevention methods are built in JavaScript and have easy math to create a token to access the site.
- > NOTE: JS Challenge is the MOST common type of cloudflare bypass, since kids don't know about cloudscraper. Use a more efficent way of parsing real from fake traffic to prevent these attacks

## Challenge (Captcha)
- This is used for UAM (Under Attack Mode) and Challenge
- Cloudflare uses a script to provide access to the website while this rule is on. This is also how UAM works for CloudFlare. 
`cdn-cgi/l/chk_captcha` Is the container for the script cloudflare uses. The script, written in PHP, gives the visitor temporary access to the website with cftokens and cookies. `/cdn-cgi/l/chk_captcha?s=&g-recaptcha-response=&cv_chal_result=%&cv_chal_fp=&bf_challenge_id=&bf_execution_time=&bf_result_hash=`is the full URI for the request to the captcha. By generating random data into the fields, you could bypass the Captcha page for usage in DDoS attacks, or for web crawling.
> To Patch: `Follow Steps at Bottom`

## JS Challenge
- This is the 5 second wait before loading webpage
- Cloudflare also implements a script to 'check' the 'browser' as an attempt to stop ddos attacks. `dn-cgi/l/chk_jschl` Is the container for the script cloudflare uses. The script, written in php, gives the visitor full access to the website if a JS Challenge is implemented using cookies and tokens. `cdn-cgi/l/chk_jschl?s=&jschl_vc=&pass=jschl_answer=`
`s=cloudflarecookie, jschl_vc=cookie, pass=randomint with last 5 rayid, jschl_answer=randomint`
> To Patch: `Follow Steps at Bottom`

## Raw Power (r/s)
With enough devices across enough locations with different ISPs and User Agents, you could shut down a website by bypassing the cache servers. Most likely, the site you are hitting doesn't recieve global traffic from 1k devices in each third world country. CloudFlare's cache servers in those countries will not have the website stored, therefore bypassing the caching process of CloudFlare, leaving the target webserver vulnerable. Using raw power along with the "Cache Bypass" method is very effective.
> To Patch: `Test your server weekly with CFBypass DDoS attacks. It will help the CloudFlare servers cache your content in small countries`

## Cloudscraper
- Cloudscraper is a JavaScript module that makes requests to cloudflare
- Cloudscraper is the most common way for bypasses now as it's easy to use since it's an external module. It follows all redirects which is needed for completely bypassing cloudflare.  Cloudscraper solves JSChallenges and reCaptcha challenges with ease and also supports Sucuri WAF Bypass, but that's for another day. Cloudscraper supports cookies and cache bypass, making DDoS more effective than normal. Cloudscraper also allows custom options on GET/POST request methods and reorders them accordingly. 

## PHP Application w/ CloudFlare Bypass
- The easiest target when attacking a website is the PHP pages. If you're server runs NGINX and PHP, then you're most likely vulnerable to this. If you get a `502 Bad Gateway` when your website gets hit, it's just a server error, nothing it wrong with your server. Either CloudFlare, or (most likely) your server's php queue is full. A simple `serivce php7.0-fpm restart` will fix it temporarily, but for a permanant fix, cache your PHP pages, setup limit_conn, proxy_cache, rate_limit, and other tools in NGINX's configuration for your php pages. Caching your PHP pages, then assuring the `cf-cache-status` is a `HIT`, will help your website take the attack and stop the 502 Gateway Errors.
> To Patch: `Cache PHP Pages | RateLimit | Limit Connections`

## Cache Bypass
- Cloudflare uses Edge servers to store cache and send requests to the webserver. If the server is caching pages such as HTML or PHP, or any content available via URI, you could send a `must-revalidate` or `max-age=0` or `no-cache no-store` query in your header to make the Cloudflare Edge server revalidate or reinstate the cache and GET a new file regardless if the file has changed or not.
- Another Cache Bypass method is random URI. Generating a random set of strings or integers and placing in the URL will bypass cached content (because it has never been loaded before). This is very effective as well, and with the script I uploaded (bypass.py), you can easily test this against your website if you have access to a server that will not ban you for DoS or have python install on your machine.
> To Patch Pt.1: `Remove must-revalidate or max-age from Cache-Control header and make CloudFlare reinstate the cache`
> To Patch Pt.2: `You must limit connections and make cloudflare cache the 404 page`

## Authentic Traffic
- To get allowed access to cloudflare regularly, you need a CloudFlare User ID cookie, labeled `__cfduid`. These can be obtained by setting the bot to allow cookies and request a cookie. Some application will require a `PHPSESSID` cookie to obtain access also.
> Patch: Limit Traffic To Webserver | RateLimiting | Limit Connections | Kill Connections

## Patching Layer7 Attacks
- Patching Layer7 Attacks can be hard, but with proper setup, can be very easy. First, start by ensuring visitor IP's. This can be done in both Apache and Nginx by using a cloudflare module to log the visitor ip, instead of cloudflare's IP. Second, setup ratelimiting for the website and install fail2ban to help enforce and jail the ratelimiting offenders. Third, block ASNs  that are known for DDoS Attacks, such as ChinaNet, and various Backbones of eastern countries such as China, Japan, and Russia. (Note: Blocking ASNs can Block some real user traffic) A very easy patch can be to cache the pages and force cloudflare to serve these files, rather than your webserver serving the files. 

- If you are getting targetted by a person individually, they are probably stupid. Look for patterns in their attack, such as proxies, spoofed IPs, UserAgents, IP Ranges, IP ASNs, etc. Usually, attackers do not randomize this data and leave this data available in the headers. Setup a firewall rule to block this common feature of the attacks, such as a common user agent, or common proxy server. This can take a huge load off your webserver and put it on cloudflare. Cloudflare's network is built for that kind of stress, so let cloudflare do its job.



