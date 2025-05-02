1. High-Level Architecture
Components:
•	Client: Web or mobile user accessing the service.
•	Frontend (S3 + CloudFront): Static frontend hosted on S3 and distributed via CloudFront CDN.
•	API Gateway: Routes incoming API requests to the respective Lambda functions.
•	Lambda (Shorten URL): Generates a short URL and stores it.
•	Lambda (Redirect URL): Fetches the original URL using short code and redirects.
•	DynamoDB: Stores mappings of short code to original URLs.
•	Redis (ElastiCache): Caching layer to improve performance of redirects.

2. Request Flow
Shorten URL
1.	Client makes a POST request to /shorten via the frontend.
2.	API Gateway invokes the Shorten URL Lambda function.
3.	Lambda function:
o	Validates the URL.
o	Generates a unique short code.
o	Stores { shortCode, originalURL, timestamp } in DynamoDB.
4.	Returns the full shortened URL to the client.
Redirect URL
1.	Client accesses a shortened URL (/{shortCode}).
2.	API Gateway routes to Redirect URL Lambda.
3.	Lambda function:
o	Looks up shortCode in Redis cache.
o	If not found, queries DynamoDB and updates Redis.
o	Issues a 302 redirect to the original URL.

3. AWS Services Used
Amazon S3
•	Hosts the static frontend application.
Amazon CloudFront
•	Distributes frontend globally with low latency.
Amazon API Gateway
•	Defines REST API with endpoints:
o	POST /shorten
o	GET /{shortCode}
AWS Lambda
•	Shorten URL Lambda:
o	Generates a base62 encoded hash.
o	Stores the mapping in DynamoDB.
•	Redirect URL Lambda:
o	Checks Redis for shortCode.
o	Fallback to DynamoDB.
o	Updates Redis and redirects.
Amazon DynamoDB
•	Schema-less NoSQL table.
•	Stores records: { shortCode, originalURL, createdAt }
Redis (Amazon ElastiCache)
•	Caches { shortCode: originalURL } for fast redirection.
•	TTL configured to balance freshness and cache hit ratio.

4. Security and Monitoring
•	API Gateway usage plans and throttling.
•	IAM roles for Lambda access control.
•	AWS CloudWatch for logs and alerts.

5. Improvements and Future Scope
•	Add custom short links.
•	Add expiry time for short URLs.
•	Add analytics (clicks, geo data).
•	Track usage via AWS X-Ray and CloudWatch Metrics.


