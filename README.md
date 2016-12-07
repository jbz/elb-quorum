## elb_quorum

This script is a purpose-built hack intended to allow for more controlled cluster restarts when operating under
an AWS load balancer.  Essentially, it has each instance which needs to restart its services first check
the ELB to determine what fraction of its instances are registered - by checking all nodes for namespacing 
indicating membership in that cluster -  and compare that fraction to a defined QUORUM fraction.  
If removing itself would take the cluster below quorum, the instance waits for a period and
then checks again.  After a constant-defined number of checks, it will exit[1], complaining that it was unable
to reach quorum.

If the ELB has sufficient instances in service, the instance will deregister itself from the ELB (which will,
if configured on AWS, include connection draining).  Once deregistered the instance will restart its local
services (using supervisorctl, in the current code) and then reregister with the appropriate ELB.

Warning: this code was written for a specific use case, and makes assumptions left, right and center.  Be sure
you know what it's doing before you attempt to use it in your environment.  One big undocumented assumption is
how it determines ELB names and clusters.  There is a list of 'supported domains' in the constants - this
script assumes that all ELBs are named in the format subdomain-domain-tld (for example, web-example-com) and
that instance hostnames will match those domain names (e.g. anything named i-xxxxx.web.example.com belongs
in web-example-com).  This refers to the ELB name, not its DNS name or alias.

It requires boto2, docopt, and various other python packages.  It was written for a python 2.7 environment.

Also note - there is a slight delay during the instance registration/deregistration, so it is normal for a
cluster to drop slightly below the set quorum fraction as a few instances 'race' each other to deregister.

NO WARRANTY IS IMPLIED OR OFFERED.  USE THIS CODE AT YOUR OWN RISK.  OSWALD ACTED ALONE. I HAVE NOT GIVEN
A PERFECT EXPLANATION OF THE FUNCTIONING HERE, READ THE CODE.
