---
layout: post
title:  "Using Paypal with Django"
date:   2008-11-12 01:40:22+05:30
categories: paypal
author: shabda
---
Paypal has a comprehensive API to use their services programatically. The ExpressCheckout API allows you to get the user's details and then process the payments on your servers. They include a SOAP
and NVP API. With NVP you do a GET to the Paypal servers with Url encoded values, get responses in plain text and work with them.

The basic flow for ExpressCheckout is something like this,

1. You make a SetExpressCheckout request to Paypal, passing your credentials and and the amount you want to charge etc.
2. Paypal returns an response with ACK of SUCCESS and token which you need to pass to subsequent requests.
3. You transfer the user to Paypal site, passing the token returned in step 2.
4. User logs in, and authorises transfer. Paypal redirects back to the SUCCESS_URL specified in step 1, passing the token and PayerID.
5. You make a GetExpressCheckoutDetails request to Paypal, apssing token, payer id and other details.
6. Paypal returns the user's and transactions details.
7. You ask user to confirm details.
8. You make a DoExpressCheckoutPayment request to Paypal. At this point the money is transferred from user's account to your account.

Now you can do the requests to Paypal very simply by sending requests, (it is just Url encoded name-value pair), but here is a [Django snippet](http://www.djangosnippets.org/snippets/1181/) to make this wonderfully easy.

Before you can make requests to Paypal, servers you need to get API credentials. While developing your app, you of course, do not want to use actual Paypal money. Paypal makes available a sandbox where
you can use virtual money before you go live.

1. Go to [developer.paypal.com](http://developer.paypal.com/) and request a new sandbox API credentials. 
2. (Optionally)Read `https://cms.paypal.com/us/cgi-bin/?cmd=_render-content&content_ID=developer/howto_api_reference` to understand how does the Paypal NVP api works.
3. We would two pages. [a] View 1: Where we do SetExpressCheckout and show a link to Paypal. [b] View 2: Where we do GetExpressCheckoutDetails, get the user's details, get a confirm from user. On a confirmation from user
we post to the same page and do a DoExpressCheckoutPayment.

[Totally untested code ahead. I am copying and simplifying from one of our applications.]

	#view 1
	
	def veiw1(request):
	        ...........
		pp = paypal.PayPal()
		token = pp.SetExpressCheckout(...)
		paypal_url = pp.PAYPAL_URL + token
		payload = {'paypal_url':paypal_url}
		return render_to_response('template1.html', payload, RequestContext(request))

In the template send user to `{{ paypal_url }}` when they want to pay.
	
	#View 2
	#View 2 would be called after Paypal redirects after user authorises the transfer.
	
	def view2(request):
		token = request.GET.get('token', '')
		pp = paypal.PayPal()
		if request.method == 'GET':
			paypal_details = pp.GetExpressCheckoutDetails(token, return_all = True)
			payload = {}
			if 'Success' in paypal_details['ACK']:
				payload['ack'] = True
				token = paypal_details['TOKEN'][0]
				first_name = paypal_details['FIRSTNAME'][0]
				.....
			return render_to_response('template2.html', payload, RequestContext(request))
		if request.method == POST:
			payment_details  = pp.DoExpressCheckoutPayment(token = token)
			if 'Success' in payment_details['ACK']:
				pass
				#We have been paid. DO things like, enable subsciprion, ship goods etc.
				return HttpResponseRedirect(payment_done_url)
	
Here in template2.html, we show the user/transaction details, and a on user's confirmation do a post to the same url. This should help you get started with integrating Paypal with Django.

Resources.

1. [Paypal developer forums](http://paypaldeveloper.com/)
2. [Paypal developer documentation](http://developer.paypal.com/)

-----------------------
Need to build a Paypal enabled webapp? [We can help](http://uswaretech.com/contact/)



