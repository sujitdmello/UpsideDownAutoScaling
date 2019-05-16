# Upside Down Auto Scaling in Cloud Services

Most modern Cloud Services support auto-scaling but these services are slow to spin-up. This approach to setting up the scaling rules helps to achieve better application performance.

## The Scenario

In the era of serverless computing, scaling up and down of cloud services/resources is not something we take much time thinking about. It is assumed that the cloud provider has pre-provisioned extra resources to meet the demands such that any sudden scaling up of resources will happen in a few seconds. That is the price we pay for serverless compute (by-the-second).

However for other traditional cloud services such as VMs, Web Apps, Kubernetes, HPC, Hadoop etc., essentially all the services that charge you by the VM and therefore require spinning up new VMs to meet your application demands, the time taken to scale up can be in the order of a few minutes if not more. This is a problem for applications that are trying to ramp up at the start of a business day or to meet sudden demands in traffic based on seasonal or other factors.

### Traditional Scaling Rules 

Most cloud resources can be setup with scaling rules. These rules can react to increases in CPU, Memory, Disk I/O or even the length of a worker queue. When the increase is observed over a period of a few minutes, certain actions can be taken. The typical scaling rule may be something like: 

```
When Average CPU over 10 minutes > 70%, scale UP by 1 instance
When Average CPU over 10 minutes < 30%, scale DOWN by 1 instance 
```

The problem with this approach is that each reaction to the increase in CPU takes a few minutes before the additional instance is up and running. During this time the system is likely under some pressure. When the additional instance is added to the pool of servers, the systm may stabalize temperorily before it comes under some additional load. Once again the scaling rules will result in an additional instance being spun up, which make take a few more minutes and again causing the system to be uner some pressure. This cycle continues each tme the system is scaled up.

### The alternative approach

The scaling rules can be made a bit more aggressive such that when the CPU increases beyond some threshold, a number of instances (perhaps the maximum you are willing to request) are added. 

So the rule may be something like

```
When Average CPU over 5 minutes > 40%, scale UP by 10 instances
When Average CPU over 30 minutes < 30%, scale DOWN by 1 instance

```

What does this do? Let's disect it:
* First off it results in a very aggressive scale up and a conservative scale down. 
* Notice how the scale up threshold is pretty low now and the observation time is also down to 5 minutes, to enable it to react a bit quicker.
* On the other hand the scale down time is over 30 minutes which gives is more time to tear down these 'expensive' resources. Expensive in the sense that they take a long time to start up so we want to be sure before taking them down.
* The advantage of this approach is that the overhead of scaling up the VMs is incurred all at one time. So within a few minutes you have more capacity than you may ever need. 
* On the surface this may appear to be a waste of resources and money but this is not a bad thing as it has given the business more than enough to account for any load over the next several minutes. 
* And in case the system does not require all that capacity, well in about 30 minutes it will start dropping instances one at a time. This will continue until the system reaches a level of instances that is a appropriate for the load. 
* So instead of scaling **up** one at a time until we get to a steady state, we scale **down** one at a time. Hence the name :)  

There is a slightly extra cost overhead compare to the traditional scaling rules but this is minimal compared to the benefit to the application to be able to meet the user demaands more consistently.

```
It should be noted that CPU was used as a trigger just as an example.
You can use Memory and Queue length as well and it works just as well. 
If you have more of a background processing application you may want to look at these other metrics. 
```

While these rules can be applied to most cloud providers, as far as Azure is considered, these are the services that would support them (as of May 2019):
* Cloud Services
* App Services
* Virtual Machines Scale Sets
* Service Fabric
 
## Acknowledgments

* Thanks to [David Crawford] (https://github.com/dc995) for giving me the idea initially when discussing this internally at Microsoft
