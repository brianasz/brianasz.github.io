---
layout: post
title: A few concepts about Fastly and Varnish
---

I started to hear about CDNs since I joined my current job. One of the CDNs we use is Fastly, it is being used in almost all of our projects so I got a brief introduction about it when I first joined the company. At that time I learned that Fastly was a Content Delivery Network that sits in front of every project's infrastructure and handles cache. 

Since then, I have barely interacted with it actively, however, I was recently assigned to a new project and they are expected to go live in a couple of weeks but, they are planning on going live only on specific regions of a country and they want authentication set up for one specific region. In order to do that I need to perform configuration changes on Fastly.

So today I learned a couple more things about Fastly and overall CDNs...

* *That behind Fastly there is a software named Varnish*
* *That Varnish is an HTTP caching system. In fact, Varnish is an HTTP accelerator designed for content-heavy dynamic web sites.* 

But what does content-heavy dynamic web sites even mean? 

Well, when a website has a lot of dynamic content (web content that changes based on the behavior, preferences, and interests of the user), that will cause the backend servers to be constantly active because every time a resource is requested it will be requested from them since dynamic content is not cacheable. So Varnish helps to easy the workload the backend servers receive by caching part of the content, the static content, hence serving it itself without the need to reach out to it to the backend servers as well.

**Note:** *Varnish will cash only static content.*

* *I also learned that Varnish is designed to work specifically with HTTP.*

* *That an HTTP accelerator is a proxy server that reduces web site access time.*

* *That VCL is Varnish Configuration Language and it is the domain specific language of Varnish and the way we apply rules to it. Varnish will translates this configurations into binary code and execute them when the request arrives. In fact, Fastly actually reads a VCL file where we can put all the Varnish rules we want to apply.*

* *That the VCL file is divided into subroutines that are applied at different times. One is executed when the request arrives to the website and another when files are fetched from the backend servers.*

**Subroutines:**

> vcl_recv = is called at the beginning of a request, after the complete request has been received and parsed. Its purpose is to decide whether or not to serve the request, how to do it, and, if applicable, which backend to use.

> vcl_fetch = is called after a document has been successfully retrieved from the backend. Normal tasks here are to alter the response headers, trigger ESI processing, try alternate backend servers in case the request failed.

**Note:** *According to the documentation, 99% of all the changes you’ll need to do in Varnish will be done in those two subroutines (vcl_recv and vcl_fetch).*

So, going back to the reason I got this task, if I need to limit the access to a website to a specific region by using Varnish rules, then I need to add a rule to the vcl_recv subroutine, because that's the subroutine than decides what to do with the request when it first arrives. Besides that, I also need to make use of Fastly Geolocation VCL features. 

In the case of Fastly, it basically has a list of variables that facilitates making configuration changes on the VCL file. The variable I'll need to make use of is "client.geo.region". So, for example, if I want to limit the access to my website, let's say www.brianasz.com to the states of Sinaloa and Sonora, Mexico, then we need to use something like this: 

```bash
if ( client.geo.region != "SIN" && client.geo.region != "SON" )
{ 
  error 404 "Not Found";
}

# Where SIN is Sinaloa and SON is Sonora according to the ISO 3166-2 
# country code subdivision
```

What this does is, if traffic coming to the website doesn't come from the state of Sinaloa or Sonora, Mexico, then the website should return a 404 not found error. 

And how do I set up authentication for one specific region, for example Sinaloa?... With something like this:
```bash
if ( client.geo.region == "SIN" ) 
{
  if (! table.lookup(customer_keys, req.http.Authorization) ) 
  {
    error 401 "Restricted";
  }
}
```
that, and with a VCL snippet that contains the actual keys, something like this:

```bash
table customer_keys {
  "Basic key_value_pair_basae64_encoded": "project_name"
}
```

**Note:** *The region code is based on ISO 3166-2 country code subdivision.*

## Conclusion ##
Sometimes I have a hard time to structure the ideas in my head in order to write something in here. There is plenty of topics to write about and a lot I don't know of technical stuff, so I though, why not just writing small blogs about what I learned today, and this came up. I'll try to keep doing this. It helps me to memorize things and understand them better. So, this is the summary of what I learned today, about how to restrict access by region to a website using Fastly and Varnish.
 
## References ##
[Fastly Geolocation](https://docs.fastly.com/vcl/geolocation/)

[ISO_3166-2](https://en.wikipedia.org/wiki/ISO_3166-2)

[Basic Authentication](https://docs.fastly.com/en/guides/basic-authentication)

[Varnish Configuration Language](http://varnish-cache.org/docs/trunk/users-guide/vcl.html)