There are two apps - `client-app` and `processor-app`

This application is **client-app**

# **client-app (external)**

This application exposes a REST API to the user via tools like **Postman**.

### Functionality:
1. **User Registration**
   * Endpoint: `POST /register`
   * Accepts email and password.
   * Stores user in a database with encrypted password (e.g., BCrypt).
2. **Authentication**
   * Endpoint: `POST /login`
   * Accepts email and password.
   * If credentials are valid, returns a **JWT token**.
3. **Submit Message**
   * Endpoint: `POST /submit`
   * Requires `Authorization: Bearer <JWT>` header.
   * Accepts user input string (max N characters).
   * Sends the string to `processor-app` via internal REST call.
   * Includes user identification as part of the internal request (e.g., in a custom header or JWT).
4. **Receive Processed Result**
   * Subscribes to a **message broker** (e.g., Kafka/RabbitMQ).
   * Waits for messages in the form:
    ```
    {
        "userEmail": "user@example.com",
        "number": 42,
        "result": "result of string processing"
    }
    ```
   * Displays this result (in Postman response or logs, depending on implementation).

# **processor-app (internal only)**

This app is **not accessible publicly** and only receives requests from `client-app`.

### Functionality:

1. **Authorization**
   * Accepts only requests from trusted `client-app` (e.g., checks service token or shared secret).
   * Extracts `userEmail` from the request (or signed token).
2. **Message Validation**
   * Validates input (e.g., length ≤ N).
3. **Generates a random integer** (serves as request ID).
4. **Returns HTTP 200 OK** with the generated integer to `client-app`.
5. **Processes the message**
   * Reverses the input string (or something stupid like that).
6. **Saves to Database**
   * Table structure:
      * `id` (auto-increment primary key)
      * `generated_number` (int)
      * `user_email` (string)
      * `original_string` (string)
      * `reversed_string` (string)
7. **Sends Result to Message Broker**
   * Sends a message in the format:
   ```
    {
        "userEmail": "user@example.com",
        "number": 42,
        "result": "result of string processing"
    }
    ```

# **Security Architecture**

* **Users authenticate via JWT** in `client-app`.
* **client-app calls processor-app** using internal REST and **attaches user identity** (e.g., signed JWT or trusted headers).
* **processor-app verifies** that the request came from a trusted internal service.
* **processor-app has no external public API** — it is protected via firewall, VPN, or service token.

# **Future Optional Enhancements (for more practice)**
* Use Docker Compose to spin up both apps + a broker (Kafka or RabbitMQ) + DBs.
* Use Spring Security + OAuth2 or JWT manually.
* Write integration tests (RestAssured, Testcontainers).
* Add message deduplication or retry mechanism on broker level.
