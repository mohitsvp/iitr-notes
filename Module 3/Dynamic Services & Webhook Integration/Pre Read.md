# Dynamic Services & Webhook Integration: Pre-Class Notes

## What You'll Learn

In this pre-read, you'll discover:

- **What webhooks really are** and how they let systems talk to each other in real-time
- **Why webhooks matter for agent systems** and what happens when they fail
- **Key design patterns** that make webhooks reliable, secure, and fault-tolerant
- **Practical techniques** to handle failures, prevent duplicates, and protect your data

---

## Introduction: What Are Webhooks?

### The Everyday Analogy

Imagine you're ordering a pizza from your favorite restaurant. In the old way, you'd have to keep calling the restaurant every five minutes asking "Is my pizza ready? Is my pizza ready?" That's exhausting and wastes everyone's time.

With webhooks, it's different. You place your order and give the restaurant your phone number. When your pizza is ready, *they* call *you*. You don't ask—they notify you automatically. That's exactly how webhooks work in software.

### The Technical Definition

A **webhook** is an automated HTTP callback that sends real-time data from one application to another whenever a specific event occurs. Think of it as a "push-based" notification system instead of a "pull-based" one. Rather than your code constantly checking "Did anything happen?", the source system sends you a message the moment something important occurs.

In simple terms: webhooks let services communicate asynchronously without constant polling, making systems faster and more efficient.

### How Webhooks Differ from Traditional APIs

When you use a traditional REST API, you have to ask for data: "Give me the user information." With webhooks, the system tells you automatically: "Hey, a user just signed up!" This fundamental difference changes how systems interact, especially in modern event-driven architectures where speed and responsiveness matter.

---

## Why Webhooks Matter for Your Systems

### 1. **Real-Time Communication Without Polling**

Webhooks eliminate the need to constantly ask "Is there new data?" Instead, services push events to you the moment they happen. This saves bandwidth, reduces server load, and delivers information faster. For agent systems that need to react quickly to external events, this is crucial. An agent monitoring payment events, order status changes, or user actions can respond immediately rather than checking repeatedly.

### 2. **Decoupling Services for Scalability**

Webhooks allow different services to communicate without needing to know about each other's internal details. Service A doesn't need to call Service B directly—it just sends an event, and whoever cares about that event can listen. This means you can add new services or change existing ones without breaking the entire system. In agent-driven systems, this flexibility is essential because agents often need to integrate with many different external services.

### 3. **Building Responsive, Event-Driven Agent Systems**

Agents in 2026 operate in event-driven architectures where they react to triggers rather than following fixed schedules. A webhook might trigger an agent to process an order, analyze data, or make a decision. Without reliable webhooks, agents would miss critical events or operate with stale information. Webhooks are the nervous system connecting your agents to the world.

---

## Building Understanding: From Simple to Robust

### The Painful Way: Without Webhooks

Let's say you're building a payment system where an agent needs to know when payments succeed. Without webhooks, here's what you'd do:

```javascript
// The old way: constantly polling
setInterval(async () => {
  const payments = await paymentService.getAllPayments();
  for (const payment of payments) {
    if (payment.status === 'completed' && !payment.processed) {
      await agent.processPayment(payment);
      await paymentService.markAsProcessed(payment.id);
    }
  }
}, 5000); // Check every 5 seconds
```

**Problems with this approach:**
- You're checking constantly, wasting resources
- You might miss events between checks
- If the polling breaks, events are lost
- Your agent can't react immediately
- The database gets hammered with repeated queries

### The Better Way: Using Webhooks

With webhooks, the payment service automatically notifies your agent:

```javascript
// The webhook way: event-driven and efficient
app.post('/webhooks/payment-completed', async (req, res) => {
  const payment = req.body;
  
  // Respond quickly so sender knows we got it
  res.status(202).send({ received: true });
  
  // Process asynchronously
  await agent.processPayment(payment);
});
```

**Advantages:**
- Events arrive immediately, no delay
- Much more efficient resource usage
- Your agent responds in real-time
- Cleaner, simpler code

The webhook approach is how modern systems work. Your agent gets notified the moment something happens, enabling real-time intelligence and decision-making.

---

## Core Components of a Robust Webhook System

### 1. **Event Identification (Unique IDs)**

Every webhook needs a unique identifier so you can track it. Think of it like a tracking number for a package. When the same event is sent multiple times (which happens), you need to recognize it's the same event, not a new one.

