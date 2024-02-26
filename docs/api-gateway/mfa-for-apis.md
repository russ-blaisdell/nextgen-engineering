# MFA (Multi-Factor Authentication) for APIKeys 

  Adding MFA to your apikey strategy can dramatically increase the security of your solution.  While this can be very straightforward
when you host single tenant solutions, the complex quickly rises when operating a sharded multi-tenant/multi-customer solution.  The 
following blog explains the design I landed on, for now, to add MFA to our APIs.
  

## MFA for Carbon
  I am sure everyone is familiar with MFA for user logins where you need to receive a text message with a code and you need to provide that 
code along with your password to successfully authenticate.  Other systems can use biometrics, like the fingerprint scanner on your phone or
laptop.  In the worst case you get stuck using e-mail and waiting for the e-mail to arrive so you can "click the link".  

## MFA for Silicon
  Sure, so now we all agree on what MFA looks like for humans, does this mean we just replicate this same model for our APIs?  
We could surely register some e-mail addressed with gmail and then use IMAP or POP3 to retrieve our email and even click the 
link using chrome on linux using test drivers.  So yes, we could implement MFA for computers just like we do with humans however
that seems pretty 🤮.  Instead of going down that path we will use other technologies which are far more elegant and suitable to our 
goal, we will use client side certificates. Client side certificates are an excellent way for a caller of your APIs to add a second layer
of authentication by presenting an [x.509 certificate](https://en.wikipedia.org/wiki/X.509#Certificates).  As these certificates are 
signed and each has a unique signature it is very easy to incorporate these into your API strategy to add a second level of authentication
for your APIs and in doing so your APIs are now MFA.

## Tying it together in a multi-tenant world
  In a single tenant (i.e. Customer) deployment world you can easily setup each independent customer environment to require
and check for only the client certificates associated with that single customer.  Thus adding MFA to your APIKEY strategy when
running independent software deployments for each customer is very straightforward.  [mTLS](https://en.wikipedia.org/wiki/Mutual_authentication#mTLS) (mutual TLS) is then a commonly supported
pattern in SaaS service that host independent services for each customer.  ServiceNow, Atlas Mongo and many others provide mTLS as each
instance of their software is deployed and setup for single tenant/customer use.  However, for multi-tenant solutions that are 
supporting multiple customers all using a single instance of your software things quickly get more interesting and complex.

## Needs in a multi-tenant deployment

* Each client can have one or more client certificates to authenticate with
* API Calls to each clients tenants are only allowed when accompanied by that clients certificates
* In short only calls to client A's tenants are only allowed when accompanied by client A's certificates
* As all [APIs authenticate with bearer tokens](../bearer-token/api-key.md), following best practices, bearer tokens and api keys must be supported
* [API first architecture](../bearer-token/api-first.md) has bearer tokens used for UI/web browser flows as well as for API calls from API keys


# Design
  The following shows the design for levering an API gateway and IAM service to manage client certificates for all API
calls not originating from the browser.  There are two paths to consider.  One is incoming calls with an API Key where the caller
is exchanging their API key for a bearer token for use in subsequent API calls.  This is the method that does all the heavy lifting
and that is a key design goal as in any normal system the majority of API calls being processed are those authenticating with a
bearer token.  So having the majority of the processing on this call means that all other api calls, using bearer tokens, have the
least additional impact.  The second set of calls to handle are those where we are processing an incoming bearer token and need to
validate that the bearer token and client certificate match to the same customer tenant.

## APIKey Authentication and bearer token generation

  As can be seen below incoming requests to swap an API key for a bearer token will handle validating that the incoming call has
the proper client certificate and will then generate a bearer token with additional information about the specific customers
certificate such that any api calls made with the bearer token can easily validate the proper client certificate is being used.

![](../images/api-gateway/MFA-For-APIKeys-MFA-for-APIKeys-1.png)

## Bearer Token Authentication
  
  With all the hard work done in the previous step the API gateway need only validate the bearer token and match the value stored within it
to the value passed in as a header from the load balancer for the callers client certificate.

![Bearer Token](../images/api-gateway/MFA-For-APIKeys-MFA-For-APIKeys-2.png)

