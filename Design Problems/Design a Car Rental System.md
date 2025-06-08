
### Functional Requirements

1. Users can create an account, log in, and manage their profile. (Customers, Admins) - **User Registration and Authentication Service**
2. Users can search for available cars based on location, date, and car type. (Customers) -  **Car Search and Availability Service**
3. Users can book a car for a specified period. (Customers) - **Booking and Reservation Service** 
4. Users can pay for their car rental using various payment methods. (Customers) (Strategy Pattern ) - **Payment Processing Service**
5. Admins can add, update, or remove cars from the inventory. (Admins) - **Car Management Service**
6. Users can view their past rentals and provide feedback. (Customers) - **Rental History and Feedback**

### Non Functional Requirements

1. **Scalability of User Management**
    
    - **Issue:** As the number of users grows, the system must efficiently handle increased registration, authentication, and profile management requests.
    - **Solution:** Implement a distributed authentication service with load balancing to manage user sessions and scale horizontally as user demand increases.

2. **Car Inventory Management**
    
    - **Issue:** The system should efficiently manage a growing inventory of cars across multiple locations.
    - **Solution:** Use a distributed database system that supports sharding to partition the car inventory data by location, ensuring quick access and updates.

3. **Reservation and Booking System**
    
    - **Issue:** High traffic during peak times can lead to bottlenecks in the reservation process.
    - **Solution:** Implement a queuing system to manage booking requests and ensure that the reservation service can scale horizontally to handle peak loads.

4. **Payment Processing Flexibility**
    
    - **Issue:** The system should support multiple payment methods and adapt to new ones as they become popular.
    - **Solution:** Use a modular payment gateway integration that allows easy addition of new payment methods without significant changes to the core system.

5. **API Versioning and Extensibility**
    
    - **Issue:** As the system evolves, new features and changes to existing APIs should not disrupt current users.
    - **Solution:** Implement API versioning to allow backward compatibility and facilitate the introduction of new features without affecting existing clients.


### Tradeoffs

1. **Complexity vs. Simplicity in Design**
    
    - **Trade-off:** A more complex design can offer greater flexibility and scalability but may increase development time and maintenance overhead.
    - **Application:** Opt for a modular design that balances complexity with simplicity, allowing for easy updates and maintenance while supporting future growth.

2. **Performance vs. Consistency**
    
    - **Trade-off:** Ensuring high performance might require sacrificing strict consistency, especially in distributed systems.
    - **Application:** Implement eventual consistency for non-critical operations like feedback submission, while maintaining strong consistency for critical operations like payment processing.

3. **Cost vs. Availability**
    
    - **Trade-off:** High availability often requires additional resources, increasing operational costs.
    - **Application:** Use cloud-based solutions with auto-scaling capabilities to manage costs while ensuring the system remains available during peak times.

4. **Security vs. Usability**
    
    - **Trade-off:** Enhancing security measures can sometimes make the system less user-friendly.
    - **Application:** Implement multi-factor authentication for sensitive operations while ensuring the login process remains straightforward for users.

5. **Data Redundancy vs. Storage Efficiency**
    
    - **Trade-off:** Storing redundant data can improve read performance but may lead to increased storage costs.
    - **Application:** Use data caching for frequently accessed information like car availability, while ensuring that the primary database remains optimized for storage efficiency.

### Entities 


6. **User**
    
    - **Attributes:**
        - `userId`: String
        - `name`: String
        - `email`: String
        - `password`: String
        - `phoneNumber`: String
        - `address`: String
        - `role`: Enum (Customer, Admin)
7. **Car**
    
    - **Attributes:**
        - `carId`: String
        - `make`: String
        - `model`: String
        - `year`: Integer
        - `type`: Enum (Sedan, SUV, Truck, etc.)
        - `location`: String
        - `availabilityStatus`: Boolean
        - `rentalPricePerDay`: Float

8. **Reservation**
    
    - **Attributes:**
        - `reservationId`: String
        - `userId`: String
        - `carId`: String
        - `startDate`: Date
        - `endDate`: Date
        - `totalPrice`: Float
        - `status`: Enum (Pending, Confirmed, Cancelled)

9. **Payment**
    
    - **Attributes:**
        - `paymentId`: String
        - `reservationId`: String
        - `amount`: Float
        - `paymentMethod`: Enum (Credit Card, PayPal, etc.)
        - `paymentDate`: Date
        - `status`: Enum (Completed, Failed)

