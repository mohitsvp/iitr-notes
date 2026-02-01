# Scalability of AI Systems: Pre-Class Notes

## What You'll Learn

In this pre-read, you'll discover:

- **How AI systems break when they get popular** — Understanding why a working AI model suddenly becomes slow and expensive when thousands of users want to use it simultaneously
- **Where time actually gets lost in AI pipelines** — Learning to identify which parts of your AI inference system create bottlenecks and how to measure them
- **Three fundamentally different ways to run AI at scale** — Comparing centralized cloud approaches, serverless architectures, and edge computing to understand their trade-offs
- **The impossible triangle of AI infrastructure** — Exploring how to navigate the constant tension between speed, cost, and the amount of work your system can handle

---

## Introduction: What Is Scalability in AI Systems?

Imagine you've built an amazing restaurant that serves incredible food. When it's just you and a few friends, everything runs smoothly—you cook the meal, plate it beautifully, and everyone's happy within minutes. But what happens when a food critic writes a glowing review and suddenly 500 people want to eat at your restaurant on Friday night? Your kitchen wasn't designed for that volume. Your oven can only cook so much food at once. Your single stove becomes a bottleneck. You run out of ingredients. Delivery times skyrocket from 20 minutes to 2 hours. Customers get frustrated and leave.

**Scalability in AI systems is exactly this problem.** It's the challenge of taking an AI model that works beautifully in a lab or with a small test group and making it work when thousands—or millions—of real users want to use it at the same time. The model itself might be perfect, but the *infrastructure* around it (the servers, networks, and processes that deliver predictions to users) can collapse under real-world demand.

Think of AI inference as a journey: a user sends a question to your system, that question travels across the internet to a server, the server runs your AI model to generate an answer, and then that answer travels back to the user. Each step in this journey takes time. Each step costs money. And when you have millions of users doing this simultaneously, small inefficiencies become massive problems. A system that handles 100 users might completely fail at 10,000 users—not because the AI model is bad, but because the *system* wasn't designed to scale.

**At its core, scalability means:** Can your AI system handle increasing demand while keeping responses fast, costs reasonable, and the quality of predictions consistent?

---

## Why Does Scalability Matter?

### 1. **User Experience Directly Depends on Speed**

When you ask an AI chatbot a question, you expect an answer in seconds, not minutes. If your system can't handle demand, response times get slower. Imagine using ChatGPT and waiting 30 seconds for each response instead of 2 seconds—you'd probably switch to a competitor. In real-world deployments, every extra 100 milliseconds of latency (delay) causes measurable drops in user engagement. This isn't just about annoyance; it's about whether your product survives in a competitive market. Companies that master scalability deliver snappy, responsive experiences. Companies that don't watch their users leave.

### 2. **Costs Can Spiral Out of Control Without Proper Scaling**

Running AI inference is expensive. You need powerful hardware (GPUs and TPUs cost thousands of dollars each), electricity to run them, network bandwidth to move data around, and engineers to maintain everything. A naive approach to scaling—just buying more servers whenever demand increases—can cost 10x more than a thoughtful, optimized approach. Understanding scalability means understanding how to serve more users without your bill multiplying by 10. Companies that optimize their inference pipelines can reduce costs by 40-80% while actually improving speed. That's the difference between a profitable business and one that burns through venture capital.

### 3. **Reliability and Consistency Become Critical**

When your system serves millions of requests daily, failures aren't rare edge cases—they're statistically guaranteed. If your system crashes once per million requests, and you serve a billion requests per day, you're crashing 1,000 times daily. Scalability isn't just about speed and cost; it's about building systems that remain reliable when handling real-world traffic patterns, unexpected spikes, hardware failures, and network problems. A system designed only for happy-path scenarios will fail spectacularly when real users start using it.

---

## Building Understanding: From Known to New

Let's start with something you already understand: **serving a single prediction from an AI model.**

When you have a trained machine learning model (say, a language model that predicts the next word), running a prediction is straightforward. You load the model into memory, feed it an input, it computes an output, and you return the result. This works beautifully when one person is using your system. The model sits in GPU memory, ready to go. A prediction might take 100 milliseconds. Everything is fast and simple.

