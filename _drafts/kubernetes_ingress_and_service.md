---
layout: post
---

I've been working for a while with Kubernetes and still I get confused about ingress and service, so in order to finally correct that, I need to write about it. 
So, what are the options? 

Let's start for an object in Kubernetes is. An object in Kubernetes is every abstraction or entity that represents the state of the system, example: pods, services, namespaces, volumes, etc, etc, etc.

Context

Pods are mortal and so its assigned IP address. So, if a frontend application is trying to communicate to a set of backend pods, how can we keep track of which IP addresses to connect to? By using a service object. 

A service object is a kubernetes abstraction that defines a policy by which to access a set of pods. This way, a frontend and backend apps can be completely decoupled leaving the service object the mission of coupling them. 

There are different service types:
Cluster IP: Default kubernetes service. It is a service inside the cluster that internal applications can access. There is no external access.

NodePort: A specific port on all the nodes (VMs) is open and any traffic sent to this port if forwarded to the service. 
Loadbalancer: A load balancer will be provisioned for the service on cloud providers that support external load balancers. This is the standard way of exposing a service to the internet where a single IP address will forward the traffic to the service. 
 
An ingress is not a service type object. An ingress sits in from of the services and act as an entry point and smart router to the cluster. In terms of definitions, Ingress is the set of routing rules that will govern how users access services. An ingress controller
however is how we implement these routing rules. The ingress controller will read the ingress resource information and processing it accordingly. An example of ingress controller is nginx. 

But if there is already a load balancer service type, why do we need an ingress that is basically another load balancer? 
Because a load balancer service type will provision a new IP address for each Lb service we deploy and it comes with cost, while the ingress will expose multiple services under the same IP address, where services will be reach by the routing rules. 

Is it possible to use both? 

#ExternalName



 
Cluster IP is a service that allows you to have an internal out of the box IP that only other application within the cluster can access, basically there is no external access. This is the default Kubernetes service. 
In order to 








In today's post I want to address what is SSL (Secure Sockets Layer) , why do we need them and how can we troubleshoot them.

When we are visiting a website, we want to make sure of two things:

1) The information we are transfering to the webiste is only between us and that website, in other words, we want to establishing an encrypted connection to the website
2) EStamos visitando el lugar que decimos visitar. 

Y como nos aseguramos de lo anteior? using a technology called SSL. 
SSL means Secure Sockets Layer and this a cryptographic protocol designed to provide communications serurity over a computer network. It is a security technology for establishing an encrypted link between a browser (your computer) and a web server (the website).

Let's put an example, let's say you want to buy a product from a website (amazon.com) so you need to pass credit card information to that website, so, by using SSL technology, you will make sure you are passing this information in an encrypted way so no one can see your credit card information and 2) that you are passing this information in amazon.com instead of an impersonator. 

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