10. **Feedback**
    
    - **Attributes:**
        - `feedbackId`: String
        - `userId`: String
        - `reservationId`: String
        - `rating`: Integer
        - `comments`: String
        - `date`: Date


### Entity Relationship

11. **User - Reservation**
    
    - **Relationship:** One-to-Many
    - **Use Case:** A user can make multiple reservations over time.
12. **User - Feedback**
    
    - **Relationship:** One-to-Many
    - **Use Case:** A user can provide feedback for each reservation they have completed.
13. **Car - Reservation**
    
    - **Relationship:** One-to-Many
    - **Use Case:** A car can be reserved by different users at different times, but only one reservation can be active at a time.

14. **Reservation - Payment**
    
    - **Relationship:** One-to-One
    - **Use Case:** Each reservation is associated with a single payment transaction.
15. **Admin - Car**
    
    - **Relationship:** Many-to-Many (through actions)
    - **Use Case:** Admins can manage multiple cars, and each car can be managed by multiple admins for updates and maintenance.

16. **User - Payment**
    
    - **Relationship:** One-to-Many
    - **Use Case:** A user can make multiple payments for different reservations.



### API Design - Refer Functional Requirements


17. **User Registration and Authentication**

    - **Endpoint:** `POST /api/users/register`
        - **Input:** `{ "name": "string", "email": "string", "password": "string", "phoneNumber": "string", "address": "string" }`
        - **Output:** `{ "userId": "string", "message": "User registered successfully" }`
    - **Endpoint:** `POST /api/users/login`
        - **Input:** `{ "email": "string", "password": "string" }`
        - **Output:** `{ "userId": "string", "token": "string" }`

Note - Password will be hashed at the time of saving using Bcrypt or adding a salt.

18. **Car Search and Availability**

    - **Endpoint:** `GET /api/cars`
        - **Input:** `?location=string&startDate=date&endDate=date&type=string`
        - **Output:** `[ { "carId": "string", "make": "string", "model": "string", "year": "integer", "type": "string", "rentalPricePerDay": "float" } ]`

19. **Booking and Reservation**
    
    - **Endpoint:** `POST /api/reservations`
        - **Input:** `{ "userId": "string", "carId": "string", "startDate": "date", "endDate": "date" }`
        - **Output:** `{ "reservationId": "string", "totalPrice": "float", "status": "string" }`
20. **Payment Processing**
    
    - **Endpoint:** `POST /api/payments`
        - **Input:** `{ "reservationId": "string", "amount": "float", "paymentMethod": "string" }`
        - **Output:** `{ "paymentId": "string", "status": "string" }`

21. **Car Management**
    
    - **Endpoint:** `POST /api/cars`
    
        - **Input:** `{ "make": "string", "model": "string", "year": "integer", "type": "string", "location": "string", "rentalPricePerDay": "float" }`
        - **Output:** `{ "carId": "string", "message": "Car added successfully" }`

    - **Endpoint:** `PUT /api/cars/{carId}`

        - **Input:** `{ "make": "string", "model": "string", "year": "integer", "type": "string", "location": "string", "rentalPricePerDay": "float" }`
        - **Output:** `{ "message": "Car updated successfully" }`

22. **Rental History and Feedback**
    
    - **Endpoint:** `GET /api/users/{userId}/reservations`
        
        - **Input:** `None`
        - **Output:** `[ { "reservationId": "string", "carId": "string", "startDate": "date", "endDate": "date", "status": "string" } ]`

    - **Endpoint:** `POST /api/feedback`
        
        - **Input:** `{ "userId": "string", "reservationId": "string", "rating": "integer", "comments": "string" }`
        - **Output:** `{ "feedbackId": "string", "message": "Feedback submitted successfully" }`



### Request Flow to create high level Design


23. **User Registration and Authentication**
    
    - **Flow:**
        1. User sends a registration request to `POST /api/users/register` with required details.
        2. System validates input and creates a new user record.
        3. System responds with a success message and user ID.
        4. For login, user sends credentials to `POST /api/users/login`.
        5. System verifies credentials and returns a token if successful.
    - **Failure Conditions:**
        - Invalid input data results in an error message.
        - Incorrect login credentials return an authentication error.