Now imagine 1,000 people want predictions simultaneously. Here's where the naive approach breaks:

```
Person 1 sends request → Server loads model → Computes output → Person 1 gets response (100ms)
Person 2 sends request → Waits for Person 1 to finish... → Person 2 gets response (200ms)
Person 3 sends request → Waits for Persons 1 & 2... → Person 3 gets response (300ms)
...
Person 1000 → Waits for everyone else → Gets response (100,000ms = 100 seconds!)
```

This is catastrophically bad. Person 1000 waited 100 seconds for a response that should take 100 milliseconds. The system is completely unusable.

**The solution introduces complexity:** Instead of processing requests one at a time, you need to process many requests *simultaneously*. This means:

- **Multiple copies of the model** running on different servers
- **Intelligent routing** that directs each request to the fastest available server
- **Batching** that groups requests together to use hardware more efficiently
- **Caching** that stores frequently-requested results so you don't recompute them
- **Monitoring** that tracks which parts are slow and which are fast

This is where **distributed inference** comes in. Instead of one server handling all requests, you distribute the work across many servers, each running its own copy of the model. But now you have new problems: How do you route requests fairly? How do you keep all the servers in sync? What happens when one server crashes? How do you decide how many servers you need?

These are the core questions of scalability. The shift from "one model, one request" to "many models, many requests" introduces fundamental challenges that require architectural thinking.

---

## Core Components: Understanding the Building Blocks

### 1. **Inference Latency — The Time It Takes to Get an Answer**

**Latency** is the time between when you send a request to an AI system and when you get a response. It's measured in milliseconds. If you ask a chatbot a question and wait 500 milliseconds for an answer, that's latency. Latency matters because humans perceive delays. Anything under 100ms feels instant. Anything over 1 second feels sluggish. For AI applications serving millions of users, even 50ms differences matter enormously.

*Micro-example:* You ask "What's the capital of France?" at 12:00:00.000. The server receives it, runs the model, and sends back "Paris" at 12:00:00.050. Your latency was 50ms—excellent. But if the system is overloaded and you get the response at 12:00:01.500, your latency was 1.5 seconds—the user has probably already closed the app.

### 2. **Throughput — How Many Requests You Can Handle**

**Throughput** is how many requests your system can process in a given time period, usually measured as "requests per second" or "tokens per second" (for language models). A system with high throughput can handle many users simultaneously. A system with low throughput gets overwhelmed easily.

*Micro-example:* Your inference server can process 100 requests per second. If 50 users send requests, you're at 50% capacity—things are fine. But if 200 users send requests, you're overloaded—requests start queuing up, and latency increases for everyone.

### 3. **Distributed Inference — Splitting Work Across Many Servers**

**Distributed inference** means running your AI model across multiple servers instead of just one. There are different strategies:

- **Horizontal scaling:** Run identical copies of your model on multiple servers. Each server handles different requests. Think of it like opening multiple checkout lines at a supermarket—each line is independent.
- **Model parallelism:** Split one large model across multiple servers. Each server holds part of the model. This is necessary for enormous models that don't fit on a single GPU. Think of it like a team assembling a car—each person does part of the work, then passes it to the next person.
- **Pipeline parallelism:** Different stages of processing happen on different servers. Request flows through Server A, then Server B, then Server C. Think of an assembly line in a factory.

*Micro-example:* You have a 70-billion parameter language model that's too big for one GPU. You split it: Server A holds the first 35 billion parameters, Server B holds the next 35 billion. A request arrives, Server A processes it and sends intermediate results to Server B. Server B finishes processing and sends the final result back. This is model parallelism.

### 4. **Latency Decomposition — Understanding Where Time Gets Lost**

**Latency decomposition** means breaking down the total response time into its component parts to identify bottlenecks. For AI inference, latency typically comes from:

- **Network latency:** Time for the request to travel from client to server and back (usually 10-100ms depending on distance)
- **Queuing latency:** Time the request waits in line before being processed (can be 0ms to seconds depending on load)
- **Model execution latency:** Time the model actually spends computing the prediction (100ms to several seconds depending on model size)
- **I/O latency:** Time spent reading/writing data from disk or memory (can be significant if not optimized)

