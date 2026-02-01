# Dynamic Services & Webhook Integration

## What You'll Learn

- **Design robust webhooks** that reliably deliver events across systems without losing data or causing duplicate processing
- **Implement idempotency** to ensure your system processes the same event multiple times safely without side effects
- **Secure webhook communications** using HMAC-SHA256 signing to verify that messages truly come from trusted sources
- **Handle failures gracefully** with retry strategies, exponential backoff, circuit breakers, and dead letter queues for maximum reliability

---

## Introduction: Why Webhooks Matter in Modern Systems

Webhooks have become the nervous system of modern software architecture. They enable real-time communication between services without constant polling. Instead of repeatedly asking "Is there anything new?", webhooks let services push notifications when something important happens.

Think of webhooks like a doorbell system for your applications. Rather than having a security guard constantly check if someone is at your door, the doorbell rings automatically when someone arrives. This saves energy, reduces latency, and creates a more responsive system.

In today's interconnected world, your application rarely works in isolation. You might integrate with payment processors, send notifications through third-party services, update databases across multiple systems, or trigger workflows in other applications. Webhooks make these integrations possible and practical.

However, webhooks introduce complexity. Networks are unreliable. Services crash. Messages get lost. Duplicate deliveries happen. Without proper design, you might charge customers twice, send duplicate notifications, or lose critical events entirely. This lecture teaches you how to build webhook systems that handle real-world chaos gracefully.

---

## Webhook Design Best Practices

### What Is a Webhook? A Relatable Analogy

Imagine you run a pizza restaurant. In the old system, customers constantly call asking "Is my pizza ready?" This wastes everyone's time. With webhooks, you implement a callback system. When a pizza finishes cooking, the kitchen automatically calls the customer's number. The customer doesn't need to check repeatedly. The information comes to them automatically.

A **webhook** is exactly this pattern for software. Instead of your application constantly polling another service asking "Did something happen?", that service pushes a notification to your application whenever something important occurs. The external service makes an HTTP POST request to a URL you provide. Your application receives the notification and processes it.

### Why Webhook Design Matters

Poor webhook design creates cascading failures. If your webhook handler crashes, events disappear. If you don't acknowledge receipt, the sender might keep retrying. If you process events out of order, your data becomes inconsistent. If you don't validate incoming data, malicious actors can inject false events. Proper design prevents all these problems.

Well-designed webhooks enable real-time integrations that scale beautifully. They decouple services, allowing them to evolve independently. They reduce database load by eliminating constant polling. They provide immediate feedback to users. They enable complex workflows across multiple systems.

### Core Webhook Architecture

A webhook system has several essential components. First, the **event source** detects something important and needs to notify other systems. Second, the **webhook dispatcher** sends HTTP POST requests to registered endpoints. Third, the **webhook receiver** (your application) accepts the request and processes it. Fourth, the **acknowledgment mechanism** confirms successful processing.

Here's a basic webhook receiver structure:

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/webhooks/payment', methods=['POST'])
def handle_payment_webhook():
    payload = request.get_json()
    event_id = payload.get('id')
    
    # Queue for async processing
    queue_event(event_id, payload)
    
    # Return success immediately
    return jsonify({'status': 'received'}), 202
