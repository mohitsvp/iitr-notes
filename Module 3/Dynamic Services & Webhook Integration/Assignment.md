# Dynamic Services & Webhook Integration: Assignment

## Assignment Overview

This assignment tests your understanding of designing robust, fault-tolerant webhooks for event-driven agent systems. You'll work through scenarios that require you to apply webhook best practices, implement security measures, and handle real-world failure scenarios.

---

## SECTION 1: EASY QUESTIONS (Knowledge & Recall)

### Question 1 (MCQ)
What is the primary advantage of using webhooks over polling in event-driven systems?

**A)** Webhooks require less storage space on the server  
**B)** Webhooks use a push-based model, delivering data immediately when events occur, reducing latency and resource consumption  
**C)** Webhooks are easier to implement than polling  
**D)** Webhooks eliminate the need for authentication

**Answer:** B

**Explanation:** Webhooks follow a push-based delivery model where data is sent automatically when an event happens, as opposed to polling's pull-based model where a client repeatedly asks for updates. This reduces unnecessary network calls, lowers latency, and improves efficiency. Polling requires continuous checking even when no events occur, wasting resources.

**Why other options are incorrect:**
- **A)** Storage space isn't a primary advantage of webhooks
- **C)** Implementation complexity varies; this isn't the main benefit
- **D)** Both webhooks and polling require authentication

---

### Question 2 (MCQ)
In webhook systems, what does "idempotency" mean?

**A)** The ability to send webhooks at different speeds  
**B)** Processing the same webhook request multiple times produces the same result as processing it once  
**C)** The webhook signature is encrypted using HMAC  
**D)** The webhook automatically retries on failure

**Answer:** B

**Explanation:** Idempotency ensures that if a webhook is delivered multiple times (due to retries or network issues), processing it repeatedly won't cause duplicate side effects or data corruption. For example, if a payment webhook is received three times, your system should only create one payment record, not three.

**Why other options are incorrect:**
- **A)** Speed of delivery is unrelated to idempotency
- **C)** HMAC signing is a security measure, not idempotency
- **D)** Retries are a delivery mechanism; idempotency is about handling duplicates

---

### Question 3 (MCQ)
Which HTTP status code should a webhook receiver return when it successfully processes a webhook?

**A)** 201 Created  
**B)** 202 Accepted  
**C)** 200 OK  
**D)** 204 No Content

**Answer:** C

**Explanation:** The webhook receiver should return **200 OK** to indicate successful processing. This tells the webhook sender that the event was received and handled correctly. While 201, 202, and 204 are valid HTTP responses in other contexts, 200 is the standard success response for webhook acknowledgment.

**Why other options are incorrect:**
- **A)** 201 Created is used when a new resource is created, not for webhook acknowledgment
- **B)** 202 Accepted indicates asynchronous processing; for webhooks, 200 is preferred
- **D)** 204 No Content is used when there's no response body to return

---

### Question 4 (MCQ)
What is the purpose of including a timestamp in webhook request headers?

**A)** To identify which user triggered the event  
**B)** To prevent replay attacks by validating that the request was sent within an acceptable time window  
**C)** To track how long the webhook took to process  
**D)** To determine the timezone of the webhook sender

**Answer:** B

**Explanation:** Timestamps in webhook headers serve as a security mechanism. When you validate that a request was sent within a recent time window (e.g., within the last 5 minutes), you prevent attackers from replaying old webhook requests. Combined with HMAC signatures, timestamp validation creates a robust defense against replay attacks.

**Why other options are incorrect:**
- **A)** User identification is handled by authentication, not timestamps
- **C)** Processing time is measured on the receiver's end, not determined by headers
- **D)** Timezone information isn't the primary purpose of webhook timestamps

---

### Question 5 (MSQ - Multiple Select)
Which of the following are essential components of a secure webhook system? (Select all that apply)

**A)** HMAC signature verification  
**B)** Timestamp validation  
**C)** TLS/HTTPS encryption  
**D)** Storing webhook secrets in plain text  
**E)** Unique event IDs for deduplication

**Answer:** A, B, C, E