If your total latency is 500ms and you discover that 400ms is queuing latency (the request just waiting in line), then adding more servers is the solution. But if 400ms is model execution latency, adding more servers won't help—you need a faster model or better hardware.

*Micro-example:* You measure end-to-end latency for a chatbot response:
- Network: 20ms (request travels to server)
- Queuing: 150ms (request waits for GPU to be free)
- Model execution: 200ms (GPU runs the model)
- I/O: 30ms (reading cached data)
- Network: 20ms (response travels back)
- **Total: 420ms**

If you add more servers, queuing time drops to 50ms. Now total is 320ms. Much better.

### 5. **Cold Start — The Penalty for Unused Capacity**

**Cold start** refers to the delay when a system that's been idle (not running) suddenly needs to serve requests. Think of it like starting a car engine on a cold morning—it takes extra time to warm up. In AI systems, cold start happens when:

- A new server spins up and needs to load the model into GPU memory (can take 5-30 seconds)
- A serverless function is invoked after being idle (takes 1-5 seconds just to initialize)
- A container needs to start and load dependencies

Cold start is a huge problem for systems with variable traffic. If your demand doubles at certain times of day, you spin up new servers. But during those 10 seconds while they're starting up, your existing servers are overloaded. Users experience terrible latency. Then traffic drops, your new servers sit idle, and when traffic spikes again, you have another cold start penalty.

*Micro-example:* It's 9 AM, traffic surges. Your system needs more capacity, so it starts 10 new GPU servers. Each one takes 15 seconds to load the model. For those 15 seconds, existing servers are handling 10x normal load. Latency shoots up to 5 seconds. Users get frustrated. At 5 PM, traffic drops, those 10 servers sit idle burning money. At 6 PM, traffic spikes again—cold start penalty again.

---

## Step-by-Step Process: How Inference Scaling Actually Works

### Step 1: Measure and Understand Your Current System

Before you can scale, you need to understand what you're working with. Measure your baseline performance: How long does a single prediction take? How many requests per second can one server handle? What's your current latency distribution? (Some requests might be 50ms, others 2 seconds—you need to know the distribution, not just the average.) Without this data, you're flying blind.

**Action:** Use monitoring tools to capture latency, throughput, and resource utilization (CPU, GPU, memory, network). Identify your bottleneck. Is it the model computation? The network? Memory bandwidth? This tells you what to optimize.

### Step 2: Identify Your Bottleneck and Constraints

Different systems have different bottlenecks. For some, the GPU is maxed out—it can't compute predictions faster. For others, the network is the bottleneck—data can't move between servers fast enough. For others, memory is the constraint—you can't fit enough data in GPU memory to batch requests effectively. Your optimization strategy depends entirely on what's actually limiting you.

**Action:** Run profiling tools. If GPU utilization is 100% but network utilization is 10%, your GPU is the bottleneck. If GPU utilization is 30% but network is 95%, your network is the bottleneck. Optimize the bottleneck, not random things.

### Step 3: Choose Your Scaling Architecture

You have three main options, each with different trade-offs:

**Centralized Cloud:** All inference happens in one data center. Simple to manage, but high latency for distant users and expensive if you're serving global traffic.

**Serverless:** You pay only for what you use. The cloud provider manages scaling automatically. But cold start penalties and less predictable latency.

**Edge Computing:** Run models on servers close to users (at the "edge" of the network). Extremely low latency, but more complex to manage.

**Action:** Choose based on your requirements. Global e-commerce site with strict latency requirements? Edge. Internal corporate tool with bursty traffic? Serverless. Consistent, high-volume traffic? Centralized cloud.

### Step 4: Optimize Your Model and Batching Strategy

Once you've chosen an architecture, optimize within it. Can you use a smaller model that's faster? Can you quantize your model (use lower precision numbers) to speed it up? Can you batch requests together to use hardware more efficiently? These optimizations often give 2-10x improvements in latency or throughput with minimal changes.