```

This simple structure demonstrates the key principle: **receive quickly, process asynchronously**. Acknowledge receipt immediately, then handle the actual work in the background. This prevents timeouts and allows the sender to mark delivery as successful.

### Payload Structure and Versioning

Your webhook payloads must contain all necessary information for the receiver to process events independently. However, as your system evolves, you'll need to change payload structures. Without versioning, this breaks existing integrations.

A well-designed webhook payload includes:

```json
{
  "id": "evt_1a2b3c4d5e6f7g8h",
  "type": "payment.completed",
  "version": "2024-01",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "payment_id": "pay_xyz789",
    "amount": 9999,
    "currency": "USD",
    "customer_id": "cust_abc123"
  }
}
```

The **version** field is crucial. It tells receivers which data structure to expect. When you need to add fields, increment the version. Receivers can handle multiple versions, supporting old integrations while enabling new features.

### Designing for Resilience

Resilient webhook design assumes failure as normal. Network timeouts happen. Servers crash. Messages get lost. Your design must survive these conditions.

Key resilience principles include designing for async processing, making payloads self-contained, using meaningful event types, and including correlation IDs. Never do heavy work in your webhook handler. Queue the event and return immediately. This prevents timeouts and allows retries without blocking.

Don't require the receiver to fetch additional data. Include everything needed in the webhook payload. If the receiver's database is temporarily down, it can still process the event later. Use clear event type names like "payment.completed" and "payment.failed" to help receivers route events appropriately.

Add a correlation ID to every webhook. This helps trace events across multiple systems and debug issues. When a receiver processes an event and triggers downstream webhooks, it should include the same correlation ID:

```json
{
  "id": "evt_1a2b3c4d5e6f7g8h",
  "type": "order.created",
  "correlation_id": "corr_abc123xyz",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "order_id": "ord_12345",
    "items": [{"sku": "WIDGET-001", "qty": 2}]
  }
}
```

### Webhook Endpoint Requirements

When asking developers to implement webhook endpoints, be clear about your requirements. Always require HTTPS. Never allow HTTP webhooks. HTTP webhooks are vulnerable to man-in-the-middle attacks where attackers intercept and modify webhook data. HTTPS with proper certificate validation prevents this.

Specify what response codes you consider successful. Typically, any 2xx status code (200-299) indicates successful delivery. Responses with 4xx status codes (client errors) usually indicate the webhook URL is invalid or the endpoint rejected the data, so you shouldn't retry. Responses with 5xx status codes (server errors) indicate temporary failures, so you should retry.

Specify how long you'll wait for a response. Typically 10-30 seconds is reasonable. If the endpoint doesn't respond within this time, treat it as a failure and retry later. This prevents your webhook system from hanging on slow endpoints.

### Common Confusions About Webhook Design

**Confusion 1: Webhooks guarantee delivery order**

Webhooks do not guarantee events arrive in order. Event A might be sent before Event B, but Event B might arrive first due to network conditions. Your system must handle events arriving out of order. Use timestamps and event ordering information to reconstruct the correct sequence.

**Confusion 2: Webhook handlers should do all the work**

Many developers make webhook handlers do heavy lifting: database queries, external API calls, complex calculations. This causes timeouts and makes retries painful. Instead, queue the event immediately and process asynchronously. Your webhook handler should take milliseconds, not seconds.

**Confusion 3: One webhook endpoint handles all events**

While possible, this creates brittle systems. If one event type handler crashes, it affects all event types. Instead, create separate endpoints for different event categories. This isolates failures and makes the system more maintainable.

### Tips and Cautions

**Tip: Implement health checks.** Create an endpoint that receives test webhooks. Senders can use this to verify your receiver is healthy. This helps detect problems early.

**Tip: Log everything.** Every webhook received should be logged with full payload, timestamp, and processing result. These logs are invaluable for debugging integration issues.

**Caution: Don't trust webhook data blindly.** Always validate webhook payloads. Check that required fields exist. Verify data types. Validate email addresses and URLs. Malicious actors might send crafted payloads to exploit your system.

**Caution: Set appropriate timeouts.** If your webhook handler takes too long, the sender might timeout and retry. Set realistic timeout expectations and communicate them clearly. Typically, 30 seconds is a reasonable timeout.

---

## Idempotency in Webhook Systems

### What Is Idempotency? A Relatable Analogy

Imagine you're listening to a voicemail system. You press "1" to confirm receipt. Sometimes the system doesn't register your input, so you press "1" again. The system should be smart enough to recognize you already confirmed and not process the confirmation twice. This is idempotency.

In software, **idempotency** means performing the same operation multiple times produces the same result as performing it once. Idempotent systems are safe to retry. If a webhook delivery succeeds but the acknowledgment gets lost, the sender might retry. An idempotent system handles this gracefully without side effects.

### Why Idempotency Matters

Without idempotency, network failures cause catastrophic problems. Imagine a payment webhook. The receiver processes the payment successfully and returns a success response. But the network connection drops before the response reaches the sender. The sender doesn't know if the payment was processed, so it retries. The receiver processes the same payment again. The customer gets charged twice.

Idempotency prevents this scenario. The receiver uses an **idempotency key** to detect duplicate requests. It processes the payment once, stores the result associated with the key, and returns the same result for all future requests with that key. The customer gets charged exactly once, regardless of retries.

Idempotency also simplifies your system architecture. You don't need complex logic to prevent duplicates. You don't need to carefully orchestrate retries. You simply implement idempotency and let the sender retry freely. This makes your system more robust and easier to reason about.

### Implementing Idempotency with Event IDs

The foundation of idempotent webhook handling is the **event ID**. Every webhook must include a unique event ID that remains constant across all delivery attempts of the same event. Store a record of every event ID you've processed:

```python
@app.route('/webhooks/payment', methods=['POST'])
def handle_payment_webhook():
    payload = request.get_json()
    event_id = payload.get('id')
    
    # Check if we've already processed this
    if event_already_processed(event_id):
        return {'status': 'already_processed'}, 200
    
    try:
        # Process the event
        process_payment_event(payload)
        
        # Record that we processed this event
        record_processed_event(event_id, payload)
        
        return {'status': 'processed'}, 200
    except Exception as e:
        return {'error': str(e)}, 500