**Explanation:**
- **A)** HMAC signatures verify that the webhook hasn't been tampered with and comes from the expected sender
- **B)** Timestamp validation prevents replay attacks
- **C)** HTTPS ensures data is encrypted in transit
- **E)** Unique event IDs allow your system to detect and ignore duplicate deliveries

**D is incorrect** because storing secrets in plain text is a critical security vulnerability. Secrets should be encrypted and stored securely.

---

### Question 6 (MCQ)
What is exponential backoff in the context of webhook retries?

**A)** Sending all retry attempts immediately, one after another  
**B)** Increasing the delay between each retry attempt, giving the failing endpoint time to recover  
**C)** Retrying only once, then giving up  
**D)** Using a fixed delay between all retry attempts

**Answer:** B

**Explanation:** Exponential backoff means that each retry attempt has a progressively longer delay. For example: retry after 1 second, then 2 seconds, then 4 seconds, then 8 seconds. This approach gives a temporarily unavailable endpoint time to recover without overwhelming it with requests. It's much more effective than flooding a struggling server with immediate retries.

**Why other options are incorrect:**
- **A)** Immediate retries would overwhelm a struggling endpoint
- **C)** Retrying only once defeats the purpose of retry logic
- **D)** Fixed delays don't adapt to recovery time and can still overwhelm endpoints

---

### Question 7 (MCQ)
In HMAC signature verification, what does the "secret key" represent?

**A)** A public key that anyone can access  
**B)** A shared secret known only to the webhook sender and receiver, used to generate and verify signatures  
**C)** The password of the webhook endpoint  
**D)** A temporary token that changes with each request

**Answer:** B

**Explanation:** The secret key is a shared credential between the webhook sender and receiver. The sender uses it to generate an HMAC signature, and the receiver uses the same key to recalculate the signature and verify it matches. If they match, you know the request came from the expected sender and hasn't been modified. This key must be kept confidential.

**Why other options are incorrect:**
- **A)** Public keys are used in asymmetric cryptography, not HMAC
- **C)** Endpoint passwords are separate from HMAC secrets
- **D)** The secret key is static; it doesn't change per request (the signature does)

---

### Question 8 (MCQ)
What is the main benefit of using a message queue (like RabbitMQ or AWS SQS) in webhook architectures?

**A)** It makes webhooks faster  
**B)** It decouples webhook ingestion from processing, allowing you to handle spikes in traffic and process events asynchronously  
**C)** It eliminates the need for retry logic  
**D)** It prevents all security threats

**Answer:** B

**Explanation:** A message queue acts as a buffer between receiving webhooks and processing them. When many webhooks arrive at once, they're queued and processed at a manageable rate. This decoupling improves resilience: if your processing service is temporarily down, webhooks are safely stored in the queue until it recovers.

**Why other options are incorrect:**
- **A)** Queues add a small latency; they don't make webhooks faster
- **C)** Queues complement retry logic but don't replace it
- **D)** Queues improve reliability but aren't a security solution

---

### Question 9 (MSQ - Multiple Select)
Which of the following are valid reasons to NOT retry a webhook? (Select all that apply)

**A)** The webhook receiver returned a 400 Bad Request error  
**B)** The webhook receiver returned a 500 Internal Server Error  
**C)** The webhook receiver returned a 401 Unauthorized error  
**D)** The webhook receiver returned a 429 Too Many Requests error  
**E)** The webhook receiver returned a 404 Not Found error

**Answer:** A, C, E

**Explanation:**
- **A)** 400 Bad Request indicates a client error (malformed request). Retrying won't help; the request itself is invalid.
- **C)** 401 Unauthorized suggests the credentials or signature is wrong. Retrying the same request won't fix this.
- **E)** 404 Not Found means the endpoint doesn't exist. Retrying won't help.

**B and D should be retried:**
- **B)** 500 Internal Server Error is a temporary server issue; the endpoint may recover
- **D)** 429 Too Many Requests means the endpoint is rate-limited; retrying after a delay is appropriate

---