**Action:** Experiment with quantization (8-bit instead of 16-bit precision), model distillation (training a smaller model to mimic a larger one), and batching strategies. Measure the impact on accuracy and speed.

### Step 5: Monitor, Measure, and Iterate

Scaling is never "done." As your traffic changes, your system's performance changes. What worked for 1,000 requests per second might not work for 10,000. You need continuous monitoring to catch problems early and continuous optimization to stay efficient.

**Action:** Set up alerts for when latency exceeds thresholds. Track cost per prediction. When you notice problems, go back to step 1—measure and understand what changed.

---

## Key Features: The Three Inference Paradigms

Understanding these three approaches is crucial because they represent fundamentally different trade-offs in scalability.

### **Paradigm 1: Centralized Cloud Inference**

All your inference happens in one (or a few) data centers owned by a cloud provider like AWS, Google Cloud, or Azure. You send requests over the internet to these data centers, they run your model, and they send results back.

**Advantages:**
- Simple to set up and manage
- You don't need to worry about maintaining servers
- Easy to scale horizontally—just buy more servers
- Good for consistent, predictable traffic

**Disadvantages:**
- High latency for geographically distant users (physics means data travels at the speed of light; from New York to Tokyo takes ~80ms minimum)
- Expensive for high-volume inference (you pay for every request)
- All traffic goes through the same data center, creating bottlenecks
- Compliance issues if data must stay in specific geographic regions

**When to use it:** Internal tools, non-latency-critical applications, or when your users are concentrated in one geographic region.

*Example:* A company using AWS SageMaker to serve a recommendation model. All requests go to AWS's data center in us-east-1. Works great for US-based users. But users in Europe or Asia experience 100-200ms of network latency just waiting for data to travel.

### **Paradigm 2: Serverless Inference**

You write code that runs your model, upload it to a serverless platform (AWS Lambda, Google Cloud Functions, Azure Functions), and the platform handles all the infrastructure. You pay only for the compute you use, measured in milliseconds.

**Advantages:**
- Pay only for what you use (no idle servers burning money)
- Automatic scaling (the platform spins up new instances when needed)
- Very low operational overhead
- Good for bursty, unpredictable traffic

**Disadvantages:**
- Cold start penalties (first request after idle time is slow)
- Limited execution time (typically 15 minutes max)
- Less control over performance tuning
- Can be expensive if you have consistent, high-volume traffic

**When to use it:** Applications with variable traffic patterns, microservices, or when you want minimal operational overhead.

*Example:* A chatbot that gets 1,000 requests at 9 AM, 100 requests at 10 AM, 5,000 requests at 11 AM. With serverless, you pay for exactly what you use. With centralized cloud, you'd need to provision for peak load (5,000 requests) and waste money during low-traffic periods. But the first request after a quiet period might take 3 seconds due to cold start.

### **Paradigm 3: Edge Inference**

Your model runs on servers distributed geographically around the world, close to where users are. When a user in Tokyo sends a request, it goes to a server in Tokyo, not to a data center in the US. This is the future of AI infrastructure.

