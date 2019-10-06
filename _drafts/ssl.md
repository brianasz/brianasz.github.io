---
layout: post
title: Understanding SSL / TLS certificates
---

If I knew nothing about SSL / TLS, how would I like that someone explain me that topic?... 
In today's post I want to address that in an easy and comprehensible way.

When we are visiting a website, we want to make sure of two things:

1. That we are visiting who we say we are visiting, basically veriying the identity of the site owner and..
2. That the information we are transfering to the website is only between us and that website, in other words, we want to establish an encrypted connection to the website.

So, how we make sure of that? By using a technology called SSL / TLS. SSL means Secure Sockets Layer and is a cryptographic protocol designed to provide communications security over a computer network. Simplifying, it is a security technology for establishing an encrypted connection between a browser (your computer) and a web server (the website). TLS stands for Transport Layer Security and is the succesor of SSL, an upgraded and more secure version of SSL.

Let's put an example of what I have written so far. Let's say you want to buy a product from a website (www.amazon.com), so you need to pass credit card information to that website, hence, by using SSL / TLS technology, you will make sure that you are passing this information to www.amazon.com instead of an impersonator and that you are passing this information in an encrypted way so no one else can see your credit card information. As you can see, it is quite important. So the question is, how can we stablish an SSL / TLS connection? Well, by using a Certificate, an SSL / TLS certificate. Let's explain a little bit about this. 

An SSL / TLS certificate is just a small data file that digitally bind a cryptographic key to an organizations's details, in simpler words, it is a file containing identity credentials. There are 3 types of certificates and although the encryption levels are the same for each, the difference is the vetting and verification processes needed to obtain them and the look and feel of it in the browser address bar (in your computer). 

## Types of certificates

* Extended Validation (EV SSL)
With this type of certificate, a Certificate Authority (CA) which is the company issuing the certificate, checks the right of the applicant to use a specific domain name (example: www.amazon.com) and conducts a thorough vetting of the organization by verifying the legal, physical and operational existence of the entity among other things. This is the most trusted type of certificate.

* Organization Validated (OV SSL)
With this type of certificate, the CA (as described before) checks the right of the applicant to use a specific domain name and conducts some vetting of the organization. *Please note "some vetting" instead of "thorough vetting".*

* Domain Validated (DV SSL)
With this type the CA checks the right of the applicant to use a specific domain name but no company vetting is performed. So you get encryption but no trust on the identity of the site owner. 


## And how does this works? 
Browser connects to a web server (website) secured with SSL (https). Browser requests that the server identify itself.

Server sends a copy of its SSL Certificate, including the server’s public key.

Browser checks the certificate root against a list of trusted CAs and that the certificate is unexpired, unrevoked, and that its common name is valid for the website that it is connecting to. If the browser trusts the certificate, it creates, encrypts, and sends back a symmetric session key using the server’s public key.

 

Server decrypts the symmetric session key using its private key and sends back an acknowledgement encrypted with the session key to start the encrypted session.

 

Server and Browser now encrypt all transmitted data with the session key.
https://www.digicert.com/ssl/
## Structure of a Certificate.root,ca, etc, etc, etc

How ca we get one? 
Who sells or grants the certificate?

SSL Certificates bind together:
A domain name, server name or hostname.
An organizational identity (i.e. company name) and location.


And finally, how do you know a website has SSL tecnology enabled? 
byt checking https protocol. 


So, how does this work? 
All this is done trhough a Certificate, an SSL certificate. 


You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
