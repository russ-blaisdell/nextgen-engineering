# Hyperscaler Account/Project/Subscription Governance

  Enabling teams to adopt and embrace the hyperscalers while concurrently applying security and governance can easily
be done in the traditional model of conflict
  
That is a bold statement. Surely many people who are experienced with networking and who have spent many years
recommending VPNs as part of their architectural solutions will find this statement hard to believe.
This post aims to help get people thinking about the potential implications of a world where API Gateways become
common across the industry. It is easy to consider API Gateways as just yet another component in the long list of
technologies that are deployed as
part of a solution. Add a load balancer, add a firewall, add an API Gateway, etc. However only seeing
API Gateways in that light leads to missing out on the additional security controls that an API Gateway
can provide and how these can be leveraged to create a stronger security posture with better governance than traditional
VPNs.

## Overview
![AWS Overview](../images/hyperscaler-governance/Hyperscaler-Operational-Management-Amazon.drawio.png)


### The good

Historically the way to control programmatic access between two systems in different network realms, such as between
two companies, was to establish a VPN. Wih a VPN we can be sure only systems in the source company that can reach the
source side of the VPN are able to access any of the systems at the destination end of the VPN. You can further restrict
which set of IPs can be reached and which ports on those IPs are available to systems at the source.

### The bad

While it was possible to restrict the port and IP at the destination it was not possible to restrict what was exposed at
the
given port and IP. So while you may have intended to expose the IP and PORT to your JIRA server for its API, as an
example, you have no way to restrict the
interactions to only access the JIRA API at that given port and IP. If someone configured ssh to be on that port, well
your VPN just exposed SSH, since it is just a layer 3 device
it is not able to recognize nor limit what is happening on the ip and ports it is exposing, it is, after all, only a
dumb gateway.

### The ugly

VPNs only stop access to the first hop in the chain. If any exposed IP/Port pairing supports SSH then everything visible
to the first
system is now at risk of being detected, attacked or accessed from the first server. Your risk and exposure only grows
from here. Not allowing SSH
is great however VPNs do not operate at layer 7, so they do not have the ability to block SSH or any other protocol from
being used. It is also
far too easy for someone to make a single typo on a CIDR and instead of exposing 4 IPs in a block they instead exposed
32 IPs, or far more. The potential
for mistakes when working with exposing your network via a VPN is quite high and the ability to detect those issues
before they are exploited is low.

## So what's this API Gateway 

An API Gateway is a solution that acts as a bridge to expose and govern access to services within your network. It
operates at layer 7 of the networking stack and is able
to restrict communications to specific protocols such as HTTP, gRPC, WebSocket and others. Thus, you are able to ensure
that only API communications can occur. By restricting
access to only specific protocols you can ensure that no one is able to SSH into your systems, nor any other protocol
you do not intend. You further restrict which APIs can be
interacted with so that you have granular control over what you are exposing as part of each integration. In addition to
having granular control over each API exposed to each
external party you can augment this with requiring a unique SSL certificate be presented with a different certificate
used for each integration. And you get to do all of this
without ever exposing your raw host/port network access to anyone.

## Sounds cool what does this look like in action?

### Traditional Layer 3 Gateway (aka VPN)

![VPN World](../images/api-gateway/API-Gateway-VPN-VPN.drawio.png)

In a VPN setup as shown above we have a single environment where we are going to establish communications with two
customer environments. In this case we would setup some IPs on the source side as virtual (not real IPs) that the source
side VPN will handle. Traffic for those IPs is then mapped to the correct client side VPN. Each client side VPN is then
further configured as to which IP/Port within the customers network the traffic should route to.
Once you have the VPN setup (route table is a simple view of these types of configuration), you then need to setup the
actual configuration of the applications at the source so they target the proper IP/Ports on the source side that
ultimately route into each customers
internal environment through their VPN.

NOTES:
Given all the mapping and NATting that often occurs due to a lack of virtual IP space, most configurations utilize
unique IP/PORT pairings, and it is not uncommon to have many communications flowing through a single IP with different
ports on the source side for a single IP ultimately leading to many IPs on the destination side. This compression of the
addressing space makes the configuration cryptic and prone to mistakes. Remembering that port 100 on IP 10.10.10.1 is
tied to customer (
A)'s JIRA Server while port 102 on the same IP 10.10.10.1 is tied to a different customer's
server is not obvious at all and tremendously easy to get wrong. Due to this you need very tight interlock between the
network/VPN configuration team and the application configuration team and any disconnect between parties leads to
incorrect configuration and data going the wrong way to or from the wrong customer.

### Next Generation Layer 7 Gateway (API Gateway) Outbound flow

![API-GATEWAY-in](../images/api-gateway/API-Gateway-VPN-API%20Gateway%20(OUTBOUND).drawio.png)

In an API Gateway outbound to clients, as above, each client has their own API Gateway which is used in lieu of VPNs.
The customer grants our source certificate access to just the APIs they are exposing to us. There is no routing table.
There is no dealing with virtual IPs, NAT, no cryptic and error-prone IP/Port pairings. The only configuration on the
API Gateway is create a new entry with the certificate for the client and select which APIs they are authorized to
interact
with. No NAT, all public IPs with trusted certificates and guaranteed SSL communications using the proper protocols
only.

### Next Generation Layer 7 Gateway (API Gateway) Inbound flow

![API_GATEWAY-INBOUND](../images/api-gateway/API-Gateway-VPN-API%20Gateway%20(INBOUND).drawio.png)

In an API Gateway inbound from clients, as above, each client has their own client side certificate to identify
themselves. At the inbound API Gateway a separate configuration is created for each client where only the certificate
unique to that client is authorized access. Once again no routing, no NAT, no cryptic and error-prone IP/PORT pairings.
Just very clear and clean configuration and all using public hostnames all with certificates.

## Summary: Layer 3 VPN Gateway .vs. Layer 7 API Gateway who wins....

It all matters. If you are working with exposing APIs and enabling programmatic integration then the power of layer 7
gateways makes them the best choice for integrations between parties. The granular controls, the additional security
restrictions that
control protocols and even which subset of a services API are exposed make a Layer 7 gateway a far superior choice. And
even if supporting older protocols such as SSH you would
restrict the VPN gateway solely to SSH traffic while relying upon the superior API gateway for as many integrations as
possible. 

### What about SSH and exposing my windows remote desktop ?

If you are dealing with SSH then you are dealing with humans coming into an environment. That is a high risk scenario
that is best suited to VPNs. Although the best recommendation is to use automation to govern your environment. And
leverage a front end, such as Ansible tower, or
others, to govern access. And limit VPN/SSH to only catastrophic issues. SSH is a case where the less you rely upon it
the better off you will be on every level.

And Microsoft Remote Desktop? That is a 1980's approach. Drop the windows and reach for a great distro of linux :)  
Windows has its place, but it is rare to find next gen technologies built on Windows instead of being built on
Kubernetes


  

