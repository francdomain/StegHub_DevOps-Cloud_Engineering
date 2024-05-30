# Apache mod_proxy_balancer Module Documentation

The mod_proxy_balancer module in Apache HTTP Server is used to provide load balancing among backend servers. This module allows the distribution of incoming HTTP requests across multiple servers, improving redundancy and scalability. Below is a comprehensive guide covering different configuration aspects of the mod_proxy_balancer module, the concept of sticky sessions, and their usage scenarios.

### 1. Enabling mod_proxy and mod_proxy_balancer
Before configuring load balancing, ensure that the necessary modules are enabled. Add the following lines to your Apache configuration file (usually ```httpd.conf``` or ```apache2.conf```):

```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```

### 2. Basic Configuration of mod_proxy_balancer
To set up a basic load balancer, you need to define a BalancerMember for each backend server. Here's an example configuration:

```apache
<Proxy "balancer://mycluster">
    BalancerMember "http://backend1.example.com:8080"
    BalancerMember "http://backend2.example.com:8080"
    ProxySet lbmethod=byrequests
</Proxy>

ProxyPass "/app" "balancer://mycluster"
ProxyPassReverse "/app" "balancer://mycluster"
```
- __BalancerMember:__ Defines a backend server.
- __ProxySet lbmethod:__ Sets the load balancing method. Options include:
  - __byrequests:__ Distributes requests evenly across all workers.
  - __bytraffic:__ Distributes requests based on the amount of traffic (bytes) handled.
  - __bybusyness:__ Distributes requests based on the number of active requests.
  - __heartbeat:__ Uses an external heartbeat system to determine load.


### 3. Advanced Configuration Options

- __Stickiness (Sticky Sessions):__
Sticky sessions (or session persistence) ensure that a user session is always served by the same backend server. This is crucial for applications requiring session data to be stored on a specific server.

```apache
<Proxy "balancer://mycluster">
    BalancerMember "http://backend1.example.com:8080" route=1
    BalancerMember "http://backend2.example.com:8080" route=2
    ProxySet lbmethod=byrequests stickysession=JSESSIONID
</Proxy>
```


   __i. route:__ Specifies a route for each BalancerMember.

  __ii. stickysession:__ The name of the cookie used to maintain sticky sessions.

- __Failover and Recovery:__
Configure how the load balancer handles failed backend servers.

```apache
<Proxy "balancer://mycluster">
    BalancerMember "http://backend1.example.com:8080" route=1 retry=5
    BalancerMember "http://backend2.example.com:8080" route=2 status=+H
    ProxySet lbmethod=byrequests stickysession=JSESSIONID
</Proxy>
```
__i. retry:__ Time (in seconds) to wait before retrying a failed server.

__ii. status:__ Marks the server's status (e.g., +H to mark as hot standby).

- __Health Checks:__
Ensure that backend servers are healthy.

```apache
<Proxy "balancer://mycluster">
    BalancerMember "http://backend1.example.com:8080" route=1 hcmethod=GET hcuri=/healthcheck
    BalancerMember "http://backend2.example.com:8080" route=2 hcmethod=GET hcuri=/healthcheck
    ProxySet lbmethod=byrequests stickysession=JSESSIONID
</Proxy>
```
__i. hcmethod:__ The HTTP method to use for health checks.

__ii. hcuri:__ The URI to request for health checks


### 4. Sticky Sessions
Sticky sessions (or session persistence) ensure that a user session is consistently served by the same backend server. This is vital for applications where session data (like shopping carts, login information) is stored on the server.

#### How Sticky Sessions Work
When a client makes a request, the load balancer assigns a session to a specific backend server and routes all subsequent requests from that session to the same server. This is typically done using a session cookie.

#### Example Configuration
```apache
<Proxy "balancer://mycluster">
    BalancerMember "http://backend1.example.com:8080" route=1
    BalancerMember "http://backend2.example.com:8080" route=2
    ProxySet lbmethod=byrequests stickysession=JSESSIONID
</Proxy>

ProxyPass "/app" "balancer://mycluster"
ProxyPassReverse "/app" "balancer://mycluster"
```
- __route:__ The route identifier for each backend server.
- __stickysession:__ The name of the session cookie (e.g., JSESSIONID).

#### When to Use Sticky Sessions

- __Stateful Applications:__ Applications that store session data on the server.
- __User-Specific Data:__ When user-specific data (like profile information) needs to be consistently accessed.
- __Performance:__ Reduces the need for backend servers to share session data, improving performance.


### 5. Monitoring and Management

Apache provides a special status page for monitoring the balancer and its members.

```apache
<Location "/balancer-manager">
    SetHandler balancer-manager
    Require ip 192.168.1.0/24
</Location>
```
- __SetHandler balancer-manager:__ Enables the balancer manager.
- __Require ip:__ Restricts access to the status page to specific IP addresses.


### Conclusion

The mod_proxy_balancer module in Apache HTTP Server offers robust features for load balancing, including support for sticky sessions, health checks, and various load balancing algorithms. Properly configuring these options ensures high availability, scalability, and reliability for web applications. Sticky sessions are particularly useful for stateful applications where session data consistency is critical.