### Question 10 (MCQ)
What is a "processed_webhooks" table used for in idempotent webhook systems?

**A)** To store the final results of webhook processing  
**B)** To track which webhook events have already been processed, preventing duplicate processing  
**C)** To log all errors that occurred during webhook processing  
**D)** To store the HMAC secrets for each webhook

**Answer:** B

**Explanation:** A `processed_webhooks` table maintains a record of unique webhook event IDs that have been successfully processed. When a duplicate webhook arrives (same event ID), your system checks this table, sees the event was already processed, and returns success without processing it again. This is the core mechanism for achieving idempotency.

**Why other options are incorrect:**
- **A)** Results are stored in domain-specific tables, not a dedupe table
- **C)** Errors are logged separately; this table tracks processed events
- **D)** Secrets are stored securely in a separate configuration system

---

### Question 11 (MCQ)
In event-driven agent systems, what role do webhooks play?

**A)** They replace the need for agents entirely  
**B)** They provide a mechanism for external systems to notify agents of events in real-time, triggering agent actions  
**C)** They only work with human-triggered events  
**D)** They are slower than polling and should be avoided

**Answer:** B

**Explanation:** Webhooks enable external systems (like payment processors, databases, or IoT devices) to push events directly to your agent system. When an event occurs, the webhook is triggered, notifying your agent to take action. This is ideal for event-driven architectures where agents need to respond immediately to external changes.

**Why other options are incorrect:**
- **A)** Webhooks and agents are complementary, not replacements
- **C)** Webhooks work with any event, automated or human-triggered
- **D)** Webhooks are faster and more efficient than polling

---

### Question 12 (MSQ - Multiple Select)
Which of the following are best practices for webhook endpoint design? (Select all that apply)

**A)** Endpoints should process webhooks synchronously and block until all side effects are complete  
**B)** Endpoints should acknowledge receipt quickly and process asynchronously  
**C)** Endpoints should validate the HMAC signature before processing  
**D)** Endpoints should store the raw webhook payload for audit trails  
**E)** Endpoints should use a weak or no timeout for processing

**Answer:** B, C, D

**Explanation:**
- **B)** Quick acknowledgment (200 OK) followed by async processing prevents timeouts and improves reliability
- **C)** Signature validation ensures security and authenticity
- **D)** Storing raw payloads aids debugging and compliance

**A is incorrect** because synchronous processing risks timeouts. **E is incorrect** because weak timeouts can cause resources to hang indefinitely.

---

### Question 13 (MCQ)
What does "at-least-once delivery" mean in webhook systems?

**A)** Webhooks are guaranteed to arrive exactly once, no more, no less  
**B)** Webhooks will be delivered at least one time, but may be delivered multiple times due to retries  
**C)** Webhooks will arrive within one hour  
**D)** Webhooks are optional and may not arrive at all

**Answer:** B

**Explanation:** "At-least-once delivery" is a reliability guarantee: your webhook will definitely be delivered, but due to retries and network conditions, it might arrive multiple times. This is why idempotency is critical—you must handle duplicates gracefully.

**Why other options are incorrect:**
- **A)** Exactly-once delivery is harder to guarantee and isn't the standard for webhooks
- **C)** Timing is separate from delivery guarantees
- **D)** At-least-once is a strong guarantee, not optional

---

### Question 14 (MCQ)
When designing a webhook system for agents, why is circuit breaker pattern useful?

**A)** To encrypt webhook payloads  
**B)** To stop sending webhooks to an endpoint that's consistently failing, preventing wasted resources and cascading failures  
**C)** To speed up webhook delivery  
**D)** To eliminate the need for authentication

**Answer:** B

**Explanation:** A circuit breaker monitors webhook delivery success rates. If an endpoint fails repeatedly (e.g., 5 failures in a row), the circuit breaker "opens" and stops sending webhooks to that endpoint temporarily. This prevents your system from wasting resources on a clearly unavailable endpoint and protects it from cascading failures. After a timeout, the circuit breaker "closes" and tries again.