**Advantages:**
- Extremely low latency (data doesn't have to travel far)
- Reduced bandwidth costs (processing happens where data is generated)
- Better compliance (you can ensure data stays in specific regions)
- Resilient to outages (if one region fails, others still work)

**Disadvantages:**
- More complex to manage (coordinating models across many locations)
- Potential consistency issues (different edge servers might have slightly different model versions)
- Higher operational complexity
- Requires more sophisticated monitoring and deployment tools

**When to use it:** Global applications where latency is critical, applications with strict data residency requirements, or when you need maximum resilience.

*Example:* A real-time video analysis system. Video is captured on a smartphone in London. Instead of sending all that video data to a US data center, running inference, and sending results back (high latency, high bandwidth), the model runs on a server in London. Inference happens locally. Results are instant. Bandwidth usage is minimal.

---

## Cost-Latency-Throughput Trade-Offs: The Impossible Triangle

Here's the fundamental truth about AI infrastructure: **You cannot optimize for all three simultaneously.** You can optimize for any two, but the third suffers. This is why making architectural decisions is hard.

### **The Three Dimensions:**

**Latency:** How fast you respond to individual requests. Measured in milliseconds.

**Throughput:** How many requests you can handle simultaneously. Measured in requests per second.

**Cost:** How much money you spend per prediction. Measured in dollars per thousand predictions.

### **The Trade-Offs:**

**Optimize for Low Latency + High Throughput → High Cost**

To serve requests incredibly fast *and* handle massive volume, you need lots of powerful hardware. You might run 1,000 GPUs constantly, even if they're not all being used. This is expensive. Think of it like keeping 100 ambulances on standby 24/7 to ensure response time is under 5 minutes. Yes, response time is great. But you're paying for ambulances that sit idle 90% of the time.

*Real example:* A trading firm running AI models for high-frequency trading needs sub-millisecond latency and can handle millions of predictions per second. They maintain a massive GPU cluster that costs $50 million annually. But latency is critical for their business model, so it's worth it.

**Optimize for Low Latency + Low Cost → Low Throughput**

You can keep costs down by using cheaper hardware or fewer servers. But then you can't handle many simultaneous requests. The first user gets a fast response. The 100th user waits in a long queue.

*Real example:* A startup using a single GPU server to serve an image recognition model. Latency is great (100ms) because the GPU is fast. Cost is low ($500/month). But throughput is limited—only 10 requests per second. If you get 100 simultaneous users, 90 of them are waiting.

**Optimize for High Throughput + Low Cost → High Latency**

You can handle massive volume cheaply by using many cheap servers and batching requests together. But batching introduces delay. Requests wait for a batch to fill up before processing starts. Latency increases.

*Real example:* A batch processing system for analyzing millions of images overnight. Cost is low (you use cheap spot instances and run everything at 3 AM when compute is cheaper). Throughput is enormous (you process 100,000 images). But individual latency is high—if you submit an image at 10 PM, results come back at 3 AM.

### **The Sweet Spot: Understanding Your Trade-Off**

The key is understanding *your* constraints. What matters most for your application?

- **Real-time chatbot?** Optimize for low latency. Users will tolerate slightly higher costs if responses are instant.
- **Batch video processing?** Optimize for low cost. Users don't care if results take hours.
- **Mobile app with millions of users?** Optimize for throughput. You need to handle spikes without crashing.

Once you know your priority, the architecture choice becomes clear.

### **Practical Example: The Same Model, Three Different Deployments**

Let's say you have a language model (like a smaller version of GPT). Same model. Three different deployments:

**Deployment A: Low-Latency Edge**
- 100 edge servers worldwide
- Each processes 10 requests/second
- Total throughput: 1,000 requests/second
- Latency: 50ms average (data doesn't travel far)
- Cost: $5 per 1,000 predictions
- Annual cost for 10 billion predictions: $50 million

**Deployment B: Balanced Cloud**
- 10 cloud servers in one data center
- Each processes 100 requests/second
- Total throughput: 1,000 requests/second
- Latency: 200ms average (network travel + queuing)
- Cost: $0.50 per 1,000 predictions
- Annual cost for 10 billion predictions: $5 million

**Deployment C: Serverless Batch**
- Serverless platform, pay per execution
- Throughput: 10,000 requests/second (highly scalable)
- Latency: 500ms average (cold starts + queuing)
- Cost: $0.10 per 1,000 predictions
- Annual cost for 10 billion predictions: $1 million

Same model. Same accuracy. Different trade-offs. Which do you choose? It depends on your requirements.

---

## Latency Decomposition Deep Dive: Where Does Time Actually Go?

Let's get concrete about where latency comes from. When a user sends a request to an AI system, here's what happens:

```
User sends request (0ms)
  ↓
Request travels over internet to server (10-100ms) [NETWORK LATENCY]
  ↓
Request arrives at server, waits in queue (0-1000ms) [QUEUING LATENCY]
  ↓
Server loads request into GPU memory (1-5ms) [I/O LATENCY]
  ↓
GPU runs the model:
  - Prefill phase: Process all input tokens (50-200ms) [COMPUTATION]
  - Decode phase: Generate output tokens one by one (100-500ms) [COMPUTATION]
  ↓
Response is ready (total computation: 150-700ms) [COMPUTATION LATENCY]
  ↓
Response travels over internet back to user (10-100ms) [NETWORK LATENCY]
  ↓
User receives response (total time: 180-1900ms)
```

### **Understanding Prefill vs. Decode**

This is crucial for language models. When you ask a chatbot a question, the model works in two phases:

**Prefill Phase:** The model reads your entire question and builds a representation of it. This phase is **compute-bound**—it benefits from having many powerful cores working in parallel. GPUs excel at this. Prefill is relatively fast—50-200ms for typical prompts.

**Decode Phase:** The model generates your answer one token at a time (a token is roughly a word). Each token generation requires reading the entire model from memory. This phase is **memory-bound**—it's limited by how fast data can move from memory to the GPU. Even the fastest GPU can't generate tokens faster than memory can feed data to it. Decode is slower—100-500ms depending on answer length.

*Why this matters:* If you're optimizing a chatbot, you need to understand which phase is your bottleneck. If users complain about slow responses, the problem might be:

- Slow prefill? Upgrade to a faster GPU or use a smaller model.
- Slow decode? Add more memory bandwidth or use quantization to reduce memory requirements.
- Long queuing? Add more servers.

Different problems need different solutions.

### **Real Latency Breakdown Example**

Imagine you're running a customer service chatbot. A customer asks: "What's the status of my order #12345?"

```
Network (customer to server):              30ms
Queuing (waiting for GPU):                 100ms
I/O (loading request into GPU memory):     5ms
Prefill (processing the question):         80ms
Decode (generating the answer):            250ms
I/O (writing response):                    5ms
Network (server to customer):              30ms
─────────────────────────────────────────────
TOTAL LATENCY:                             500ms
```

If you want to reduce latency, where should you focus?

- The 30ms network latency is hard to reduce (physics). It's the speed of light.
- The 100ms queuing latency suggests you need more servers.
- The 80ms prefill is decent.
- The 250ms decode is the biggest component. This is where optimization helps most.

By optimizing decode (maybe through quantization or a faster model), you could reduce it to 100ms, bringing total latency to 350ms. That's 30% faster.

---

## The Cold Start Problem: Why New Servers Are Slow

When traffic spikes, your system needs to spin up new servers. But those new servers are "cold"—they're starting from scratch. Here's what happens:

```
Traffic is normal. You have 5 servers running.
Each server is 80% utilized.
  ↓
Traffic doubles suddenly.
Your system detects this: "We need more capacity!"
  ↓
It spins up 5 new servers.
Each new server must:
  1. Boot the operating system (2-5 seconds)
  2. Load Python/CUDA libraries (3-8 seconds)
  3. Load the model from disk into GPU memory (5-30 seconds depending on model size)
  ↓
For 15-30 seconds, the new servers are useless.
During this time, only your original 5 servers are handling 2x traffic.
They're now 160% utilized (overloaded).
  ↓
Latency shoots up to 5-10 seconds for affected users.
Users get frustrated.
  ↓
Finally, new servers are ready.
Load is distributed across 10 servers.
Latency drops back to normal.
  ↓
But now you've damaged user experience during the spike.
```

### **Solutions to Cold Start**

**1. Warm Pools:** Keep extra servers running (but not fully loaded) so they're ready if traffic spikes. Trade-off: You pay for servers that might not be needed.

**2. Predictive Scaling:** Use historical data to predict when traffic will spike and pre-warm servers before the spike hits. Trade-off: Requires data science to predict accurately.

**3. Serverless with Provisioned Concurrency:** Pay a bit extra to keep a certain number of serverless instances warm. Trade-off: Less flexible than pure serverless, but no cold starts.

**4. Model Caching:** Keep the model in memory on all servers, even if they're not serving requests. Trade-off: Wastes memory and money.

**5. Faster Model Loading:** Use techniques like memory mapping or incremental loading to reduce cold start time from 30 seconds to 5 seconds. Trade-off: More complex engineering.

---

## Putting It All Together: A Real-World Scaling Scenario

Let's trace through a complete example to tie everything together.

### **The Scenario**

You're building an AI-powered customer support chatbot for an e-commerce company. The model is a 7-billion parameter language model. Current state:

- Running on 2 GPU servers in AWS
- Average latency: 400ms
- Throughput: 200 requests/second
- Cost: $10,000/month
- Business problem: During peak hours (8 AM - 5 PM), the system is overloaded. Latency reaches 2 seconds. Customers complain.

### **Step 1: Measure and Decompose Latency**

You instrument the system to measure where time is spent:

```
Network latency (customer to server):       40ms
Queuing (waiting for GPU):                  180ms  ← BIGGEST COMPONENT
I/O (loading into GPU):                     5ms
Prefill (processing customer message):      60ms
Decode (generating response):               100ms
I/O (writing response):                     5ms
Network (server to customer):               40ms
────────────────────────────────────────────────
TOTAL:                                      430ms
```

**Discovery:** 180ms of your 430ms latency is queuing. The GPU isn't the bottleneck—you just don't have enough servers. Adding more servers will directly reduce queuing latency.

### **Step 2: Choose an Architecture**

You have options:

**Option A: Scale centralized cloud horizontally**
- Add more servers in AWS
- Cost: +$5,000/month per server
- Latency: Would drop to ~250ms
- Throughput: Would increase to 400 requests/second

**Option B: Switch to serverless**
- Use AWS Lambda with SageMaker
- Cost: ~$0.001 per request (much cheaper at scale)
- Latency: 300-500ms (cold starts add uncertainty)
- Throughput: Unlimited (automatic scaling)

**Option C: Deploy to edge**
- Run the model on servers in multiple AWS regions
- Cost: +$20,000/month (more infrastructure)
- Latency: 100-150ms (users connect to nearest region)
- Throughput: 400+ requests/second

### **Step 3: Make a Decision**

Business requirements:
- Must serve global customers (rules out single-region centralized cloud)
- Cost is important (serverless is tempting)
- Latency matters (customers get frustrated with 2-second delays)

**Decision: Deploy to 3 AWS regions (US, Europe, Asia)**

This gives you:
- Latency: 120-180ms (users connect to nearest region)
- Throughput: 600 requests/second
- Cost: ~$25,000/month (more expensive but you're solving the problem)

### **Step 4: Optimize Within the Architecture**

Now that you've chosen multi-region deployment, optimize further:

- **Quantize the model:** Convert from 16-bit to 8-bit precision. Reduces memory requirements, speeds up inference by 20%, and reduces GPU memory needed by 50%.
- **Implement batching:** Group requests together. Instead of processing 1 request at a time, process 32 requests together. This increases latency slightly (requests wait for a batch to fill) but increases throughput dramatically.
- **Add caching:** Cache responses to common questions. If 10 customers ask "What's your return policy?" you only run inference once.

These optimizations reduce latency to 100ms and increase throughput to 1,000 requests/second while reducing costs to $15,000/month.

### **Step 5: Monitor and Iterate**

You deploy the system. For the first month, you monitor:

- Average latency: 105ms ✓ (target was 200ms)
- p99 latency: 400ms (99% of requests are faster than 400ms—acceptable)
- Throughput: 950 requests/second
- Cost: $14,800/month ✓ (under budget)
- Error rate: 0.01% (excellent)

But you notice something: During 2-3 AM, latency spikes to 2 seconds for 30 seconds. Investigation reveals: Your cold start process is kicking in. Servers are restarting. Solution: Implement warm pools so extra capacity is always ready.

This is the reality of scaling: it's never done. You measure, optimize, deploy, monitor, discover new problems, and iterate.

---

## Practice Exercises: Test Your Understanding

### Pattern Recognition — Identify the Bottleneck**

You're running an AI image recognition system. You measure:

- GPU utilization: 95%
- Network utilization: 20%
- Memory utilization: 60%
- Average latency: 500ms
- Throughput: 100 images/second

**Question:** What's your bottleneck, and what should you optimize?

**Hint:** Look at what's maxed out. What resource is at 95%?