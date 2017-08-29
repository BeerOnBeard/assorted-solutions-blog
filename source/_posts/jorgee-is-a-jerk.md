---
title: Jorgee Is a Jerk
subtitle: Malware Rejection Quickly
date: 2017-08-28
---

I was reviewing my NGINX access logs recently and found, to my surprise, hundreds of requests to Wordpress endpoints. Since I was only running a couple NodeJS apps and no Wordpress on this server, I quickly realized. . . my little server was being attacked!

I noticed that all the requests had the same user agent string: "Mozilla/5.0 Jorgee". After some Google sleuthing, I found [an article on Jorgee from Dog Is My Copilot](http://www.skepticism.us/2015/05/new-malware-user-agent-value-jorgee/) written in 2015. The article listed many of the requests that I was seeing in my access logs. I decided it was time to short-circuit Jorgee and tell him how I felt.

Using NGINX as my reverse-proxy allows me to short-circuit requests like this fairly easily. I can add a quick check for the user agent Jorgee seems to use for all requests and deny him before his requests take up any more CPU time.

```nginx
if ($http_user_agent ~* "Mozilla/5.0 Jorgee") {
  return 403 "screw you, Jorgee";
}
```

According to [the NGINX docs on if-statements](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#if), I have the ability to add this if-statement to any server or location block. Success! After some copy-pasting and testing I was able to stem the Jorgee onslaught.

"But Adam, how did you test?", you ask. Let me tell you. Chrome has some awesome developer tools that are readily available to assist you in testing in the browser. In this case, I opened up the dev tools (F12), clicked the three dots on the top-right, expanded "More tools" and selected "Network conditions". Doing so loads a new tab next to the "Console" tab for you. The "User agent" option will probably have the "Select automatically" checkbox checked. Un-check that, make sure you have "Custom..." selected in the drop-down and start typing your custom user agent string.

![Jorgee user agent](/images/jorgee-user-agent.png)

I know Jorgee isn't the only jerk out there on the internet. In the future, I'm going to want to add more of these kill switches. I don't want to have to copy-paste my way to unmaintainable code. [NGINX include to the rescue!](http://nginx.org/en/docs/ngx_core_module.html#include). I pulled the if-statement out of the server blocks and into a separate `malware_sucks.conf` file. Now I can include Jorgee and any of his crappy friends in all my server and/or location blocks easily.

```nginx
server {
  ...
  include /etc/nginx/malware_sucks.conf
  ...
}
```