**Why other options are incorrect:**
- **A)** Circuit breakers don't handle encryption
- **C)** Circuit breakers add a small delay; they don't speed things up
- **D)** Circuit breakers are about delivery reliability, not authentication

---

### Question 15 (MSQ - Multiple Select)
Which of the following should be included in a webhook event payload? (Select all that apply)

**A)** Unique event ID  
**B)** Event type or category  
**C)** Timestamp of when the event occurred  
**D)** The receiver's password  
**E)** Relevant event data  
**F)** API version or schema version

**Answer:** A, B, C, E, F

**Explanation:**
- **A)** Enables deduplication and tracking
- **B)** Helps receivers route and handle different event types
- **C)** Useful for ordering, replay, and debugging
- **E)** The actual information the receiver needs
- **F)** Allows receivers to handle schema changes gracefully

**D is incorrect** because passwords should never be included in webhook payloads. Authentication is handled separately via signatures and headers.

---

## SECTION 2: MODERATE QUESTIONS (Implementation & Application)

### Question 16 (Subjective - Code Implementation)

**Scenario:** You're building a payment processing webhook receiver for an e-commerce platform. The webhook receives order payment events from a payment provider. Your system needs to:
1. Verify the HMAC signature to ensure authenticity
2. Check for duplicate events using an event ID
3. Process the payment asynchronously
4. Return a 200 OK response immediately

**Task:** Write a Node.js/JavaScript function that implements a webhook endpoint handler. Your code should:
- Extract and verify the HMAC signature from headers
- Check if the event has already been processed
- Queue the event for async processing (you can use a mock queue)
- Return appropriate HTTP responses

Assume:
- `req.headers['x-signature']` contains the HMAC signature
- `req.headers['x-event-id']` contains the unique event ID
- `req.body` is the JSON payload
- You have a `processedEvents` Set to track processed IDs
- `WEBHOOK_SECRET` is available as an environment variable
- `crypto` module is available for HMAC verification

**Expected Output:** A complete, production-ready function with error handling.

---

**Sample Input:**
```javascript
// Mock request object
const mockReq = {
  headers: {
    'x-event-id': 'evt_12345',
    'x-signature': 'sha256=abcd1234...',
    'x-timestamp': '1704067200'
  },
  body: {
    orderId: 'order_999',
    amount: 9999,
    status: 'completed'
  }
};
```

**Expected Behavior:**
- First request: Process the event, return 200
- Duplicate request (same event ID): Return 200 without reprocessing
- Invalid signature: Return 403 Forbidden
- Processing error: Return 202 Accepted (already acknowledged)

---

**Answer Key & Solution:**