```javascript
// Event with unique ID
{
  "id": "evt_1a2b3c4d5e6f7g8h",
  "type": "payment.completed",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": {
    "payment_id": "pay_xyz789",
    "amount": 99.99
  }
}
```

**Why it matters:** If your webhook handler receives the same event twice, the unique ID helps you detect this and avoid processing it twice (called idempotency, which we'll explore next).

### 2. **Idempotency (Handling Duplicates Gracefully)**

**Idempotency** means that processing the same webhook multiple times produces the same result as processing it once. It's like pressing an elevator button—pressing it five times gets you to the same floor, not five floors higher.

Here's a practical example:

```javascript
// Idempotent webhook handler
app.post('/webhooks/order', async (req, res) => {
  const eventId = req.headers['x-webhook-id'];
  const payload = req.body;
  
  // Check if we've already processed this event
  const alreadyProcessed = await db.query(
    'SELECT id FROM processed_events WHERE event_id = ?',
    [eventId]
  );
  
  if (alreadyProcessed.length > 0) {
    // We've seen this before, respond with success
    return res.status(200).json({ status: 'already_processed' });
  }
  
  try {
    // Process the event
    await processOrder(payload);
    
    // Record that we processed it
    await db.query(
      'INSERT INTO processed_events (event_id) VALUES (?)',
      [eventId]
    );
    
    res.status(200).json({ status: 'processed' });
  } catch (error) {
    // Don't record as processed if there's an error
    res.status(500).json({ error: error.message });
  }
});
```

**Why it matters:** Networks are unreliable. A webhook might be sent multiple times due to timeouts, retries, or network glitches. Idempotency ensures your agent doesn't accidentally process the same event twice, which could lead to duplicate orders, double charges, or other serious problems.

### 3. **Request Signing for Security (HMAC)**

How do you know a webhook actually came from the service that claims to send it? Someone could fake a webhook and trick your system. **HMAC (Hash-based Message Authentication Code)** solves this by creating a cryptographic signature that proves the sender is who they claim to be.

Here's how it works:

```javascript
// Sender creates a signature
const crypto = require('crypto');

function createSignature(payload, secret) {
  return crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');
}

// Example: sender sends this header
const signature = createSignature(payload, 'webhook-secret-key');
// Result: "abcd1234ef5678gh9012ijkl3456mnop"

// Receiver verifies the signature
function verifySignature(payload, receivedSignature, secret) {
  const expectedSignature = createSignature(payload, secret);
  // Use timing-safe comparison to prevent attacks
  return crypto.timingSafeEqual(
    Buffer.from(receivedSignature),
    Buffer.from(expectedSignature)
  );
}

// In your webhook handler
app.post('/webhooks/payment', (req, res) => {
  const signature = req.headers['x-webhook-signature'];
  
  if (!verifySignature(req.body, signature, process.env.WEBHOOK_SECRET)) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  // Safe to process—we know it's legitimate
  processPayment(req.body);
  res.status(200).send('OK');
});
```

**Why it matters:** Without signature verification, an attacker could send fake webhooks to your system, potentially triggering unauthorized actions. HMAC signatures prove authenticity and integrity—the message came from the right source and hasn't been tampered with.

### 4. **Retry Logic with Exponential Backoff**

Webhooks sometimes fail. The receiving server might be temporarily down, the network might have issues, or there could be a timeout. **Exponential backoff** means waiting longer between each retry attempt, which gives failing systems time to recover without overwhelming them.

Here's the concept:

```
Attempt 1: Send immediately
Attempt 2: Wait 2 seconds, then send
Attempt 3: Wait 4 seconds, then send
Attempt 4: Wait 8 seconds, then send
Attempt 5: Wait 16 seconds, then send
...and so on, doubling each time
```

A real implementation:

```javascript
async function sendWebhookWithRetry(url, payload, options = {}) {
  const {
    maxRetries = 5,
    baseDelay = 2000, // 2 seconds
    maxDelay = 300000, // 5 minutes
    jitter = 0.1 // 10% randomness
  } = options;
  
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload),
        timeout: 10000 // 10 second timeout
      });
      
      if (response.ok) {
        return { success: true, attempt };
      }
      
      // Don't retry on 4xx errors (client's fault)
      if (response.status >= 400 && response.status < 500) {
        return { success: false, error: 'Client error', attempt };
      }
      
      // Will retry on 5xx errors (server's fault)
    } catch (error) {
      // Network error, will retry
    }
    
    // Calculate delay with exponential backoff and jitter
    const exponentialDelay = Math.min(
      baseDelay * Math.pow(2, attempt),
      maxDelay
    );
    
    // Add jitter: randomness from 0 to 10% of delay
    const jitterAmount = exponentialDelay * jitter * Math.random();
    const delayMs = exponentialDelay + jitterAmount;
    
    console.log(`Retry attempt ${attempt + 1}, waiting ${delayMs}ms`);
    await new Promise(resolve => setTimeout(resolve, delayMs));
  }
  
  return { success: false, error: 'Max retries exceeded' };
}
```

**Why it matters:** Without smart retries, transient failures (temporary network issues, brief server downtime) would cause you to lose important events. Exponential backoff ensures you keep trying without hammering the endpoint when it's struggling.

### 5. **Circuit Breaker Pattern for Failing Endpoints**

Imagine an endpoint keeps failing. If you keep retrying it with exponential backoff, you'll eventually send a massive wave of traffic when it recovers, potentially crashing it again. The **circuit breaker pattern** prevents this by stopping attempts after repeated failures.

Think of it like an electrical circuit breaker in your home: when there's a problem, it cuts power to prevent damage, then tries to restore it later.

```javascript
class CircuitBreaker {
  constructor(options = {}) {
    this.failureThreshold = options.failureThreshold || 5; // failures before trip
    this.resetTimeout = options.resetTimeout || 60000; // 1 minute
    this.state = 'CLOSED'; // CLOSED, OPEN, or HALF_OPEN
    this.failureCount = 0;
    this.lastFailureTime = null;
  }
  
  async execute(fn) {
    if (this.state === 'OPEN') {
      // Check if timeout has passed
      if (Date.now() - this.lastFailureTime > this.resetTimeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }
    
    try {
      const result = await fn();
      
      // Success! Reset the breaker
      if (this.state === 'HALF_OPEN') {
        this.state = 'CLOSED';
        this.failureCount = 0;
      }
      
      return result;
    } catch (error) {
      this.failureCount++;
      this.lastFailureTime = Date.now();
      
      if (this.failureCount >= this.failureThreshold) {
        this.state = 'OPEN';
        console.log('Circuit breaker OPENED due to repeated failures');
      }
      
      throw error;
    }
  }
}

// Usage
const breaker = new CircuitBreaker({
  failureThreshold: 5,
  resetTimeout: 60000
});

app.post('/webhooks/external-service', async (req, res) => {
  try {
    await breaker.execute(async () => {
      await sendToExternalService(req.body);
    });
    res.status(200).send('OK');
  } catch (error) {
    if (error.message.includes('Circuit breaker')) {
      // Queue for later retry
      await queueWebhook(req.body);
      res.status(503).send('Service temporarily unavailable');
    } else {
      res.status(500).send('Error');
    }
  }
});
```

**Why it matters:** Circuit breakers prevent your system from hammering a failing endpoint, which could make things worse. They also give you time to queue up events for later processing, so nothing is lost.

### 6. **Dead Letter Queues (DLQ) for Permanent Failures**

After retrying with exponential backoff and circuit breakers, some webhooks will still fail. These events go to a **Dead Letter Queue (DLQ)** for manual inspection and recovery.

```javascript
// When webhook exhausts all retries
async function handlePermanentFailure(webhook, error) {
  // Store in DLQ for later analysis
  await dlq.store({
    webhook_id: webhook.id,
    event_type: webhook.type,
    payload: webhook.payload,
    error: error.message,
    failed_at: new Date(),
    retry_count: webhook.retries,
    last_attempt: webhook.lastAttempt
  });
  
  // Alert the team
  await alerting.send({
    severity: 'high',
    message: `Webhook ${webhook.id} failed after ${webhook.retries} retries`,
    details: error
  });
}
```

**Why it matters:** DLQs ensure no data is lost. Even if a webhook fails permanently, you have a record of it in the DLQ where engineers can investigate what went wrong and potentially retry it manually.

---

## Step-by-Step: Building a Resilient Webhook Handler

### Step 1: Receive and Validate Quickly

Your webhook endpoint should respond quickly (within 2-3 seconds) so the sender knows you got the message. This doesn't mean you've processed it—just that you received it.

```javascript
app.post('/webhooks/event', async (req, res) => {
  // Step 1: Validate signature immediately
  const signature = req.headers['x-signature'];
  if (!verifySignature(req.body, signature)) {
    return res.status(401).send('Unauthorized');
  }
  
  // Step 2: Respond immediately with 202 (Accepted)
  res.status(202).send({ received: true });
  
  // Step 3: Queue for async processing
  await eventQueue.push(req.body);
});
```

**Key point:** Respond quickly, then process asynchronously. This prevents timeouts and makes your system responsive.

### Step 2: Check for Duplicates

Before processing, check if you've already handled this event.

```javascript
async function processWebhookEvent(event) {
  const eventId = event.id;
  
  // Check if already processed
  const processed = await db.query(
    'SELECT id FROM processed_webhooks WHERE event_id = ?',
    [eventId]
  );
  
  if (processed.length > 0) {
    console.log(`Event ${eventId} already processed, skipping`);
    return;
  }
  
  // Mark as processing to prevent race conditions
  try {
    await db.query(
      'INSERT INTO processed_webhooks (event_id, status) VALUES (?, ?)',
      [eventId, 'processing']
    );
  } catch (error) {
    // Unique constraint violation means another process is handling it
    console.log(`Event ${eventId} is being processed elsewhere`);
    return;
  }
  
  // Continue to next step...
}
```

### Step 3: Verify and Enrich Data

Ensure the event data is valid and get any additional context you need.

```javascript
async function verifyAndEnrich(event) {
  // Validate required fields
  if (!event.id || !event.type || !event.data) {
    throw new Error('Invalid event structure');
  }
  
  // Enrich with additional data if needed
  if (event.type === 'order.completed') {
    const order = await orderService.getOrder(event.data.order_id);
    event.enriched_data = {
      customer_name: order.customer.name,
      total_amount: order.total,
      items_count: order.items.length
    };
  }
  
  return event;
}
```

### Step 4: Process with Circuit Breaker Protection

Execute your business logic with circuit breaker protection.

```javascript
async function processEvent(event, circuitBreaker) {
  try {
    await circuitBreaker.execute(async () => {
      if (event.type === 'order.completed') {
        await agent.handleOrderCompletion(event);
      } else if (event.type === 'payment.failed') {
        await agent.handlePaymentFailure(event);
      }
      // ... other event types
    });
  } catch (error) {
    if (error.message.includes('Circuit breaker')) {
      // Queue for later retry
      await retryQueue.push(event);
    } else {
      throw error;
    }
  }
}
```

### Step 5: Handle Failures Gracefully

If processing fails, decide whether to retry or send to DLQ.

```javascript
async function handleFailure(event, error, retryCount) {
  if (retryCount < 5) {
    // Calculate exponential backoff
    const delay = Math.min(1000 * Math.pow(2, retryCount), 300000);
    
    // Re-queue with updated retry count
    await retryQueue.push({
      ...event,
      retry_count: retryCount + 1,
      scheduled_for: Date.now() + delay
    });
    
    console.log(`Event ${event.id} queued for retry in ${delay}ms`);
  } else {
    // Max retries exceeded, send to DLQ
    await deadLetterQueue.store({
      event,
      error: error.message,
      retry_count: retryCount,
      failed_at: new Date()
    });
    
    console.log(`Event ${event.id} sent to DLQ after ${retryCount} retries`);
  }
}
```

### Step 6: Mark as Complete

Once successfully processed, mark the event as done.

```javascript
async function markComplete(eventId) {
  await db.query(
    'UPDATE processed_webhooks SET status = ?, completed_at = ? WHERE event_id = ?',
    ['completed', new Date(), eventId]
  );
}
```

---

## Key Features for Production Webhooks

### 1. **Observability and Monitoring**

Track every webhook's journey through your system:

```javascript
// Log webhook lifecycle
const webhookLogger = {
  received: (eventId) => console.log(`Received: ${eventId}`),
  validated: (eventId) => console.log(`Validated: ${eventId}`),
  processing: (eventId) => console.log(`Processing: ${eventId}`),
  succeeded: (eventId) => console.log(`Succeeded: ${eventId}`),
  failed: (eventId, error) => console.log(`Failed: ${eventId}, Error: ${error}`),
  retrying: (eventId, attempt) => console.log(`Retrying: ${eventId}, Attempt: ${attempt}`),
  dlq: (eventId, reason) => console.log(`DLQ: ${eventId}, Reason: ${reason}`)
};

// Metrics to track
const metrics = {
  total_received: 0,
  total_processed: 0,
  total_failed: 0,
  total_dlq: 0,
  avg_processing_time: 0,
  retry_count_distribution: {}
};
```

**Why it matters:** When something goes wrong, detailed logs help you understand what happened and fix it quickly. Metrics help you spot patterns and optimize your system.

### 2. **Replay Capability**

Sometimes you need to reprocess old events (e.g., after fixing a bug). Store complete webhook history:

```javascript
// Store complete webhook history
async function storeWebhookHistory(event) {
  await db.query(
    `INSERT INTO webhook_history 
     (event_id, event_type, payload, received_at, processed_at, status)
     VALUES (?, ?, ?, ?, ?, ?)`,
    [
      event.id,
      event.type,
      JSON.stringify(event),
      new Date(),
      null,
      'received'
    ]
  );
}

// Replay from history
async function replayEvent(eventId) {
  const event = await db.query(
    'SELECT payload FROM webhook_history WHERE event_id = ?',
    [eventId]
  );
  
  if (event) {
    await eventQueue.push(JSON.parse(event[0].payload));
  }
}
```

**Why it matters:** Replay capability lets you recover from bugs or fix processing errors without losing data.

---

## Putting It All Together: A Complete Example

Here's a complete, production-ready webhook handler that incorporates all the concepts:

```javascript
const express = require('express');
const crypto = require('crypto');
const app = express();

// Configuration
const WEBHOOK_SECRET = process.env.WEBHOOK_SECRET;
const MAX_RETRIES = 5;
const BASE_DELAY = 2000;

// Utility: Verify HMAC signature
function verifySignature(payload, signature) {
  const expectedSignature = crypto
    .createHmac('sha256', WEBHOOK_SECRET)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

// Utility: Calculate exponential backoff
function calculateBackoff(retryCount) {
  const exponential = Math.min(
    BASE_DELAY * Math.pow(2, retryCount),
    300000 // 5 minute max
  );
  const jitter = exponential * 0.1 * Math.random();
  return exponential + jitter;
}

// Main webhook handler
app.post('/webhooks/events', async (req, res) => {
  const event = req.body;
  const signature = req.headers['x-webhook-signature'];
  
  // Step 1: Verify signature
  try {
    if (!verifySignature(event, signature)) {
      return res.status(401).json({ error: 'Invalid signature' });
    }
  } catch (error) {
    return res.status(401).json({ error: 'Signature verification failed' });
  }
  
  // Step 2: Respond immediately
  res.status(202).json({ received: true, event_id: event.id });
  
  // Step 3: Process asynchronously
  processWebhookAsync(event).catch(error => {
    console.error(`Failed to queue webhook ${event.id}:`, error);
  });
});

// Async processor
async function processWebhookAsync(event) {
  const eventId = event.id;
  
  // Check for duplicates
  const alreadyProcessed = await checkIfProcessed(eventId);
  if (alreadyProcessed) {
    console.log(`Event ${eventId} already processed`);
    return;
  }
  
  // Mark as processing
  await markAsProcessing(eventId);
  
  try {
    // Verify data integrity
    if (!event.id || !event.type || !event.data) {
      throw new Error('Invalid event structure');
    }
    
    // Process based on event type
    switch (event.type) {
      case 'order.completed':
        await handleOrderCompleted(event);
        break;
      case 'payment.processed':
        await handlePaymentProcessed(event);
        break;
      default:
        console.warn(`Unknown event type: ${event.type}`);
    }
    
    // Mark as completed
    await markAsCompleted(eventId);
    console.log(`Event ${eventId} processed successfully`);
    
  } catch (error) {
    console.error(`Error processing event ${eventId}:`, error);
    await handleProcessingError(event, error, 0);
  }
}

// Retry handler with exponential backoff
async function retryWithBackoff(event, retryCount) {
  if (retryCount >= MAX_RETRIES) {
    // Send to DLQ
    await sendToDLQ(event, `Max retries (${MAX_RETRIES}) exceeded`);
    return;
  }
  
  const delay = calculateBackoff(retryCount);
  console.log(`Scheduling retry for event ${event.id} in ${delay}ms`);
  
  setTimeout(async () => {
    try {
      await processWebhookAsync({ ...event, retry_count: retryCount + 1 });
    } catch (error) {
      await handleProcessingError(event, error, retryCount + 1);
    }
  }, delay);
}

// Error handler
async function handleProcessingError(event, error, retryCount) {
  if (retryCount < MAX_RETRIES) {
    await retryWithBackoff(event, retryCount);
  } else {
    await sendToDLQ(event, error.message);
  }
}

// Business logic handlers
async function handleOrderCompleted(event) {
  console.log(`Processing order completion: ${event.data.order_id}`);
  // Your agent logic here
  // await agent.processOrder(event.data);
}

async function handlePaymentProcessed(event) {
  console.log(`Processing payment: ${event.data.payment_id}`);
  // Your agent logic here
  // await agent.processPayment(event.data);
}

// Database helpers (pseudo-code)
async function checkIfProcessed(eventId) {
  // Query database
  return false;
}

async function markAsProcessing(eventId) {
  // Update database
}

async function markAsCompleted(eventId) {
  // Update database
}

async function sendToDLQ(event, reason) {
  console.log(`Sending event ${event.id} to DLQ: ${reason}`);
  // Store in DLQ for manual review
}

// Start server
app.listen(3000, () => {
  console.log('Webhook server listening on port 3000');
});
```

This example demonstrates all key concepts: signature verification, quick response, async processing, duplicate detection, error handling, and retry logic.

---

## Practice Exercises

### Exercise 1: Pattern Recognition - Spot the Problem

Read this webhook handler and identify what's missing or problematic:

```javascript
app.post('/webhooks/payment', (req, res) => {
  const payment = req.body;
  
  // Process the payment
  await chargeAccount(payment.account_id, payment.amount);
  
  // Send email
  await sendConfirmationEmail(payment.email);
  
  res.status(200).send('OK');
});
```

**What could go wrong?** Think about:
- What if the same webhook is sent twice?
- What if the process crashes halfway through?
- How does the sender know if it was processed?
- Is there any security verification?

### Exercise 2: Concept Detective - What's This Pattern?

You see this code in a production system:

```javascript
if (failureCount > 5) {
  circuitBreaker.state = 'OPEN';
  await queue.push(event); // Queue for later
}
```

**Question:** What problem is this pattern trying to solve? Why queue the event instead of just giving up?

### Exercise 3: Real-Life Application

Think about your own projects or systems you use. List three situations where webhooks would be valuable:

1. **Example:** An e-commerce system that needs to update inventory when an order is placed
2. **Your example #1:** ...
3. **Your example #2:** ...
4. **Your example #3:** ...

For each, describe what event would trigger the webhook and what system would need to receive it.

### Exercise 4: Spot the Error - Security Issue

You're reviewing this webhook handler:

```javascript
app.post('/webhooks/user-signup', (req, res) => {
  const signature = req.headers['x-signature'];
  const payload = req.body;
  
  // Verify signature
  if (signature === createSignature(payload)) {
    await createUserAccount(payload);
    res.status(200).send('OK');
  } else {
    res.status(401).send('Unauthorized');
  }
});

function createSignature(payload) {
  return crypto
    .createHmac('sha256', WEBHOOK_SECRET)
    .update(JSON.stringify(payload))
    .digest('hex');
}
```

**What's the security issue?** (Hint: Think about timing attacks and string comparison)

### Exercise 5: Planning Ahead - Design Your Webhook

You're building an agent system that monitors GitHub events. Design a webhook handler that should:
- Receive push events from GitHub
- Verify they're legitimate (GitHub sends HMAC signatures)
- Trigger your agent to analyze code changes
- Handle failures gracefully

Sketch out the main steps your handler would follow. What happens if GitHub sends the same event twice? What happens if your agent crashes?

---

## Key Takeaways

- **Webhooks enable real-time communication** between services without constant polling, making systems faster and more efficient
- **Idempotency is essential** because webhooks can be delivered multiple times, and you need to handle duplicates gracefully
- **HMAC signatures prove authenticity** and ensure you only process webhooks from trusted sources
- **Exponential backoff with jitter** prevents overwhelming failing endpoints while still retrying aggressively enough to recover from transient failures
- **Circuit breakers protect your infrastructure** by stopping attempts to failing endpoints after repeated failures
- **Dead Letter Queues ensure no data is lost**, even when webhooks fail permanently after all retries
- **Quick responses (202 Accepted) combined with async processing** make your webhook handlers responsive and reliable
- **Observability and monitoring** are crucial for understanding webhook flow and debugging issues

---

## What's Next?

In the upcoming class, we'll dive deeper into:
- Building production webhook systems with actual code
- Implementing these patterns in agent frameworks
- Handling edge cases and failure scenarios
- Testing webhook handlers thoroughly
- Monitoring and alerting for webhook systems
- Scaling webhooks to handle millions of events

Come prepared to code and think through failure scenarios. Bring questions about your own webhook challenges!