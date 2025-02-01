

# **URL Shortener Service**

## **1) Overview**

The **URL Shortener Service** is a high-performance system designed to convert long URLs into short, easily shareable links and efficiently redirect users back to the original URLs.

---

## **2) Functional Requirements**
- Convert a **long URL** into a **short URL**.
- Retrieve the **original URL** from a **short URL** and redirect users.

## **3) Non-Functional Requirements**
- **Low Latency**: Ensure quick URL lookups and redirects.
- **High Availability**: Maintain service uptime and reliability.

---

## **4) System Assumptions**
1. **Traffic**: 1,000 URL shortening requests per second (~31.5 billion per year).
2. **Read/Write Ratio**: Read requests are **10x** the write requests (31.5 billion writes, 315 billion reads per year).
3. **Character Set for Short URLs**: `a-z`, `A-Z`, `0-9` (62 unique characters).
4. **Uniqueness**: Each user gets a **unique** short URL, even if multiple users shorten the same long URL.
5. **URL Expiry**: Short URLs remain valid for **10 years**.
6. **Capacity**: The system should handle **315 billion URL requests over 10 years**.
7. **ID Space**: 7-character combinations allow for **3.5 trillion unique URLs**.
8. **Storage Requirements**:
   - **Short URL**: 7 bytes
   - **Long URL**: 100 bytes
   - **User Metadata**: 500 bytes
   - **Total per request**: ~1 KB
   - **Total data for 315 billion requests**: ~315 TB

---

## **5) High-Level Architecture**
![Flow Diagram](https://github.com/Yashas-naidu/url_shortener/blob/main/flow_diagram.png)

### **Key Components**
- **Load Balancer**: Distributes incoming traffic across multiple servers.
- **Web Server**: Handles user requests and interacts with databases.
- **Database (NoSQL + SQL)**:
  - **NoSQL**: Stores large-scale URL mappings.
  - **SQL**: Ensures transactional integrity where needed.
- **Redis Cache (LRU, TTL)**: Caches frequently accessed URLs for quick retrieval.

---

## **6) API Design**
### **REST API Endpoints**
#### **1. Shorten a URL**
- **Endpoint**: `POST /api/url`
- **Request**:
  ```json
  {
    "url": "<INSERT_LONG_URL>"
  }
  ```
- **Response**:
  ```json
  {
    "shortUrl": "https://short.ly/abc123"
  }
  ```
- **Status Codes**:
  - `201` - URL shortened successfully.
  - `400` - Bad Request (invalid URL).
  - `409` - Conflict (URL already shortened).

#### **2. Retrieve Original URL**
- **Endpoint**: `GET /api/url/{shortUrlId}`
- **Response**:
  - Redirects to the original long URL.
- **Status Codes**:
  - `301` - Permanent Redirect (short URL maps to long URL).
  - `404` - Not Found (short URL does not exist).
  - `410` - Gone (short URL expired or deleted).

---

## **7) Database Design**
### **Schema**
| Column         | Type          | Description                     |
|---------------|--------------|---------------------------------|
| shortUrlId    | String (PK)   | Unique short URL identifier    |
| longUrl       | String        | Original long URL              |
| timestamp     | Timestamp     | Creation time                  |
| metadata      | JSON          | User-related metadata          |
| numberOfClicks | Integer      | Click count for analytics      |
| userId        | String (FK)   | User ID (if applicable)        |

### **Storage Considerations**
- **Fast reads** (10x more frequent than writes).
- **Replication & Sharding** for scalability.
- **Master-Slave Architecture** for redundancy.

### **Database Choices**
- **NoSQL** (for fast retrieval):
  - MongoDB (Document Store)
  - Cassandra (Column Store)
  - DynamoDB (Key-Value Store)
- **SQL** (for transactional integrity):
  - PostgreSQL or MySQL for controlled operations.

---

## **8) Ensuring URL Uniqueness**
1. **Hashing**: Generates unique IDs (possible collisions).
2. **Auto-increment**: Predictable IDs (not ideal).
3. **Custom Algorithm** (Best Approach): 
   - Generates 7-character unique keys.
   - Uses a boolean flag to track usage.

---

## **9) Security Measures**
- **Input Validation**: Prevents malformed/malicious URLs.
- **SQL Injection Protection**: Uses ORM to sanitize inputs.
- **DDoS Mitigation**: Implements **rate limiting** (Token Bucket Algorithm).
- **HTTPS Enforcement**: Secures data transmission.
- **Monitoring & Logging**: Tracks system activity and security threats.

---

## **10) Implementation Details**
### **Technology Stack**
- **Backend**: Java (Spring Boot)
- **Database**: Redis, PostgreSQL
- **Cache**: Redis (LRU + TTL)
- **Build Tool**: Gradle

### **Project Structure**
```
src/
├── common/
│   ├── IDConverter.java       # Generates and decodes short URLs
│   ├── URLValidator.java      # Validates URLs
│
├── controller/
│   ├── URLController.java     # Handles API requests
│
├── repository/
│   ├── URLRepository.java     # Manages Redis read/write logic
│
├── service/
│   ├── URLConverterService.java  # Shortening and retrieval logic
│
└── URLShortenerApplication.java  # Main application entry point
```

---

## **11) Running the Service**
### **Step 1: Start Redis Server**
```sh
redis-server
```

### **Step 2: Build the Project**
```sh
gradle build
```

### **Step 3: Run the Application**
```sh
gradle run
```

### **Step 4: Test API**
#### **Shorten a URL**
Send a `POST` request to:
```
http://localhost:8080/api/url
```
with JSON body:
```json
{
  "url": "https://example.com"
}
```

#### **Retrieve Original URL**
Access:
```
http://localhost:8080/api/url/abc123
```
It should **redirect** to `https://example.com`.

---

## **12) Future Enhancements**
- **Analytics Dashboard**: Track click rates and user engagement.
- **Custom Aliases**: Allow users to create personalized short links.
- **Expiration Policies**: Users can set custom expiry dates for URLs.
- **Geo-Location Based Redirection**: Redirect users based on region.
- **Load Balancing & Auto-Scaling**: Improve system reliability.

---

## **13) Conclusion**
This URL Shortener Service is **scalable, efficient, and secure**, leveraging **Redis for caching, NoSQL for storage, and Java Spring Boot** for rapid development. The system is designed to handle **billions of requests per year** while ensuring **low latency and high availability**.
