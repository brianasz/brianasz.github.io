Ok, I needed to reduce the number of redirects from a virtual host recently but, I'm no expert on the subject and I don't know much about about apache redirects, so I had to start the investigation. 

Let's called my website wwww.pandita.com

First, how many redirects are happening right now? 
I used the following website to validate how many redirects were happening on my website:

http://www.redirect-checker.org

Actual result of the analysis:

Result
http://my.pandita-eu.com
301 Moved Permanently
https://my.pandita-eu.com/
302 Moved Temporarily
http://my.pandita-eu.com/en
301 Moved Permanently
https://my.pandita-eu.com/en
302 Moved Temporarily
https://my.pandita-eu.com/content/b2b/en/login
302 Moved Temporarily
https://my.pandita-eu.com/en/login
200 OK

Problems found: 
Too many redirects. Please try to reduce your number of redirects for http://my.shimano-eu.com. Actually you use 5 Redirects. Ideally you should not use more than 3 Redirects in a redirect chain. More than 3 redirections will produce unnecessary load on your server and reduces speed, which ends up in bad user experience.
You use 301 and 302 redirect at the same time. This might be confusing for search engine. Generally, please do not use 301 and 302 redirects at the same time.
You use a 302 redirect. This means, that the actually content is temporary not reachable and will come back soon. To use a 302 redirection for generally moved pages is a bad idea. Search engine bot might not follow it or handle it as temporary. For SEO this is also a bad idea, because no link juice will be transferred to the linked page.

Another way to check the number of redirects is by using the following curl command:
curl -s -L -D - http://www.pandita.com -o /dev/null -w '%{url_effective}'

What does this command shows:
-s, --silent
              Silent  or  quiet mode. Don't show progress meter or error messages.  Makes Curl mute. It will still output the data you ask for, potentially even to the terminal/stdout
              unless you redirect it.
-L, --location
              (HTTP) If the server reports that the requested page has moved to a different location (indicated with a Location: header and a 3XX response code), this option will make
              curl  redo  the request on the new place. If used together with -i, --include or -I, --head, headers from all requested pages will be shown.
-D, --dump-header <filename>
              (HTTP FTP) Write the received protocol headers to the specified file.
-o, --output <file>
              Write output to <file> instead of stdout.
-w, --write-out <format>
              Make curl display information on stdout after a completed transfer.
              url_effective  The URL that was fetched last. This is most meaningful if you've told curl to follow location: headers.

Turns out, the site is making in total 5 redirects, which the customer said were a lot, so REDUCE THEM!!! NOW!!!

But, this specific configuration is a cluster of webservers and it has an aws load balancer in front of them. So what happens if we git one of the servers directly?
When running curl, one of the redirects dissapears, this one: 
301 Moved Permanently
https://my.pandita-eu.com/en

When skipping over the CDN (cloudfront) and the load balancer, the site goes from https://my.pandita-eu.com/ to https://my.pandita-eu.com/en, but when going trhough cloudfront and the load balancer it goes from https://my.pandita-eu.com/ to http://my.pandita-eu.com/en (https to http), which makes the site to redirect it back again to https, adding one more 301 redirect. 

So what's happening at the cloudfront or load balancer level. 

Well, cloudfront is only sending the traffic to the load balancer without any special configuration. But the load balancer has the following listeners:
Load Balancer Protocol      Load Balancer Port      Instance Protocol       Instance Port       Cipher      SSL Certificate
HTTP	                    80	                    HTTP	                80	                N/A	        N/A
HTTPS	                    443	                    HTTP	                80	                Change	    c114578a-ae69-42c4-885a-bce409dcb49e (ACM) Change

So every time the load balancer is receiving traffic over port 443, it is sending forwarding the traffic to the backend instances trhough port 80, it can only means it is making SSL termination at the load balancer level.


Actual result of the analysis:

Result
http://my.pandita-eu.com
301 Moved Permanently
https://my.pandita-eu.com/
302 Moved Temporarily
http://my.pandita-eu.com/en
301 Moved Permanently
https://my.pandita-eu.com/en
302 Moved Temporarily
https://my.pandita-eu.com/content/b2b/en/login
302 Moved Temporarily
https://my.pandita-eu.com/en/login
200 OK

Problems found: 
Too many redirects. Please try to reduce your number of redirects for http://my.shimano-eu.com. Actually you use 5 Redirects. Ideally you should not use more than 3 Redirects in a redirect chain. More than 3 redirections will produce unnecessary load on your server and reduces speed, which ends up in bad user experience.
You use 301 and 302 redirect at the same time. This might be confusing for search engine. Generally, please do not use 301 and 302 redirects at the same time.
You use a 302 redirect. This means, that the actually content is temporary not reachable and will come back soon. To use a 302 redirection for generally moved pages is a bad idea. Search engine bot might not follow it or handle it as temporary. For SEO this is also a bad idea, because no link juice will be transferred to the linked page.


First let's understand some apache rules for redirects and we will be adding more as we find them on the conf file:
NC = (nocase) Makes the pattern comparison case-insensitive.
L = (last) Stop the rewriting process immediately and don't apply any more rules.
R - (redirect) Forces an external redirect, optionally with the specified HTTP status code.
RewriteCond = The RewriteCond directive defines a rule condition. One or more RewriteCond can precede a RewriteRule directive. The following rule is then only used if both the current state of the URI matches its pattern, and if these conditions are met.

The RewriteRule directive is the real rewriting workhorse. The directive can occur more than once, with each instance defining a single rewrite rule. The order in which these rules are defined is important - this is the order in which they will be applied at run-time.

sintax: RewriteRule Pattern Substitution [flags]

REQUEST_URI	= The path part of the request's URI
%{ENV:variable}, where variable can be any environment variable, is also available. This is looked-up via internal Apache httpd structures and (if not found there) via getenv() from the Apache httpd server process.
REMOTE_ADDR
The IP address of the remote host (see the mod_remoteip module).

-f
Is regular file.
Treats the TestString as a pathname and tests whether or not it exists, and is a regular file.


Now let's look at the conf file and compare with the analysis result:

http://my.pandita-eu.com
301 Moved Permanently

due to rules:
RewriteCond %{HTTPS} !=on
RewriteCond %{HTTP:X-Forwarded-Port} !^443$
RewriteRule ^/(.*) https://%{HTTP_HOST}/$1 [NC,R=301,L]

https://my.pandita-eu.com/
302 Moved Temporarily

RewriteRule ^(/)?$ /en [R=302,L]

http://my.pandita-eu.com/en
301 Moved Permanently



https://my.pandita-eu.com/en
302 Moved Temporarily
https://my.pandita-eu.com/content/b2b/en/login
302 Moved Temporarily
https://my.pandita-eu.com/en/login
200 OK




References
https://httpd.apache.org/docs/2.4/custom-error.html
