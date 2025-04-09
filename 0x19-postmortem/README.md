Postmortem: 502 Bad Gateway on Main Website (farm.brickservers.ng)
Issue Summary
Duration:
 April 8, 2025, from 10:15 AM to 11:02 AM (WAT) — 47 minutes
Impact:
 The main website and API (farm.brickservers.ng) for Brick Farms System was returning 502 Bad Gateway errors. Approximately 85% of users were affected — unable to log in, retrieve dashboard data, or access services. Website traffic dropped significantly, and some customers reported downtime on hosted web applications.
Root Cause:
 A sudden surge in API traffic triggered excessive memory consumption in the Nginx reverse proxy, which led to a worker crash. Nginx continued running, but it could no longer connect to the upstream Node.js app server.

Timeline
10:15 AM – Monitoring system (UptimeRobot) alerted 502 errors on farm.brickservers.ng.


10:17 AM – Support team confirmed downtime via manual browser check.


10:20 AM – DevOps team began inspecting Nginx logs and server load.


10:25 AM – Initially suspected a bug in the backend application due to recent code push.


10:30 AM – Rolled back last deployment to eliminate the code push as root cause — no improvement.


10:35 AM – Checked Node.js app logs — app was healthy and responsive on localhost.


10:40 AM – Realized netstat showed no active connection between Nginx and Node server.


10:45 AM – Restarted Nginx service and confirmed temporary recovery.


10:50 AM – Traffic surge occurred again, causing Nginx crash.


10:55 AM – Increased worker_connections and memory limits in Nginx config, then restarted service.


11:02 AM – Service fully restored and stable.



Root Cause and Resolution
Root Cause:
 The root cause was a configuration bottleneck in Nginx, which was set with a low worker_connections value (1024) and a worker_rlimit_nofile that couldn’t support the sudden spike in concurrent connections. Once the limit was breached, the Nginx worker died silently, unable to proxy requests to the Node.js backend, leading to the 502 errors.
Resolution:
 The team:
Updated nginx.conf to increase worker_connections to 4096 and worker_rlimit_nofile to 8192.


Enabled error logging at a more verbose level to catch future silent crashes.


Added a systemd watchdog to auto-restart Nginx if the worker dies.


Monitored memory usage post-deployment to confirm no further crashes.



Corrective and Preventative Measures
Improvements:
Increase resilience in the Nginx reverse proxy layer.


Improve visibility of silent failures in the stack.


Implement traffic rate limiting and load balancing for critical endpoints.


TODOs:
Increase worker_connections and worker_rlimit_nofile in Nginx.


Add systemd health check for Nginx worker processes.


Set up alerting for high memory and connection count.


Deploy rate-limiting middleware for API endpoints.


Introduce autoscaling logic for traffic spikes (horizontal scaling).


Run load tests weekly to simulate traffic patterns and improve readiness.


Document system failure recovery protocol for on-call engineers.