```

This implementation checks your deduplication store before processing. If you've already seen this event ID, return success without reprocessing. If not, process the event and record that you've seen this ID.

### Idempotency Key Pattern

Some systems use an **idempotency key** provided by the client. This is particularly useful for synchronous operations where the client wants to ensure their request is only processed once. The server stores the idempotency key and response together:

```python
@app.route('/api/payments', methods=['POST'])
def create_payment():
    idempotency_key = request.headers.get('Idempotency-Key')
    
    # Check if we've already processed this key
    existing_response = get_cached_response(idempotency_key)
    if existing_response:
        return existing_response
    
    # Process the request
    payment = process_payment(request.json)
    response = {'payment_id': payment.id, 'status': 'created'}
    
    # Cache the response for future identical requests
    cache_response(idempotency_key, response)
    
    return response, 201
```

### Time-Based Deduplication Windows

You don't need to store every event forever. Most systems use a **deduplication window** of 24-72 hours. Events older than this window are assumed to be legitimate duplicates that can be reprocessed (the original must have been lost).

This approach balances reliability (catching duplicates within a reasonable window) with storage efficiency (not storing ancient history). After the window expires, you can safely delete old event records from your deduplication store.

### Designing Idempotent Operations

Making your webhook handlers idempotent requires careful design of the operations they perform. Non-idempotent operations produce different results when repeated. For example, incrementing a counter is non-idempotent. If this runs twice, the counter increments twice.

Idempotent operations produce the same result when repeated. Setting a value to a specific state is idempotent. If this runs twice, the database still shows the same value.

Database **upsert** (update or insert) operations are naturally idempotent. If the record exists, update it. If not, create it. Running an upsert multiple times with the same data produces the same result as running it once:

```python
# SQL upsert - idempotent
INSERT INTO sales (sale_id, amount, status)
VALUES (?, ?, ?)
ON CONFLICT (sale_id) 
DO UPDATE SET 
    amount = EXCLUDED.amount,
    status = EXCLUDED.status;
