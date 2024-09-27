**Issue Summary**  
**Duration**: The failure occurred on September 15th, 2024, from 10:30 AM to 12:00 PM UTC (1 hour 30 minutes).  
**Impact**: During the failure, 70% of users could not access the API backend, leading to failed data requests and unresponsive dashboards for real-time metrics. Managers reported severe delays in the application’s performance, with a significant portion experiencing total service disruption.  
**Root Cause**: A database connection misconfiguration caused a conflict between multiple clusters in the MongoDB environment, leading to inter-campaign data leakage and connection pooling exhaustion.

**Timeline**

- **10:30 AM**: Monitoring alert triggered, indicating abnormally high response times from the API.
- **10:32 AM**: On-call engineer investigates, notices 500 HTTP errors in the API logs and escalating response times.
- **10:35 AM**: Initial assumption: network issue affecting API to database connections. Network team is notified.
- **10:45 AM**: Further investigation shows no network anomalies. Attention shifts to MongoDB performance.
- **11:00 AM**: DevOps team checks connection pooling and notices that database connections are hitting the upper limit.
- **11:10 AM**: A misleading path: engineers assume this is due to an increase in user traffic. Traffic logs are reviewed, showing no significant spike.
- **11:25 AM**: Root cause identified: connection misconfiguration between database clusters causing requests to be routed to incorrect databases.
- **11:40 AM**: Temporary fix applied: manual re-routing of database connections and clearing of misrouted queries.
- **12:00 PM**: Full service restored after applying configuration updates to database connection handlers.

**Root Cause and Resolution**  
The root cause is a problem related to MongoDB connection management in a multi-tenant environment. The system was designed to manage separate databases for each campaign, but due to a misconfiguration in the Mongoose connection logic (`mongoose.createConnection` was incorrectly used), database instances for multiple campaigns began to leak into each other. Queries destined for one campaign were therefore routed to another campaign's database, creating data integrity problems and overloading some connection pools.

To solve this problem, Mongoose's connection logic was redesigned. Instead of sharing connection instances between requests, each campaign was assigned a separate, independent connection pool. This change ensured that database queries were routed to the right database instance without any overlap.

**Corrective and Preventative Measures**  

**Improvements**:
- Strengthen the database connection management logic to ensure campaign isolation.
- Improve the monitoring system to detect data leaks between databases earlier.
- Implement stricter load balancing and connection pooling rules to avoid overwhelming any single cluster.

**TODO**:
1. Patch Mongoose connection logic to ensure unique instances are created per campaign.
2. Add monitoring for connection pool saturation on each database cluster.
3. Increase alerting sensitivity to detect unusual routing patterns.
4. Perform routine load tests to assess connection limits under varying traffic conditions.
5. Redesign the application API to handle database errors elegantly and avoid cascading failures.

These changes will help prevent similar failures in the future, improving both system stability and user experience.
