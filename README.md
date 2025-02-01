# ***URL Shortener Service***
<br>


## 1) CONTEXT ON URL SERVICE :


## Functional Requirements
1. Given a long URL, convert it into a short URL.
2. Given a short URL, redirect it to the original long URL.

## Non-Functional Requirements
- **Low Latency**: Ensure fast response times.
- **High Availability**: Ensure the service is always accessible.

## Assumptions
1. **Assumption 1**: 1000 URL requests per second, totaling approximately 31.5 billion requests per year.
2. **Assumption 2**: Write requests = 31.5 billion, read requests = 315 billion (10x write requests).
3. **Assumption 3**: Characters used for short URLs: `a-z`, `A-Z`, `0-9` (62 characters total).
4. **Assumption 4**: If multiple users shorten the same long URL, each user gets a unique short URL.
5. **Assumption 5**: The system must generate unique URLs for 10 years.
6. **Assumption 6**: 315 billion URL shortening requests over 10 years.
7. **Assumption 7**: 7-character combinations allow for 3.5 trillion unique URLs.
8. **Assumption 8**: Storage requirements:
   - Short URL: 7 bytes
   - Long URL: 100 bytes
   - User metadata: 500 bytes
   - **Total**: ~1000 bytes per request
   - **Total data for 315 billion requests**: ~315 TB.

## High-Level Design
![Flow_diagram](https://github.com/Yashas-naidu/url_shortener/blob/main/flow_diagram.png)

- **LRU**: Least Recently Used
- **TTL**: Time to Live (URLs expire if unused within a fixed time)

## API Design
### REST API
#### POST `/api/url`
- **Input**: JSON payload with long URL.
- **Output**: JSON payload with shortened URL.
- **Status Codes**:
  - `201`: Success
  - `400`: Bad Request
  - `409`: Conflict (URL already shortened)

#### GET `/api/url/{shortUrlId}`
- **Input**: Short URL ID (e.g., `abc12345`).
- **Output**: Redirect to long URL.
- **Status Codes**:
  - `301`: Permanently Moved (Short URL maps to long URL)
  - `404`: Not Found (Short URL not found)
  - `410`: Gone (Short URL expired or deleted)

## Database
### Schema
- `shortUrlId` (Primary Key)
- `longUrl`
- `timestamp`
- `metadata`
- `numberOfClicks`
- `userId`

### Requirements
- Fast read (10x write) and write operations.
- **NoSQL Options**:
  1. MongoDB (Document Store)
  2. Cassandra (Column Store)
  3. DynamoDB (Key-Value Store)
- **Replication and Sharding**: For high availability and fault tolerance.
- **Master-Slave Architecture**: For failover and recovery.

### SQL Use Case
- Required for transaction isolation when multiple users access the same short URL.

### Methods to Ensure Uniqueness
1. **Hashing**: Prone to collisions.
2. **Auto-increment**: Predictable unique IDs.
3. **Custom Algorithm**: Best approach. Generate 7-character combinations, store as keys, and use Booleans to track usage.

## Security
- Validate URLs to prevent malicious content.
- Sanitize URLs to prevent SQL injection.
- Set rate limits to prevent DDoS attacks (Token Bucket Algorithm).
- Use HTTPS for encrypted communication.
- Implement monitoring and logging for security and debugging.

## 2) Implementation 

<h2>A Java Service built with Spring Boot, and Redis.</h1>

<h3>common</h3>
<b>IDConverter.java</b> <br />
A Singleton class responsible for: <br />
1. Generating ID <br />
2. Using ID to create unique URL ID <br />
3. Using unique URL ID to retrieve original ID <br />
<br /> <br />
<b> URLValidator.java</b> <br />
A Singleton class responsible for validating URL's validity

<h3>controller</h3>
<b>URLController.java</b> <br />
A Spring Boot Controller responsible for: <br/>
1. Serving an endpoint to shorten URL <br />
2. Redirect shortened URL to the original URL <br />

<h2>repository</h3>
<b>URLRepository</b> <br />
A Java class responsible for abstracting Redis(database) read/write logic

<h2>service</h3>
<b>URLConverterService.java</b> <br />
A Java class used to abstract URL Shortening and URL Retrieval process
<br />
<b>URLShortenerApplication.java</b> <br />
The entry point for the Spring application
<br /> <br />
<h2>To run:</h2>
1. Start up Redis' Server

```
redis-server
```

2. Build the project

```
gradle build
```


3. Run the project

```
gradle run
```

<br />
By default the Server will run on localhost:8080/shortener <br/>
To test, send POST Request to localhost:8080/shortener with a body of type application/json with body 

```
{
  'url' : '<INSERT URL>'
}
```

