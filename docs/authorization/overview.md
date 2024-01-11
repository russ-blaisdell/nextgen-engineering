# Authorization, Entitlements and Visibility

  Over the years I continue to find that people confuse, and conflate, these concepts and fail to recognize and treat each as 
independent but related.  As one of my very smart colleagues always reminds me, terms matter. Establishing the proper
terms and distinguishing between each of them is essential to enabling the best solutions.


## Resources
  In any system or application the authorization model, when done appropriately, will focus on the data or resources that the system or
application operates upon.  For sharepoint the resource type would be "file" and authorization for a sharepoint server would focus on who is 
authorized to Create files, who can Read files, who can Update files and who can Delete files.

### Advanced Step One
  Obviously only operating on the files would have left sharepoint a very weak solution.  Thus another key resource in this example is folders.
Once again who can Create folders, who can Read Folders, Who can Update folders and who can delete folders is another level of necessary
authorization.

### Getting Fine Grained
  Before we can make Sharepoint into a fully functioning solution we need to not only control the CRUD operations of files and folders as resource types
  (.i.e User A can create a Folder or Delete a Folder).  We need to support applying these authorizations at the individual instance level of these resources.
So now we can say user A can Read Folder "General Info" however only User B can read folder "Top Secret".  This fine grained authorization is needed in most 
systems as the set of users increases and the amount of resources that the system manages grows.  i.e. In a sharepoint deployment with only one folder have fine
grained authorization is unlikely to be critical, expand that to 50 users and 200 folders and it becomes essential.

## Features
  In any system or application the concept of features are present.  These are functional areas of the solution that produce a 
distinct value and business outcome for its users.  As systems evolve they will grow features and in time it is common for people
to group the individual features together.  Some simple examples are a tiered model such as "Basic", "Intermediate" and "Advanced".  
  Others will move to a "base" and then a set of independent feature groups in our sharepoint example this might include "graphing" 
and "analytics" where the graphing package/feature set allowed for draw.io and visio built in while the analytics included spreadsheet/excel like capabilities.  


## Authorization
This is the concept most people get, and one they often confuse and conflate with the others.  Authorization is all about the resources
as defined above.  And, as mentioned in our sharepoint example, it is rare that a user is forbidden (403) from asking for the list of files or folders
and instead it is most common that the set of files or folders they see are less than the total set within the system.  

### Pro tip:
  It is often best that if a user asks for a specific file or folder they are not authorized to read that you not return 403 but instead return 404.
If you return 403 you now leaked to that unauthorized user that something they should not know about exists.  Returning 404 ensures you did not 
validate a fishing expedition.  Github is a great example of this.  If you are not authorized to access a repo you will get a 404 when you ask for it and not a 403.

### What distinguished it from the other concepts:

<span style="color:blue"> Authorization is normally a personal state.  And this means that in any normal system with multiple users some users will be
authorized for operations against certain resources while others are not.  This also means that when a user finds they are not authorized to view, create, modify
or delete a resource or resource type in the system then they will seek out people who are authorized or seek out to have someone grant them the necessary authorization.</span>

## Entitlement
This is the concept where a system can provide many features for end users and where some of those features are optional.  This is most frequently
the case where a system offers a base set of features to users at one price, possibly free, however additional features require that someone
request/purchase and become "entitled" to these use these additional features.

### Pro tip:
  Where possible you will want to make features visible to users to inform them of their existence in the hopes of driving up interest and
ultimately leading customers to purchase/acquire these.  You also want to not overload the end user as it is easy to annoy and offering to hide
this information is one way to not overwhelm and annoy your customers, always something to be avoided.

### What distinguished it from the other concepts:

<span style="color:blue"> Entitlement is normally a system-wide state.  If, when using a system you see that you are not entitled to use a certain feature this
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