```

When handling state changes, ensure transitions are idempotent. Only allow valid transitions and check if you're already in the target state before transitioning.

### Idempotency and External Services

When your webhook handler calls external services, maintaining idempotency becomes more complex. Store the result of that call and reuse it on retries:

```python
@app.route('/webhook/charge', methods=['POST'])
def handle_charge_webhook():
    event = request.json
    event_id = event['id']
    
    # Check if we've already processed this
    existing = get_processed_event(event_id)
    if existing:
        return {'status': 'already_processed'}, 200
    
    try:
        # Call external service
        charge_result = call_payment_processor(event['data'])
        
        # Store both the event and the result
        record_processed_event(event_id, event, charge_result)
        
        return {'status': 'processed'}, 200
    except Exception as e:
        return {'error': str(e)}, 500
```

When calling external services from your webhook handler, provide an **idempotency key** to the external service. Use the webhook event ID as the idempotency key. The payment processor will deduplicate based on this, ensuring that even if your webhook handler retries, the external service won't process the operation twice.

### Common Idempotency Confusions

**Confusion 1: Do I need to store every event forever?**

No. Use a deduplication window (24-72 hours). Store events longer than this and assume they're new events, not duplicates.

**Confusion 2: What if I accidentally process a duplicate before checking?**

This is why you check for duplicates **before** processing, not after. Always check your deduplication store first.

**Confusion 3: Can I make non-idempotent operations idempotent?**

Sometimes. Redesign the operation to be state-based rather than action-based. Instead of "increment counter," do "set counter to value." Instead of "add to list," do "ensure item exists in list."

### Tips and Cautions

**Tip: Use database unique constraints.** Create a unique constraint on your event ID column. If you try to insert the same ID twice, the database rejects it. This provides a safety net.

**Tip: Store both success and failure results.** If processing fails, cache the error response. Retries should receive the same error, not retry the failed operation.

**Caution: Don't use timestamps as event IDs.** Timestamps aren't unique. Two events might have the same timestamp. Use proper unique identifiers.

**Caution: Idempotency is not free.** Storing results requires storage space and lookup time. Use efficient data structures. In high-throughput systems, consider using a dedicated cache like Redis.

---

## Request Signing for Security

### What Is Request Signing? A Relatable Analogy

Imagine you receive a letter claiming to be from your bank. How do you know it's really from your bank and not a forged letter from a scammer? Banks sign letters with a special seal. You can verify the seal using the bank's public key. If the seal is authentic, you trust the letter.

**Request signing** works the same way. The webhook sender signs the request using a secret key. Your application verifies the signature using the sender's public key. If the signature is valid, you know the request truly came from the sender and hasn't been tampered with. If the signature is invalid, someone forged the request.

### Why Request Signing Matters

Without request signing, anyone can send webhook requests to your endpoint. A malicious actor could send fake payment notifications, fake order confirmations, or fake user data. Your application would process these fraudulent events as if they came from the legitimate sender.

Request signing prevents this. Only someone with the secret key can create valid signatures. Since you trust the sender, you trust that they keep their secret key secure. If you receive a request with a valid signature, you know it came from the legitimate sender.

Signing also detects tampering. If someone modifies the webhook payload in transit, the signature becomes invalid. This protects against man-in-the-middle attacks where an attacker intercepts and modifies webhook data.

### HMAC-SHA256: The Standard Approach

The industry standard for webhook signing is **HMAC-SHA256**. HMAC stands for Hash-Based Message Authentication Code. SHA256 is a cryptographic hash function. Together, they create a signature that proves both authenticity and integrity.

Here's how HMAC-SHA256 signing works:

1. The sender creates a message (the webhook payload)
2. The sender computes HMAC-SHA256 of the message using a secret key
3. The sender includes this signature in the webhook request (usually in a header)
4. Your application receives the request
5. Your application computes HMAC-SHA256 of the payload using the same secret key
6. Your application compares the computed signature with the received signature
7. If they match, the request is authentic and unmodified

Here's the receiver-side implementation:

```python
import hmac
import hashlib
from flask import Flask, request, jsonify

app = Flask(__name__)
WEBHOOK_SECRET = "your_secret_key_here"

