# cfresearch
This repository contains my research from CloudFlare's AntiDDoS, JS Challenge, Captcha Challenges, and CloudFlare WAF.
This was built for educational purposes such as learning how CloudFlare works, how to bypass CloudFlare challenges, and how to prevent attacks that are bypassing CloudFlare.
> Contact Me: [Telegram](https://t.me/trespassed) | The old information is at [OLD.md](https://github.com/scaredos/cfresearch/blob/master/OLD.md)

## New UAM Default
- CloudFlare has implemented a follow up after solving the JS Challenge for some users. CloudFlare now requires you to solve a captcha after solving the JS Challenge when the user has UAM enabled. For the first challenge, CloudFlare has introduced these two new items
- New Cookie
`cf_chl_1`: `id-of-challenge`

- New Form Data
`cf_ch_cp_return`: `id-goes-here|{"follow_up":"captcha"}`

- If there is not a `cf_ch_cp_return` item in the form data, there is no follow up Captcha. 


## JS Challenge
- This is used for UAM (Under Attack Mode)
- CloudFlare implements a "Browser Check" to generate a token for CloudFlare clearance. Upon solving the JS Challenge, the solver is given a `__cfuid` cookie stating the CloudFlare visitor ID, a `cf_clearance` cookie to freely load the website with another challenge, a `__cf_bm` cookie, and a `cf_chl_seq_challenge-number-here`. The URI is `?__cf_chl_jschl_tk__=GENERATED_TOKEN` and no longer `/cdn-cgi/`.

`r`: CloudFlare Analytics (Not required to solve challenge)

`jschl_vc`: Identity of CloudFlare JS challenge

`jschl_answer`: JavaScript challenge solution

`pass`: Used in solving JavaScript Challenge

`cf_ch_verify`: `plat`


### JS Challenge Generation
- To generate a JavaScript challenge through CloudFlare, the browser will send a POST request to `cdn-cgi/challenge-platform/generate/ov1/..CloudFlare RAY ID.../...CloudFlare Challenge Sequence...
- The CloudFlare Challenege Sequence is given by the challenge, sending the request will tell your browser to set the cookie. Failing the said challenge gives your browser a new cookie, the `cf_chal_retry=c` will tell CloudFlare to send you another challenge
- The CloudFlare Challenge Sequence number is then given to you in a cookie named `cf_chl_1=` that is equal to the sequence number. 

- An example URI looks like `https://example.com/cdn-cgi/challenge-platform/generate/ov1/599aceb96f6373f1/35fde4ae886667b/`.

## CloudFlare hCaptcha Challenge
The URI has not changed, but the POST data has. hCaptcha is the replacement for reCaptcha. hCaptcha uses the users input to train machine learning modles and neural networks, making the businesses that own the models pay the website owner.

`r`: CloudFlare Analytics (Not required to solve challenge)

`cf_catpcha_kind`: 'h' or 're' (hCaptcha or reCaptcha)

`id`: CloudFlare's ID of the request

`g-captcha-response`: Response of hCaptcha or reCaptcha

`h-captcha-response`: Response of hCaptcha or reCaptcha


## Captcha Challenge
- The new CloudFlare captcha challenge was introduced early this year in an attempt to stop the CloudFlare "bypasses".
- The new method is `POST` to `?__cf_chl_catpcha_tk__=GENERATED_TOKEN`. It hands a `cf_clearance` cookie, allowing the user to freely load the website without the captcha, and a `__cfuid` cookie stating the CloudFlare visitor ID. 


## Attacks through CloudFlare
- Most commonly, if you're website is being attacked while you have CloudFlare active, it's most likely a misconfiguration on your end. Do not ratelimit CloudFlare's IPs, but to ratelimit from your webserver, you can start by `restoring visitor's IPs` then apply a firewall rule to ratelimit HTTP requests. For starters, you should try caching the webpage. Do not cache any pages that require authentication such as a dashboard and a login or signup form. Caching the webpage will take load off of your server and put it onto the CloudFlare network, minimizing the network traffic to your server. Argo Smart Routing can increase the load time and also help with attacks. Argo Smart Routing reduces the requests to the web server.
- If you cannot cache the content on the webpage, try CloudFlare workers to deliver dynamic content to visitors or Railgun to decrease the size of the response for dynamic content. You can combine ESI with CloudFlare workers to cache some content on a dyanmic page, decreasing the load on your webserver. 

## Random URI in the attack
- When there is a random URI in the attack, it's an attempt to stress your webserver. You can cache the 404 page so CloudFlare will deliver the 404 content instead of your webserver. You can whitelist all URIs you use on your website in the CloudFlare firewall, then block everything else. Let's say you only use '/admin/' and '/home'. You can set a firewall rule "if not /home/ in URI" to block or captcha challenge the traffic. This is only a slight fix of course, if someone notices this, then they can do /home/random/random. 

## Patching the attacks
- CloudFlare can only do so much to block attacks. If the attack is large enough, there is probably nothing CloudFlare can do about this attack. With residential proxies, constantly resetting user-agents, and other randomized attributes that prevent CloudFlare from fingerprinting requests/devices and apply mitigation or an IP jail.
- Look for patterns in the attacks, such as proxies, user agents, ip ranges/asn, etc. Usually this data is randomized by using a proxy pool or list and generating pseudo-random user agents to abuse. If it is a specific ASN such as Digital Ocean being abused, send a Catpcha challenge to that ASN and it should drop the attack. Most attacks can only "bypass" the JavaScript challenge as solving the Captcha challenge requires a paid service or a unqiue bypass.
