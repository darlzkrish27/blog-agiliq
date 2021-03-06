---
layout: post
title:  "Using subdomains with Django"
date:   2008-10-10 20:31:57+05:30
categories: tips
author: shabda
---
Django makes using beautiful urls a snap. However using subdomains with Django is not so obvious. Here is an article explaining this.

Advantages of subdomain

1. Sometimes looks more professional. Users will prefer someblog.wordpress.com over wordpress.com/someblog.
2. If each part of your site is Under a different untrusted users, you might want to give them a separate subdomain, so that if they do Blackhat Seo on their part, the main site is not harmed.
3. Your users can point a Cname to their subdomain to use domain of their choice with their subdomain.

Disadvantages of subdomain

1. When you need subdomains, you will know it. If you do not if you need it or not, you probably do not.


Getting the current subdomain is ridiculously easy.

	bits = urlparse.urlsplit(request.META['HTTP_HOST'])[0].split('.')
	bits[0]
	
now bits[0] is you subdomain.

However if you are using subdomains you are probably going to be needing this,

1. In all your views.
2. In all your templates.

So you need to expose the subdomains using 

1. A [Middleware](http://docs.djangoproject.com/en/dev/topics/http/middleware/) for all requests.
2. A [request context](http://docs.djangoproject.com/en/dev/ref/templates/api/) for all templates.

A Middleware is nothing but a normal Python class which can implement `process_request`, `process_response` and others.

The code to expose subdomains for all requests via a middleware is,

	import urlparse

	class GetSubdomainMiddleware:
	    
	    def process_request(self, request):
		bits = urlparse.urlsplit(request.META['HTTP_HOST'])[0].split('.')
		if not( len(bits) == 3):
		    pass#Todo Raise an exception etc
		request.subdomain = bits[0]
		
The way to populate subdomain in all templates is similar

	def populate_board(request):
	    "Populate the board in the template"
	    return {'board':request.subdomain}#request.subdomain has been populated via the Middleware.
	    
And now you need to edit you settings.py file to add `TEMPLATE_CONTEXT_PROCESSORS` and `MIDDLEWARE_CLASSES` to include your Middleware and context processor. 

You are almost ready to go, however your cookies will not work across sub domains. To
make your cookies work across subdomains, add this line to your settings.py

`SESSION_COOKIE_DOMAIN` = '.example.com' if not DEBUG else '.local

[Thanks](http://sharjeel.2scomplement.com/2008/07/24/django-subdomains/)

