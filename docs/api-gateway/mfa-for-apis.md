# MFA (Multi-Factor Authentication) for APIKeys 

  Adding MFA to your apikey strategy can dramatically increase the security of your solution.  While this can be very straightforward
when you host single tenant solutions, the complexity quickly rises when operating a sharded multi-tenant/multi-customer solution.  The 
following blog explains the design I landed on to add MFA to our APIs.
  

## MFA for Carbon
  I am sure everyone is familiar with MFA for user logins where you need to receive a text message with a code and then you provide that 
code along with your password to successfully authenticate.  Other systems use biometrics, like the fingerprint scanner on your phone or
laptop.  In the worst case you get stuck using e-mail and waiting for the e-mail to arrive so you can "click the link".  

## MFA for Silicon
  So now that we all agree on what MFA looks like for humans, does this mean we just replicate this same model for our APIs?  
We could easily register an e-mail address with gmail and use IMAP or POP3 to retrieve our email and even click the 
link using chrome test drivers on linux.  So yes, we could implement MFA for computers just like we do with humans however
that seems pretty 🤮.  Instead of going down that path we will instead use a technology which is perfectly suitable to our 
goal, we will use client side certificates. Client side certificates are an excellent way for a caller of your APIs to add a second layer
of authentication by presenting an [x.509 certificate](https://en.wikipedia.org/wiki/X.509#Certificates).  As these certificates are 
signed with a unique signature it is very easy to incorporate these into your API strategy to add a second level of authentication
for your APIs and in doing so your APIs are now MFA.

## Tying it together in a multi-tenant world
  In a solution where you deploy a different instance of your software for each customer it is relatively easy to setup such that it requires
and checks for only the client certificates associated with that individual customer.  Thus adding MFA to your APIKEY strategy when
running independent software deployments for each customer is very straightforward.  [mTLS](https://en.wikipedia.org/wiki/Mutual_authentication#mTLS) 
(mutual TLS) is therefore a common pattern found in many SaaS service that host independent instances for each customer.  ServiceNow, Atlas Mongo 
and many others provide mTLS as each instance of their software is deployed and setup for only one single tenant/customer use.  However, for 
multi-tenant solutions where multiple customers all interact with the same instance of your software leads us to need to adopt a much more
advanced approach.

## Needs in a multi-tenant deployment

* Each client can have one or more client certificates to authenticate with in addition to their API key or Bearer Token
* API Calls to each clients tenants are only allowed when accompanied by that particular clients certificates
* In short all calls to client A's tenants require that the certificate for client A be presented along with a credential (API Key or Bearer Token) for that same client
* Client B cannot ues their client certificate to make calls against Client A's tenants even if they have a valid credential (API Key or Bearer Token) for client A's tenant
* Thus each client API call has two independent authentication elements, the client cert for the client and an APIkey or bearer token tied to the same client/tenant.
* As all [APIs authenticate with bearer tokens](../bearer-token/api-key.md), following best practices, bearer tokens and api keys must be supported
* [API first architecture](../bearer-token/api-first.md) has bearer tokens used for UI/web browser flows as well as for API calls from API keys


# Design
  The following shows the design for levering an API gateway and IAM service to manage client certificates for all API
calls not originating from the browser.  There are three key aspects to consider.
  
## API Calls requesting a bearer token in exchange for an API Key
This is the primary mechanism where we need to do all the heavy lifting of supporting client certificates.  
As this call occurs far less frequently then other API calls that use the bearer token provided by this call, it is best to do 
most of the processing as possible on this call and ensuring that all the other API calls have the least additional processing
overhead to support client side certificates.  

  As can be seen below incoming requests to swap an API key for a bearer token will handle validating that the incoming call has
the proper client certificate and will then generate a bearer token with additional information about the specific customers
certificate such that any api calls made with the bearer token can easily validate the proper client certificate is being used.

![](../images/api-gateway/MFA-For-APIKeys-MFA-for-APIKeys-1.png)

## Bearer Token Authentication
  
  With all the hard work done in the previous step the API gateway need only validate the bearer token and match the value stored within it
to the value passed in as a header from the load balancer for the callers client certificate.

![Bearer Token](../images/api-gateway/MFA-For-APIKeys-MFA-For-APIKeys-2.png)

## UI Interactions
  As mentioned in API First Architecture we utilize bearer tokens for all UI/Web Browser calls to our APIs.  As we are not going to require every customer have certificates
all of their users browsers, we need to be sure our support for mTLS does not break our UI.  This is easily handled by ensuring that when we generate a bearer token for a user
web browser login we do not include any certificate claim in the token.  Our API Gateway is smart enough to check for this claim and when blank or not present it skips any
client certificate validation checks.

# Summary
  Adding mTLS to your APIs can dramatically increase the security of your solution.  Doing so in the above approach, while a bit of work, ultimately makes for a very highly performant
and secure solution and helps you avoid having to deploy dedicated instances for each of your customers which would only increase the total cost of your solution.
