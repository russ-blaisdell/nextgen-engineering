# Authorization, Entitlements and Visibility

  Over the years I continue to find that people confuse, and conflate, these concepts and fail to recognize and treat each as 
independent but related.  As one of my very smart colleagues always reminds me, terms matter. Establishing the proper
terms and distinguishing between each of them is essential to enabling the best solutions.

## Authorization
This is the concept most people get, and one they often confuse and conflate with the others.  

What distinguished it from the other concepts:

<span style="color:blue"> Authorization is normally a personal thing.  And this means that in any normal system with multiple users some users will be
authorized for some things that others are not.  This also means that when a user finds they are not authorized to view, create, modify
or delete something in the system then they will seek out people who are or seek to get their admin to grant them the necessary authorization
so they can complete a given operation.</span>

## Entitlement
This is the concept where a system can provide many capabilities for end users and where some of those capabilities are optional.  This is more frequently
the case where a system offers a base set of features to users at one price, possibly free, however additional features and capabilities require that someone
request/purchase and become "entitled" to these new capabilities.

What distinguished it from the other concepts:

<span style="color:blue"> Entitlement is normally a system-wide thing.  If, when using a system you see that you are not entitled to use a certain feature this
leads you to seek out an evaluation of the capability and the costs and to then work to get your company to pay to acquire entitlement to the given capability
and become entitled to use it.</span>

## UI Visibility

  Visibility is all about determining what the user can see and where they can navigate within an applications interface.   

Relationship to other concepts above.

### Authorization
  It is common to not want to show a user a link that, were they to click it, would not lead them down a path they could succeed.  It is also common to want to show but disable items.
Thus if someone is not authorized to place an order the "order" button would be gray.  If there is a link in a menu for "orders" perhaps you do not want to show the link as clicking it
is of no value to someone who cannot place an order.  A more advanced application would show the user that they are not authorized to "order" but would still allow the user to see the
catalog of items to be ordered as seeing the catalog provides value to users who cannot themselves order but who may first peruse the catalog and then request a colleague who is authorized to place the order.  Thus even where someone is not authorized for something it is important to keep this granular.  
Allowing a user who cannot create an order to see their list of orders is not inherently a problem as it should appear blank.  Blocking that view entirely would seem of little benefit. 

### Entitlement
  It is common to not wish to show parts of an application that the customer, and therefore all their users, are not entitled to interact with.  So obscuring or hiding is an often sought technique.  More advance solutions, however
do not hide the feature, instead they show it as an option and if selected will take the user to a page to learn about the feature in hopes of driving interest in the company acquiring the feature by subscribing for the capability for use in their company.

## API Visibility
  While it is hard to "hide" APIs from different users or customers (and all their users) based on authorization or entitlement there are 
still some key good best practices we can follow.

  
## API handling 
  APIs need to be able to respond appropriately to callers.  These responses can often be interpreted by the systems on UI however these also will appear when 
someone is interacting with the service directly making API calls.  What is important is to ensure you distinguish between Authorization and Entitlement error codes 
so that the UI can handle these appropriately as can the end user who is directly making API calls.  403 is the answer for an authorization problem and the UI should be 
clear in informing the user that they are not authorized.  This is something they can then seek authoorization from an admin to address.  Calling an API for a function that
is not entitled should not be a 403, however but a 400 with a detailed response.  404 is also a viable answer in this case and the function they are requesting is not found
as it is not available as they are not entitled.  Providing details in the response payload should help the caller understand why the call failed.