@app.route('/webhooks/payment', methods=['POST'])
def handle_payment_webhook():
    # Get the signature from the header
    signature = request.headers.get('X-Webhook-Signature')
    
    # Get the raw request body
    body = request.get_data()
    
    # Compute the expected signature
    expected_signature = hmac.new(
        WEBHOOK_SECRET.encode(),
        body,
        hashlib.sha256
    ).hexdigest()
    
    # Compare using constant-time comparison
    if not hmac.compare_digest(signature, expected_signature):
        return jsonify({'error': 'Invalid signature'}), 401
    
    # Signature is valid, process the webhook
    payload = request.json
    process_event(payload)
    
    return jsonify({'status': 'success'}), 200
```

### Sender-Side Signing

If you're building the webhook sender (the service sending webhooks), here's how to sign requests:

```python
import hmac
import hashlib
import json
import requests

def send_webhook(url, payload, secret_key):
    # Convert payload to JSON string
    body = json.dumps(payload, separators=(',', ':'), sort_keys=True)
    
    # Compute signature
    signature = hmac.new(
        secret_key.encode(),
        body.encode(),
        hashlib.sha256
    ).hexdigest()
    
    # Send request with signature header
    headers = {
        'Content-Type': 'application/json',
        'X-Webhook-Signature': signature
    }
    
    response = requests.post(url, data=body, headers=headers)
    return response
```

### Preventing Replay Attacks

A **replay attack** occurs when an attacker captures a valid webhook request and resends it multiple times. The signature is still valid, so your application processes the same event repeatedly. This is especially dangerous for payment webhooks.

Prevent replay attacks by including a timestamp in the signature:

```python
import time

def send_webhook_with_timestamp(url, payload, secret_key):
    timestamp = int(time.time())
    body = json.dumps(payload, separators=(',', ':'), sort_keys=True)
    
    # Create signed content: timestamp.body
    signed_content = f"{timestamp}.{body}"
    
    signature = hmac.new(
        secret_key.encode(),
        signed_content.encode(),
        hashlib.sha256
    ).hexdigest()
    
    headers = {
        'Content-Type': 'application/json',
        'X-Webhook-Signature': f't={timestamp},v1={signature}'
    }
    
    requests.post(url, data=body, headers=headers)