```javascript
const crypto = require('crypto');

// Simulated processed events tracker (in production, use a database)
const processedEvents = new Set();

// Simulated async queue (in production, use RabbitMQ, SQS, etc.)
const eventQueue = [];

async function handleWebhook(req, res) {
  try {
    // Step 1: Extract event ID and signature
    const eventId = req.headers['x-event-id'];
    const signature = req.headers['x-signature'];
    const timestamp = req.headers['x-timestamp'];
    const payload = req.body;

    // Validate required headers
    if (!eventId || !signature || !timestamp) {
      return res.status(400).json({ error: 'Missing required headers' });
    }

    // Step 2: Verify HMAC signature
    const secret = process.env.WEBHOOK_SECRET;
    if (!secret) {
      console.error('WEBHOOK_SECRET not configured');
      return res.status(500).json({ error: 'Server configuration error' });
    }

    // Create the string to sign (timestamp + payload)
    const signString = `${timestamp}.${JSON.stringify(payload)}`;
    const expectedSignature = 'sha256=' + 
      crypto.createHmac('sha256', secret)
        .update(signString)
        .digest('hex');

    // Compare signatures using constant-time comparison
    if (!crypto.timingSafeEqual(
      Buffer.from(signature),
      Buffer.from(expectedSignature)
    )) {
      console.warn(`Invalid signature for event ${eventId}`);
      return res.status(403).json({ error: 'Invalid signature' });
    }

    // Step 3: Validate timestamp (prevent replay attacks)
    const currentTime = Math.floor(Date.now() / 1000);
    const requestTime = parseInt(timestamp);
    const timeDifference = Math.abs(currentTime - requestTime);
    const maxTimeDifference = 300; // 5 minutes

    if (timeDifference > maxTimeDifference) {
      console.warn(`Timestamp too old for event ${eventId}`);
      return res.status(403).json({ error: 'Request timestamp too old' });
    }

    // Step 4: Check for duplicate events (idempotency)
    if (processedEvents.has(eventId)) {
      console.log(`Event ${eventId} already processed, returning success`);
      return res.status(200).json({ 
        message: 'Event already processed',
        eventId: eventId 
      });
    }

    // Step 5: Mark event as processed immediately
    processedEvents.add(eventId);

    // Step 6: Queue for async processing
    eventQueue.push({
      eventId: eventId,
      payload: payload,
      receivedAt: new Date()
    });

    console.log(`Event ${eventId} queued for processing`);

    // Step 7: Return 200 OK immediately
    return res.status(200).json({ 
      message: 'Event received and queued',
      eventId: eventId 
    });

  } catch (error) {
    console.error('Webhook handler error:', error);
    
    // Return 202 if we've already acknowledged the request
    return res.status(500).json({ 
      error: 'Internal server error',
      message: error.message 
    });
  }
}

// Async processor (runs separately)
async function processQueuedEvent(queuedEvent) {
  try {
    const { eventId, payload } = queuedEvent;
    
    // Your business logic here
    console.log(`Processing event ${eventId}:`, payload);
    
    // Example: update database, trigger actions, etc.
    // await updateOrderPaymentStatus(payload.orderId, payload.status);
    
    console.log(`Event ${eventId} processed successfully`);
  } catch (error) {
    console.error(`Error processing event ${eventId}:`, error);
    // Implement retry logic here
  }
}

// Export for use
module.exports = { handleWebhook, processQueuedEvent };
```

**Key Points in Solution:**
1. **Signature Verification:** Uses HMAC-SHA256 with constant-time comparison to prevent timing attacks
2. **Timestamp Validation:** Checks that the request is recent (within 5 minutes) to prevent replay attacks
3. **Idempotency:** Checks if event ID exists before processing; returns success without reprocessing
4. **Quick Acknowledgment:** Returns 200 OK immediately after queuing
5. **Error Handling:** Distinguishes between client errors (400, 403) and server errors (500)
6. **Security:** Uses `crypto.timingSafeEqual()` to prevent timing-based attacks

---

### Question 17 (Subjective - Design & Analysis)

**Scenario:** Your e-commerce platform experiences a sudden traffic spike. Webhook delivery from your payment provider increases from 100 events/second to 5,000 events/second. Your current webhook handler is receiving requests but struggling to keep up, causing timeouts.

**Task:** Design a resilient webhook architecture that handles this spike gracefully. Your design should address:

1. **Ingestion Layer:** How will you receive and acknowledge webhooks quickly?
2. **Buffering & Queuing:** How will you prevent request timeouts?
3. **Processing Layer:** How will you process events at a manageable rate?
4. **Failure Handling:** What happens if the processing layer falls behind?
5. **Monitoring:** What metrics would you track?

**Requirements:**
- Webhooks must be acknowledged within 1 second
- No events should be lost
- The system should auto-scale based on queue depth
- Include a diagram or ASCII representation of your architecture

---

**Answer Key & Solution:**

**Recommended Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│                    Payment Provider                          │
│              (5,000 webhooks/second spike)                   │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────┐
        │   API Gateway / Load Balancer   │
        │  (Fast, stateless ingestion)    │
        └────────────┬───────────────────┘
                     │
        ┌────────────▼──────────────┐
        │  Webhook Receiver Service  │
        │  (Validate signature)      │
        │  (Return 200 OK in <1s)    │
        │  (Queue event)             │
        └────────────┬───────────────┘
                     │
        ┌────────────▼──────────────────┐
        │   Message Queue (RabbitMQ/SQS) │
        │  (Persistent, scalable buffer) │
        │  Queue Depth: Monitored        │
        └────────────┬───────────────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
    ▼                ▼                ▼