24. **Car Search and Availability**
    
    - **Flow:**
        1. User sends a search request to `GET /api/cars` with filters.
        2. System queries the database for available cars matching criteria.
        3. System returns a list of available cars.
    - **Failure Conditions:**
        - No cars available results in an empty list.
        - Invalid query parameters return an error message.

25. **Booking and Reservation**
    
    - **Flow:**
        1. User sends a booking request to `POST /api/reservations` with car and date details.
        2. System checks car availability and calculates total price.
        3. System creates a reservation record and returns confirmation.

    - **Failure Conditions:**
        - Car not available results in a booking error.
        - Invalid date range returns an error message.

26. **Payment Processing**
    
    - **Flow:**
        1. User sends a payment request to `POST /api/payments` with reservation ID and payment details.
        2. System processes the payment through the selected method.
        3. System updates reservation status and returns payment confirmation.
    - **Failure Conditions:**
        - Payment failure results in a transaction error.
        - Invalid payment details return an error message.
27. **Car Management**
    
    - **Flow:**
        1. Admin sends a request to `POST /api/cars` to add a new car.
        2. System validates input and adds the car to the inventory.
        3. System responds with a success message.
        4. For updates, admin sends a request to `PUT /api/cars/{carId}`.
        5. System updates car details and confirms the update.
    - **Failure Conditions:**
        - Invalid car details result in an error message.
        - Car ID not found returns an update error.
28. **Rental History and Feedback**
    
    - **Flow:**
        1. User requests rental history via `GET /api/users/{userId}/reservations`.
        2. System retrieves and returns the user's past reservations.
        3. User submits feedback through `POST /api/feedback`.
        4. System records feedback and returns a confirmation.

    - **Failure Conditions:**

        - No past reservations result in an empty list.
        - Invalid feedback input returns an error message.


### Failure Scenario Analysis

29. **User Authentication Failure**
    
    - **Failure Point:** Users are unable to log in due to authentication service downtime.
    - **Solution:** Implement a failover mechanism with a secondary authentication server to ensure continuous access.

30. **Database Connection Loss**
    
    - **Failure Point:** The system cannot access the database, affecting all operations.
    - **Solution:** Use a database cluster with automatic failover and replication to maintain data availability and integrity.

31. **Payment Gateway Unavailability**
    
    - **Failure Point:** Payment processing fails due to third-party gateway issues.
    - **Solution:** Integrate multiple payment gateways and implement a fallback mechanism to switch to an alternative provider if the primary one fails.

32. **Reservation Overbooking**
    
    - **Failure Point:** Multiple users book the same car simultaneously, leading to overbooking.
    - **Solution:** Implement a locking mechanism during the booking process to ensure that a car is reserved for only one user at a time.

33. **API Rate Limiting Exceeded**
    
    - **Failure Point:** High traffic causes API rate limits to be exceeded, resulting in denied requests.
    - **Solution:** Implement rate limiting with a retry mechanism and provide users with clear error messages and guidance on retrying after a cooldown period


### Things to consider

34. Use Blob storage like AWS S3 for storing images and videos.
35. Use AWS API Gateway for rate limiting, caching and authentication purposes.
36. Use Distributed Cache like redis for locking mechanism.
37. Use RDBMS, for payment processing.
38. Use NoSQL for storing data like car, user and feedback details.
39. Capacity Estimations.

### References

https://bugfree.ai/system-design/car-rental-system


## **3. System Workflow**

### **Car Reservation Process**

40. **User selects a car and time window** (e.g., 10 AM - 12 PM).
41. **System checks availability**:
    - Queries Redis cache.
    - If not in cache, checks the database.
42. **If available**, the system:
    - Creates a reservation entry in the database.
    - Updates the availability status in Redis.
43. **User confirmation**:
    - The system locks the reservation for a short time (e.g., 5 minutes).
    - If the user confirms within the time limit, the reservation is finalized.
    - If the user doesn’t confirm, the slot is released.
44. **Send notifications** (confirmation, reminders).

---

## **4. Handling Concurrency (Race Conditions)**

### **Optimistic Locking (Versioning)**

- Use a `version` column in the database.
- If two users try booking the same car, only one succeeds.

### **Pessimistic Locking**

- Lock a row when a booking attempt is in progress.
- Ensures only one transaction modifies the reservation at a time.

### **Redis-based Locking**

- Use **Redis SETNX (Set if Not Exists)** to create a distributed lock for time slots.

## **How Redis SETNX Works for Distributed Locking?**

