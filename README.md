# Proxy Rotator 

## Description 

The proxy rotator project focues on optimizing the speed at which webpages requested through Selenium are returned through proxy servers. It is achieved through a rotation algorithm that prioritizes speed and will utilize the fastest server until a request fails or the average response speed becomes higher than another server. Before the optimal server is determined each server in the provided pool is given a chance to determine a basline request speed. If one of these two rotation conditions is triggered, requests will be sent to another server that is deemed the fastest.

The application is optimized to handle public ips that are likley to be unreliable. The algorithm assumes a failed request is associated with a dead server and that server will be removed from the set of proxies considered for requests. If the connection is guarenteed to be reliable this functionality can be bypassed by repopulating the undefined queue with that proxy. 

### A set of proxies is initialized (represented by yellow dots)
![init_proxy](https://user-images.githubusercontent.com/29416921/111508571-d7d62580-8719-11eb-82b4-9ccc1f9f055a.gif)
### A proxy is sent requests (proxy currently being sent requests is represented in purple, response time of animation matches actual)
![first_active](https://user-images.githubusercontent.com/29416921/111506956-297db080-8718-11eb-9b29-9c523708faf7.gif)
### The remaining proxies are sent requests in a round robin 
#### Proxies that fail to respond are removed (represented by red dots), Valid proxy are added to the heap (represented by blue dots with gray animation)
![remove](https://user-images.githubusercontent.com/29416921/111506966-2be00a80-8718-11eb-8c10-045f6ab6a0cb.gif)
### After all proxies have been tested, the fastest is given requests. This rotates if the speed falls below another proxy or a request fails.
![optimize](https://user-images.githubusercontent.com/29416921/111506974-2edafb00-8718-11eb-8726-7d23fb86a2d4.gif)

## Usage

|parameter |data type | description |
|----------|----------|-------------|
|local_preferred (optional)|boolean|If set to True, the local ip is prioritized and the proxies are only accessed after a request to the local ip fails. If set to False, requests only go to proxies and the local ip is not considered.|

```python
proxy_rotator = proxy_manager(local_preferred = True)
proxy_rotator = proxy_manager()

proxy_rotator.get_webpage(url)
```

## How is the proxy pool populated? 

Currently the system is limited to the free proxies provided https://www.us-proxy.org.
An integrated scraping script extracts the proxy information provided on the sites proxylisttable HTML element.

This dataset is updated frequently and provides a consistent pool for the application to access at any given time.

Future iterations of the application intend on implementing additional functions to target other public ips and to allow user provided ips.

The population process is run everytime the existing set of alive proxy servers is empty.
Future optimizations intend to implement a threshold value to attempt to provide new proxies when alive servers exist but may have poor preformance. 


## Primary Components of the proxy manager

### Proxy

#### Key Attributes

* **local perfered boolean:** if set by the user, proxies will be bypasse in favor of local ip requests. The proxies will only be targeted if a local request fails. 
* **ip**
* **port**
* **average request:** (total request time / number of requests)
* **location data:** (latitude and longitude used for visualization)


### Undefined Queue

A queue data structure that represents the pool of proxies that do not have an associated average request time yet. These proxies are prioritized to recieve requests. Each proxy will be given one request. 

|Request | Result|
|------- | ------|
|Success | the time to complete the request is recorded, the ip is deemed active, and it is forwarded on to the proxy heap.|
|Failure | the ip is removed from future request consideration and freed from memory.| 



### Proxy Heap

A min heap data structure. For each request directed at the proxy heap the active proxy with the shortest average response time is popped from the heap and chosen to recieve the request. 

|Request | Result|
|------- | ------|
|Success | The average request time is updated and the proxy is pushed back onto the heap. If the updated request time remains the minimum it will be targeted again for the next request due to the nature of the data structure.|
|Failure | The ip is removed from future request considerations and freed from memory.|



## Rotation Conditions
1. A webpage request fails

2. The updated average request time increases to a degree that causes a different ip to be the fastest

## Algorithm

```
if no alive ips
	if undefined proxy pool empty
		populate undefined proxy pool
	else
		pop proxy from undefined proxy queue and submit request
		if request sucessful
			update average request time and push proxy to heap
		else
			remove proxy from consideration
else 
	pop minimum request proxy from heap and submit request
	if request sucessful
		update average request time and push proxy to heap
	else 
		remove proxy from consideration
```![init_proxy](https://user-images.githubusercontent.com/29416921/111506914-1ec31b80-8718-11eb-95c9-2b8961e4b1f5.gif)
