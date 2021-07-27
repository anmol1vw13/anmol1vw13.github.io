---
layout: post
title: "Making Actor's VM infrastructure reproducible!"
date: '2021-07-21T00:00:00.000-08:00'
author: Anmol Vijaywargiya
tags:
- tech
- compute engine
- chef
- knife
- ruby on rails
- ror
- ruby
---

It was the month of March, and we got to know that the Ubuntu 16.04 LTS EOL (End of Life) was scheduled for 30th April 2021.

Most of the infrastructure the actors relied on was based on ubuntu 16.04 and not having the support for security updates could mean putting 
them at risk. 

The ecosystem around actor was built with the help of Lambda such that the infrastructure would be provisioned only during first time deployments
and subsequent infrastructure related changes had to be done manually. 

Having a total of about 250 actors in production and integration environment, we had about a 1000 VMs to cater to.

There were a total of three options to go about this.
* Getting Extended Security Maintenance (ESM) with an Ubuntu Advantage subscription from
Canonical which provides at least three additional years of security patches. 
<br/>
<i>This would have costed a lot!</i>
<br/>
* Shifting each of the VMs manually to ubuntu 20.04. 
<br/>
<i>Not at all productive. Nuh-uh!</i>
<br/>
* Extending the lambda ecosystem to support reproducible infrastructure. 
<br/>
<i>We went ahead with this because this would help us support all future migrations via a click of a button.</i>
<br/>

Let's talk a bit about what the actor ecosystem with respect to infrastructure currently looks like .

* Actor team commits code on gitlab (our version control system).
* Gitlab then starts a pipeline which does the following:
    * Create a package and push it on to artifactory (the package manager).
    * Start the deployment of the code onto desired environment. (integration/production)
* In the deployment step, if infrastructure isn't already present, infrastructure creation gets trigger.
* The creation takes place via terraform scripts, which first creates Compute Engines on the GCP project(based on environment)
and registers the VMs on CHEF, our configuration management tool.
* chef-client, when run on the VMs, pulls the package from artifactory and runs related services using the tags, recipes and attributes set on the CHEF Node. 
These entities are currently set by teams themselves, based on their use-case.
* One such important service that runs in case of actors is consul agent. Consul helps in the service discovery and helps in getting the traffic to VMs via consul aware haproxy that the actor
ecosystem uses.

Now that we are familiar with how the actor ecosystem is, let's discuss the implementation for reproducible infrastructure.

The implementation was broken into 4 high level tasks.
1. Recreation of infrastructure set.
2. Listing of all available infrastructure sets.
3. Regulation of traffic between infrastructure sets.
4. Archival of infrastructure set(s)

PS: An infrastructure set is a collection of VMs that were provisioned at the same time and have the same characteristics.

The main challenge here was to create an identical infrastructure set. 

The initial infrastructure is provisioned with a set of tags/recipes/attributes which is sort of common to all the applications. Since all the subsequent changes
to the configuration were done by the teams themselves, there were a lot of unknowns. 

To cater to this we decided to bucket the chef configuration into features thereby making it extendable to other actors as well. 
These features are then used to extrapolate relevant recipes, tags and attributes which are passed on for infrastructure creation.

Since the users have complete access to the underlying infrastructure, in the recreation flow, we also have a check to understand whether the features recorded for the actor with us match the ones
found on the existing nodes. If they don't, recreation is not allowed to go through. If they do, infrastructure is recreated.

The infrastructure set created although will not have any traffic flow through and will controlled using traffic regulation.

Once the traffic regulation is complete and the traffic is on the recreated infra set, the user can then delete the infrastructure set
which doesnot have any traffic.


After the development was complete, we complete the infrastructure for all the actors in integration as well as production in a week's time.
Listing below, the impact of our effort: 
* Dev days saved: ~228
* The total cost saved: ~$11.3k per month (VM infrastructure)
* Potentially saved: ~$171k (on extended security support)
* Truly zero downtime migration

The whole process was seamless and we are now looking for extending this tool to other types of applications and other usecases as well.








 













   