┌─────────┐    ┌─────────┐    ┌─────────┐
│ Worker 1│    │ Worker 2│    │ Worker N│
│ Process │    │ Process │    │ Process │
│ Events  │    │ Events  │    │ Events  │
└────┬────┘    └────┬────┘    └────┬────┘
     │              │              │
     └──────────────┼──────────────┘
                    │
        ┌───────────▼──────────┐
        │   Business Database   │
        │  (Order, Payment data)│
        └──────────────────────┘
```

**Detailed Design:**

**1. Ingestion Layer (Fast & Stateless)**
- Use API Gateway or Load Balancer to distribute requests across multiple instances
- Each instance runs a lightweight webhook receiver that:
  - Validates HMAC signature quickly
  - Extracts event ID
  - Pushes to message queue
  - Returns 200 OK within 1 second
- No database queries at this layer—minimal I/O

**2. Buffering & Queuing Strategy**
- Use managed message queue (AWS SQS, RabbitMQ, Apache Kafka)
- Benefits:
  - Decouples ingestion from processing
  - Provides persistent storage (survives failures)
  - Allows processing layer to work at its own pace
  - Handles traffic spikes automatically
- Set appropriate retention (e.g., 24 hours) to replay if needed

**3. Processing Layer (Auto-Scaling)**
- Deploy worker instances that consume from the queue
- Each worker:
  - Processes one event at a time
  - Checks idempotency (processed_webhooks table)
  - Updates database with payment info
  - Acknowledges message after success
- Use auto-scaling rules:
  - Scale up when queue depth > 1,000 events
  - Scale down when queue depth < 100 events
  - Maximum 50 workers (configurable)

**4. Failure Handling**
- **Transient Failures:** Automatic retry with exponential backoff (1s, 2s, 4s, 8s)
- **Persistent Failures:** Move to Dead Letter Queue (DLQ) after 5 retries
- **Circuit Breaker:** If database is down, pause processing and alert ops
- **Idempotency:** Prevents duplicate charges even if events are reprocessed

**5. Monitoring & Alerts**
Key metrics to track:
- **Queue Depth:** Current events waiting to be processed
- **Processing Latency:** Time from queue to completion
- **Error Rate:** Percentage of failed events
- **Worker Utilization:** CPU/Memory per worker
- **DLQ Size:** Number of permanently failed events

Alert thresholds:
- Queue depth > 10,000 → Page on-call engineer
- Error rate > 5% → Alert immediately
- Processing latency > 30 seconds → Investigate bottleneck
- DLQ receiving events → Review and fix root cause

**Advantages of This Design:**
✅ Handles 5,000 events/second spike  
✅ Events acknowledged within 1 second  
✅ No events lost (persistent queue)  
✅ Auto-scales based on demand  
✅ Idempotent (safe to retry)  
✅ Observable (rich metrics)  
✅ Fault-tolerant (failures isolated)

---

### Question 18 (Subjective - Debugging & Problem-Solving)

**Scenario:** Your webhook system is receiving events successfully, but you notice that some payments are being processed twice, creating duplicate charges for customers. The webhook provider confirms they're only sending each event once. Your HMAC verification and timestamps are all correct.

**Task:** Debug this issue. Provide:
1. Three possible root causes for duplicate processing
2. For each cause, explain how you would detect it
3. For each cause, provide a code fix or architectural change

**Constraints:**
- You have logs showing event IDs and timestamps
- You can query your database
- You cannot modify the webhook provider's code

---

**Answer Key & Solution:**

**Possible Root Causes & Debugging:**

**Root Cause #1: Idempotency Check is Not Atomic**

*Explanation:* Your code checks if an event was processed, then processes it, but between the check and the processing, another webhook receiver instance processes the same event.

```javascript
// ❌ WRONG - Race condition possible
if (!processedEvents.has(eventId)) {
  // Between this check and the next line, another instance
  // might also pass the check!
  processedEvents.add(eventId);
  await processPayment(payload); // Duplicate charge happens here
}
```

*Detection:*
- Query your database: `SELECT eventId, COUNT(*) FROM processed_webhooks GROUP BY eventId HAVING COUNT(*) > 1`
- If you see event IDs with count > 1, this is your issue
- Check logs: Do you see the same event ID being processed by different worker instances around the same time?

*Fix:*
```javascript
// ✅ CORRECT - Atomic operation
try {
  // This INSERT will fail if eventId already exists (unique constraint)
  await db.query(
    'INSERT INTO processed_webhooks (eventId, createdAt) VALUES ($1, NOW())',
    [eventId]
  );
  
  // Only if INSERT succeeds do we process
  await processPayment(payload);
} catch (error) {
  if (error.code === '23505') { // Unique constraint violation
    console.log(`Event ${eventId} already processed`);
    return; // Silently succeed
  }
  throw error;
}
```

**Root Cause #2: Webhook Receiver Retries Without Proper Deduplication**

*Explanation:* Your webhook provider is actually sending retries (which is normal behavior), but your receiver isn't deduplicating them. The provider's retry logic might be triggered by your slow responses.

```javascript
// ❌ WRONG - No deduplication of retries
app.post('/webhook', async (req, res) => {
  // Process immediately without checking if already processed
  await processPayment(req.body);
  res.status(200).send('OK');
});
```

*Detection:*
- Check webhook provider's dashboard/logs: Are there multiple delivery attempts for the same event ID?
- Query database: `SELECT eventId, COUNT(*) as count FROM payments WHERE eventId IS NOT NULL GROUP BY eventId HAVING COUNT(*) > 1`
- Look at timestamps: Are duplicates arriving seconds/minutes apart?

*Fix:*
```javascript
// ✅ CORRECT - Deduplication with database
app.post('/webhook', async (req, res) => {
  const eventId = req.headers['x-event-id'];
  
  try {
    // Attempt to record this event as processed
    await db.query(
      'INSERT INTO processed_webhooks (eventId, payload) VALUES ($1, $2)',
      [eventId, JSON.stringify(req.body)]
    );
    
    // Process payment
    await processPayment(req.body);
  } catch (error) {
    if (error.code === '23505') {
      // Already processed - silently return success
      console.log(`Duplicate event ${eventId}, ignoring`);
      return res.status(200).send('OK');
    }
    throw error;
  }
  
  res.status(200).send('OK');
});
```

**Root Cause #3: Multiple Webhook Receiver Instances Processing the Same Event**

*Explanation:* You're running multiple instances of your webhook receiver (for scalability), and the load balancer sometimes routes the same webhook to two instances before either has time to mark it as processed.

```javascript
// ❌ WRONG - In-memory tracking only
const processedInMemory = new Set();