1. **Attempt to Acquire Lock**

    - When a user tries to reserve a car, the system executes:


```bash
SETNX car_lock:<car_id> <timestamp>
```

- - If the key (`car_lock:<car_id>`) **does not exist**, it is set, meaning the car is now locked for booking.

- **Set Expiry for the Lock**
    
    - To ensure the lock doesn’t stay forever in case of failure, set an expiration time:


```shell
EXPIRE car_lock:<car_id> 5
```

- - This ensures the lock auto-releases after **5 seconds** if the transaction doesn’t complete.
- **Handle Lock Failures (Another User Already Booked)**
    
    - If `SETNX` fails (meaning the key already exists), another user already locked the car.
    - The system can **retry after a short delay** or **show an error message**.
- **Release the Lock on Successful Booking**
    
    - After confirming the booking, remove the lock:

```bash
DEL car_lock:<car_id>
```

This allows the next reservation attempt.


```python
import redis
import time

redis_client = redis.StrictRedis(host='localhost', port=6379, db=0)

def acquire_lock(car_id, ttl=5):
    lock_key = f"car_lock:{car_id}"
    timestamp = str(time.time())

    if redis_client.setnx(lock_key, timestamp):  # Try to acquire the lock
        redis_client.expire(lock_key, ttl)  # Set expiry to prevent deadlocks
        return True
    return False  # Lock already held

def release_lock(car_id):
    lock_key = f"car_lock:{car_id}"
    redis_client.delete(lock_key)  # Remove lock

# Example usage
car_id = "123"
if acquire_lock(car_id):
    print("Car reserved, processing booking...")
    time.sleep(3)  # Simulate processing time
    release_lock(car_id)
    print("Booking confirmed, lock released.")
else:
    print("Car is already being reserved by another user.")

```

### Capacity Estimations

Capacity estimation for a **Car Rental Application** depends on multiple factors, including the number of users, concurrent requests, storage requirements, and expected growth. Below is a structured approach to estimating capacity:

---

## 1. **Understanding Requirements**

- **Users:**
    - Daily Active Users (DAU) = ~100K (example)
    - Peak concurrent users = ~10% of DAU → 10K users
- **Requests:**
    - Average API calls per user = 20 per session
    - Peak concurrent requests = 10K × 20 = **200K requests per second (RPS)**
- **Storage:**
    - Number of cars = 50K
    - Bookings per month = 1M
    - Each booking entry ~1KB → 1TB per year

---

## 2. **Traffic Estimation**

- **Read-heavy operations (~80%)**: Searching for cars, availability checks
- **Write operations (~20%)**: Booking a car, cancellations, user registrations

**Assumption: 200K RPS at peak**

- **Read requests (~80%)**: 160K RPS → Can use **caching (Redis, CDN)**
- **Write requests (~20%)**: 40K RPS → Requires strong **database scaling**

---
## 3. **Database Capacity**

- **Read Operations**: Use **replicated databases (Read Replicas)** or caching
- **Write Operations**: Sharding strategy for **bookings, payments**
- **Storage Needs**:
    - **Bookings Table (~1M entries per month)** → 1 TB per year
    - **Users Table (~10M users)** → 10 GB (Assume each record ~1KB)
    - **Cars Table (~50K records)** → ~500 MB

**Database choice:** PostgreSQL (for consistency) or DynamoDB (for scalability)

---

## 4. **Server & Scaling**

- **Application Servers**
    - Assume each server handles **5K RPS**
    - Needed servers = **200K RPS / 5K RPS = 40 servers**
    - Deploy in **autoscaling groups** (AWS EC2, Kubernetes)

- **Load Balancing**
    - AWS ALB / Nginx to distribute traffic

- **Caching**
    - Redis/Memcached for frequently accessed data (e.g., car availability)

---
## 5. **Network & CDN**

- **Static Content (Images, Car Photos)**: Serve via **CDN (Cloudflare, AWS CloudFront)**
- **API Rate Limiting**: Prevent abuse by limiting high-frequency requests.

---

## 6. **Backup & Disaster Recovery**

- **Database Backups**: Daily snapshots, log shipping
- **Multi-Region Deployment**: Use Active-Passive setup

---

## Conclusion

For a high-traffic car rental system, you need:

- **40–50 application servers** with autoscaling
- **Distributed databases with sharding**
- **Caching for reads (Redis)**
- **Load balancers for traffic management**
- **CDN for static content**