```

Receiver verification with timestamp check:

```python
def verify_webhook_with_timestamp(request, secret_key, max_age_seconds=300):
    signature_header = request.headers.get('X-Webhook-Signature')
    body = request.get_data()
    
    # Parse signature header
    parts = signature_header.split(',')
    timestamp = None
    signature = None
    
    for part in parts:
        key, value = part.split('=')
        if key == 't':
            timestamp = int(value)
        elif key == 'v1':
            signature = value
    
    # Check timestamp is recent
    current_time = int(time.time())
    if current_time - timestamp > max_age_seconds:
        return False
    
    # Verify signature
    signed_content = f"{timestamp}.{body.decode()}"
    expected_signature = hmac.new(
        secret_key.encode(),
        signed_content.encode(),
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(signature, expected_signature)
```

### Common Confusions About Request Signing

**Confusion 1: Signing encrypts the payload**

Signing doesn't encrypt. The payload is still visible to anyone who intercepts it. Signing only proves authenticity and detects tampering. For sensitive data, use HTTPS to encrypt the entire connection.

**Confusion 2: You need public/private key cryptography**

HMAC uses symmetric cryptography. Both sender and receiver know the secret key. This is simpler than public/private key systems and sufficient for webhooks.

**Confusion 3: Signature must be in the body**

Signatures go in headers, not the body. This prevents the signature from being included in the signed content (which would be circular). Standard practice is the `X-Webhook-Signature` header.

### Tips and Cautions

**Tip: Use constant-time comparison.** Always use `hmac.compare_digest()` to compare signatures. Regular string comparison (`==`) is vulnerable to timing attacks where attackers can deduce the correct signature by measuring comparison time.

**Tip: Log signature verification failures.** Every failed signature should be logged. This helps detect attacks or configuration problems. However, don't log the actual signatures or secret keys.

**Caution: Rotate secret keys regularly.** If a secret key is compromised, an attacker can forge signatures. Implement key rotation. Keep multiple versions of the key active during transition periods.

**Caution: Never commit secrets to version control.** Store secret keys in environment variables or secret management systems. Never hardcode them in your code or commit them to Git.

---

## Resilient Failure Handling and Retries

### What Is Resilient Failure Handling? A Relatable Analogy

Imagine you're calling a friend. The first time, you get a busy signal. You don't give up. You wait a bit and try again. The second time, you get no answer. You wait longer and try again. Eventually, you reach your friend. This is resilience.

Your webhook system needs the same persistence. When a webhook delivery fails, don't give up immediately. Retry with increasing delays. If the receiver is temporarily down, it will recover and process the event. If the network is congested, waiting before retrying helps. Resilient systems assume temporary failures are normal and handle them gracefully.

### Why Resilient Failure Handling Matters

Networks are unreliable. Services crash. Servers run out of memory. Databases lock up. These failures are temporary in most cases. A webhook delivery that fails today might succeed tomorrow. Without retry logic, you lose events permanently.

Resilient failure handling ensures your system survives temporary failures. Events are eventually delivered. Systems recover gracefully. Users don't lose data. Critical operations complete despite transient problems.

Proper retry logic also prevents overwhelming a struggling service. If you retry immediately and aggressively, you hammer a recovering service with more requests, making recovery slower. If you retry with exponential backoff (waiting longer between retries), you give the service time to recover while still ensuring eventual delivery.

### Retry Strategies: From Simple to Sophisticated

**Simple Retry: Immediate Retry**

The simplest approach: if delivery fails, try again immediately.

```python
def send_webhook_simple_retry(url, payload, max_attempts=3):
    for attempt in range(max_attempts):
        try:
            response = requests.post(url, json=payload, timeout=10)
            if response.status_code < 500:
                return response
        except requests.RequestException:
            pass
    
    raise Exception("Webhook delivery failed")
```

This is too aggressive. It doesn't give the receiver time to recover. It hammers struggling services. Use this only for development.

**Exponential Backoff**

Better approach: wait longer between retries, with delays growing exponentially.

```python
import time

def send_webhook_with_backoff(url, payload, max_attempts=5):
    for attempt in range(max_attempts):
        try:
            response = requests.post(url, json=payload, timeout=10)
            if response.status_code < 500:
                return response
        except requests.RequestException:
            pass
        
        if attempt < max_attempts - 1:
            # Exponential backoff: 1s, 2s, 4s, 8s
            wait_time = 2 ** attempt
            time.sleep(wait_time)
    
    raise Exception("Webhook delivery failed")
```

This is better. First retry after 1 second, then 2 seconds, then 4 seconds, then 8 seconds. The delays give struggling services time to recover. However, this is still too predictable.

**Exponential Backoff with Jitter**

If multiple webhooks retry at the same time with the same exponential backoff, they all retry simultaneously. This creates **thundering herd** problems where all requests hit the receiver at the same time, overwhelming it.

Add randomness (**jitter**) to stagger retries:

```python
import random

def send_webhook_with_jitter(url, payload, max_attempts=5):
    for attempt in range(max_attempts):
        try:
            response = requests.post(url, json=payload, timeout=10)
            if response.status_code < 500:
                return response
        except requests.RequestException:
            pass
        
        if attempt < max_attempts - 1:
            # Exponential backoff with jitter
            base_wait = 2 ** attempt
            jitter = random.uniform(0, base_wait * 0.1)
            wait_time = base_wait + jitter
            time.sleep(wait_time)
    
    raise Exception("Webhook delivery failed")
```

Now retries are staggered. If 100 webhooks fail, they don't all retry at the exact same time. They retry at slightly different times, spreading load on the receiver.

### Determining Which Errors Are Retryable

Not all failures should be retried. Some indicate permanent problems:

- **500 Internal Server Error**: Retryable. The server had a temporary problem.
- **503 Service Unavailable**: Retryable. The server is overloaded or down.
- **429 Too Many Requests**: Retryable. The server is rate-limiting us. Back off and retry later.
- **400 Bad Request**: Not retryable. The payload is malformed. Retrying won't help.
- **401 Unauthorized**: Not retryable. Authentication failed.
- **404 Not Found**: Not retryable. The endpoint doesn't exist.
- **Network timeout**: Retryable. The network is slow or the server didn't respond.

```python
def is_retryable(response_or_exception):
    if isinstance(response_or_exception, requests.RequestException):
        return True
    
    status_code = response_or_exception.status_code
    retryable_codes = {408, 429, 500, 502, 503, 504}
    return status_code in retryable_codes

def send_webhook_smart_retry(url, payload, max_attempts=5):
    for attempt in range(max_attempts):
        try:
            response = requests.post(url, json=payload, timeout=10)
            if response.status_code < 400:
                return response
            
            if not is_retryable(response):
                return response
        
        except requests.RequestException as e:
            if not is_retryable(e):
                raise
        
        if attempt < max_attempts - 1:
            base_wait = 2 ** attempt
            jitter = random.uniform(0, base_wait * 0.1)
            time.sleep(base_wait + jitter)
    
    raise Exception("Webhook delivery failed")
```

### The Circuit Breaker Pattern

Imagine a circuit breaker in your home. If too much current flows, the breaker trips and cuts power. This prevents fires. Once the problem is fixed, you reset the breaker.

The **circuit breaker pattern** applies this concept to webhooks. If a receiver is consistently failing, stop sending webhooks to it temporarily. This prevents wasting resources on a broken service. Once the service recovers, resume sending.

A circuit breaker has three states:

- **Closed**: Normal operation. Requests are sent and processed normally.
- **Open**: The service is broken. Requests fail immediately without attempting delivery.
- **Half-open**: Testing if the service has recovered. Send a few requests. If they succeed, close the circuit. If they fail, open it again.

```python
import time
from enum import Enum

class CircuitBreakerState(Enum):
    CLOSED = 1
    OPEN = 2
    HALF_OPEN = 3

class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=60):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.state = CircuitBreakerState.CLOSED
        self.failure_count = 0
        self.last_failure_time = None
    
    def call(self, func, *args, **kwargs):
        if self.state == CircuitBreakerState.OPEN:
            if time.time() - self.last_failure_time > self.recovery_timeout:
                self.state = CircuitBreakerState.HALF_OPEN
                self.failure_count = 0
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise
    
    def on_success(self):
        self.failure_count = 0
        self.state = CircuitBreakerState.CLOSED
    
    def on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = CircuitBreakerState.OPEN
```

### Dead Letter Queues

Despite all retry efforts, some webhooks fail permanently. The receiver might be permanently down. The payload might be malformed. The endpoint might be deleted.

**Dead letter queues** (DLQs) are where failed events go. They're like a holding area for messages that couldn't be delivered. You can inspect them later, fix the problem, and reprocess the events.

```python
class DeadLetterQueue:
    def __init__(self):
        self.failed_events = []
    
    def add(self, event, reason, attempt_count):
        dead_letter = {
            'event': event,
            'reason': reason,
            'attempt_count': attempt_count,
            'timestamp': time.time(),
            'status': 'pending'
        }
        self.failed_events.append(dead_letter)
    
    def get_all(self):
        return self.failed_events
    
    def reprocess(self, event_id):
        # Find and reprocess the event
        for entry in self.failed_events:
            if entry['event']['id'] == event_id:
                entry['status'] = 'reprocessing'
                return entry
        return None
```

When a webhook fails after all retries, add it to the DLQ instead of discarding it. This preserves the event for later investigation. You can inspect the DLQ to understand why deliveries failed. Once you fix the problem, you can reprocess events from the DL