app.post('/webhook', async (req, res) => {
  const eventId = req.headers['x-event-id'];
  
  if (!processedInMemory.has(eventId)) {
    processedInMemory.add(eventId);
    await processPayment(req.body); // Duplicate possible!
  }
  res.status(200).send('OK');
});
// Problem: Each instance has its own Set; they don't share state!
```

*Detection:*
- Check your logs: Do you see the same event ID being processed by different instances (check instance IDs/hostnames)?
- Look at timing: Are duplicates happening within milliseconds of each other?
- Query database: `SELECT eventId, COUNT(*) FROM payments GROUP BY eventId HAVING COUNT(*) > 1` combined with logs showing simultaneous processing

*Fix:*
```javascript
// ✅ CORRECT - Use database as source of truth
app.post('/webhook', async (req, res) => {
  const eventId = req.headers['x-event-id'];
  
  try {
    // This is atomic across all instances
    await db.query(
      'INSERT INTO processed_webhooks (eventId, processedAt) VALUES ($1, NOW())',
      [eventId]
    );
    
    // Safe to process now
    await processPayment(req.body);
  } catch (error) {
    if (error.code === '23505') {
      // Another instance already processed this
      console.log(`Event ${eventId} processed by another instance`);
      return res.status(200).send('OK');
    }
    throw error;
  }
  
  res.status(200).send('OK');
});
```