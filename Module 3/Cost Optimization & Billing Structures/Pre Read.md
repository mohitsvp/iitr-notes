# Cost Optimization & Billing Structures: Pre-Class Notes

## What You'll Learn

In this pre-read, you'll discover:

- **How cloud AI costs actually work** – Understanding the three main cost drivers (compute, tokens, and storage) and why they matter to your bottom line
- **The difference between paying-as-you-go and reserving capacity** – When to use each model and how to make the right choice for your situation
- **Practical strategies to reduce costs without sacrificing performance** – Including batching, prompt optimization, and smart resource allocation
- **How to monitor and control costs like a pro** – Setting up tracking systems that show you exactly where your money goes and which features or users are most expensive

---

## Introduction: What Is Cost Optimization for AI?

Imagine you're running a restaurant. At first, you might buy ingredients daily (paying for exactly what you need each day). But if you always use the same amount of flour and eggs, it makes sense to buy in bulk at a discount. That's the core idea behind **cost optimization for AI systems**. 

**Cost optimization is the practice of making your AI systems run efficiently while spending the least amount of money possible.** It's not about cutting corners or making things slower—it's about being intentional with your spending, just like a smart business owner.

In cloud AI, you're paying for three main things: the computing power (GPUs and processors) that run your models, the **tokens** (small units of text that AI models process), and the storage space for your data. Each one has different pricing models, and understanding how they work is your first step toward controlling costs.

---

## Importance: Why Does Cost Optimization Matter?

### 1. **Your Budget Stays Under Control**

Without cost optimization, AI expenses can spiral unexpectedly. A single popular AI feature could suddenly cost thousands per month as more users interact with it. By tracking and optimizing costs, you know exactly what to expect and can predict your monthly bills with confidence. This means fewer surprises and more money left for other projects.

### 2. **You Can Scale Your AI Faster**

When you optimize costs, you free up budget to build more features and serve more users. Instead of spending $10,000 per month on inefficient infrastructure, you might spend $3,000 and use that extra $7,000 to expand your AI capabilities. It's like getting a 3x bigger budget without actually spending more.

### 3. **You Gain Competitive Advantage**

Companies that master cost optimization can offer cheaper services to customers or higher profit margins. If your competitor pays twice as much for the same AI capabilities, you win. Plus, cost-conscious practices often lead to faster, more reliable systems as a bonus.

---

## Building Understanding: From Known to New

Let's start with what you probably already know: **paying for things as you use them**. If you use a video streaming service, you might pay per movie or per month. Simple, right?

Now imagine this painful scenario: You're running an AI chatbot that answers customer questions. Every customer question costs you money in tokens. Without any optimization, here's what happens:

- Customer asks a long, rambling question → you send their entire question to the AI model → you get charged for every word
- The AI gives a long, verbose answer → you get charged for every word in the response
- You do this 1,000 times per day → your bill keeps growing

The "painful way" would be just accepting these costs. But there's a better approach.

**The solution:** By understanding pricing models, batching requests, and optimizing prompts, you can cut costs dramatically. Instead of processing questions one-by-one at full price, you can batch them together for a 50% discount. Instead of sending the entire customer message, you can extract only the relevant parts, cutting tokens in half. Suddenly, the same work costs 70% less.

This is where **FinOps** (Financial Operations for cloud) comes in—it's a systematic approach to making these smart decisions across your entire organization.

---

## Core Components: The Three Cost Drivers

### 1. **Compute Costs (The Power to Run Models)**

**What it is:** The cost of using processing power—typically GPUs (graphics processors) or specialized AI chips—to run your AI models.

**How it works:** When you run an AI model, it needs a computer with enough power. Cloud providers rent you this power by the hour. A powerful GPU like an NVIDIA H100 might cost $1.89 to $2.74 per hour, while a smaller one might cost just $0.09 per hour.

**Example:** Running a large language model for 24 hours on an H100 GPU costs roughly $45–$65. If you run it for a month, that's $1,350–$1,950 just for compute.

**Key insight:** You can choose between paying full price for flexibility (called **on-demand**) or paying less upfront to reserve capacity for a longer period (**reserved instances**).

### 2. **Token Costs (The Price of Processing Text)**

**What it is:** The cost of processing text through an AI model, measured in **tokens** (roughly 4 characters or 1 word).

**How it works:** Every time you send text to an AI API, you pay for the input tokens (your question) and output tokens (the answer). Prices vary by model—GPT-4 is more expensive than GPT-3.5, and output tokens usually cost more than input tokens.

**Example:** Sending a 100-word question and getting a 50-word answer might cost you $0.001 to $0.01, depending on the model. With 10,000 users doing this daily, costs add up fast.

**Key insight:** Token costs are where most teams find the biggest savings opportunities through optimization techniques like batching and prompt compression.

### 3. **Storage Costs (Keeping Your Data Safe)**

**What it is:** The cost of storing data in the cloud—including model weights, training data, vector databases, and logs.

**How it works:** Cloud providers charge monthly fees for storage, usually per gigabyte. Keeping unnecessary data costs money, and it adds up over time.

**Example:** Storing 1 TB of training data might cost $20–$50 per month, depending on the storage type and region. A year of storing old logs could cost hundreds.

**Key insight:** Regular data cleanup and smart retention policies (keeping what matters, deleting what doesn't) can cut storage costs by 30–50%.

---

## Pricing Models: Pay-Per-Use vs. Reserved Capacity

Understanding these two models is crucial because they represent different strategies for different situations.

### **Pay-Per-Use (On-Demand) Pricing**

**How it works:** You pay only for what you use, when you use it. No long-term commitment. It's like paying per ride instead of buying a car.

**Pros:**
- Maximum flexibility—scale up or down instantly
- No waste if demand drops
- Perfect for unpredictable workloads

**Cons:**
- Higher hourly rates (30–50% more expensive than reserved)
- Costs can spike unexpectedly if demand surges
- Hard to predict monthly bills

**Best for:** Development environments, experiments, or workloads with highly variable demand (like seasonal AI features).

### **Reserved Capacity (Committed Pricing)**

**How it works:** You commit to using a certain amount of compute or throughput for a set period (typically 1 month to 1 year) and get a significant discount—often 30–70% off.

**Pros:**
- Much cheaper per unit (30–70% savings)
- Predictable costs—you know exactly what you'll pay
- Guaranteed access to resources

**Cons:**
- Less flexibility—you're locked in for the commitment period
- Wasted money if you don't use the full capacity
- Requires accurate forecasting

**Best for:** Production workloads with predictable, steady demand (like a chatbot that handles consistent daily traffic).

### **Hybrid Approach (The Smart Way)**

Most teams use a combination: reserve capacity for predictable baseline traffic and use on-demand for spikes. For example:

- Reserve 70% of your expected compute for production AI features
- Use on-demand for the remaining 30% to handle traffic spikes
- Use on-demand exclusively for development and experiments

This balance gives you cost savings without sacrificing flexibility.

---

## Batching: The 50% Cost Reduction Opportunity

**Batching** is one of the most powerful cost-saving techniques available. Here's why it works and how to use it.

### **What Is Batching?**

Instead of processing requests one-at-a-time (real-time), you collect multiple requests and process them together in a batch. Most AI providers (OpenAI, Google Gemini, Together AI) offer **batch APIs** that cost roughly **50% less** than real-time APIs.

**Example:** Instead of processing 100 customer questions one-by-one throughout the day, you collect them and process them all together at night. Same results, half the cost.

### **When to Use Batching**

Batching works best when:
- You don't need immediate answers (latency doesn't matter much)
- You have many similar requests
- You can wait hours or even a day for results

**Perfect use cases:**
- Generating marketing copy for 10,000 products
- Classifying customer feedback from yesterday's interactions
- Analyzing large datasets for insights
- Creating summaries of documents overnight

**Not ideal for:**
- Real-time chatbots (users expect instant replies)
- Live customer support
- Anything where speed is critical to user experience

### **How Batching Works in Practice**

1. **Collect requests** throughout the day or week
2. **Format them** in the batch API's required format (usually JSON files)
3. **Submit the batch** to the provider
4. **Wait** for processing (could be hours or overnight)
5. **Retrieve results** when complete

**Code example (pseudocode):**

```
# Collect requests throughout the day
requests = []

for user_question in incoming_questions:
    requests.append({
        "prompt": user_question,
        "model": "gpt-4",
        "max_tokens": 500
    })

# Submit batch at night
batch_job = submit_batch_api(requests)

# Next morning, retrieve results
results = batch_job.get_results()

# Process results
for result in results:
    save_to_database(result)
```

### **Real Cost Savings**

Let's do the math. Suppose you have 1,000 customer questions per day:

**Real-time approach:**
- 1,000 questions × $0.01 per question = $10/day = $300/month

**Batch approach:**
- 1,000 questions × $0.005 per question = $5/day = $150/month

**Savings: $150/month, or 50%**

Now imagine you have 100,000 questions per day. That's $5,000/month saved. Batching is that powerful.

---

## Token Cost Optimization: Making Every Token Count

Tokens are the most controllable cost in AI. Here are the top strategies to reduce token usage without sacrificing quality.

### **Strategy 1: Optimize Your Prompts**

Your prompt is the instruction you send to the AI model. Longer prompts = more tokens = higher costs.

**Technique: Be specific but concise**

❌ **Wasteful prompt (89 tokens):**
```
Please analyze the following customer feedback. I would really appreciate it if you could take a careful look at what the customer is saying and provide me with your thoughts on whether this feedback is positive or negative. Please be thorough in your analysis.

Customer feedback: "The product works well."
```

✅ **Optimized prompt (25 tokens):**
```
Classify as positive or negative: "The product works well."
```

Same task, 72% fewer tokens, same quality result.

**Technique: Use system prompts efficiently**

Instead of repeating instructions in every prompt, use a system prompt (sent once) that defines the AI's behavior for the entire conversation. This saves tokens on repeated instructions.

### **Strategy 2: Extract Relevant Context**

Don't send the entire customer message or document. Extract only the relevant parts.

**Example:** If a customer asks about a specific order, don't send their entire 5-year chat history. Extract just the last 5 messages related to that order. You might cut 80% of the tokens.

### **Strategy 3: Use Cheaper Models When Possible**

Different models have different prices:

- **GPT-3.5**: ~$0.0005 per 1K input tokens (cheapest)
- **GPT-4o**: ~$0.005 per 1K input tokens (mid-range)
- **GPT-4**: ~$0.03 per 1K input tokens (most expensive)

For simple tasks (classification, extraction), GPT-3.5 or smaller models work great and cost 90% less. Reserve GPT-4 for complex reasoning tasks.

**Smart routing:** Use different models for different tasks:
- Customer sentiment analysis → GPT-3.5
- Complex problem-solving → GPT-4
- Code generation → GPT-4o

### **Strategy 4: Implement Caching**

If the same user asks similar questions, you're paying for similar tokens repeatedly. Cache the results.

**Example:** If 100 users ask "What's your refund policy?" you process it once and serve cached results to the other 99 users. That's 99 fewer API calls.

### **Strategy 5: Compress Prompts Intelligently**

Remove filler words while keeping meaning:

```
Original: "Could you please help me understand what the main themes are in this text?"
Compressed: "Extract main themes:"
```

Aim to cut 20–30% of words without losing clarity.

---

## Monitoring Costs: Tracking Per-Feature and Per-User

You can't control what you don't measure. Here's how to set up cost tracking that actually tells you what's happening.

### **Why Granular Tracking Matters**

Without detailed tracking, you only know your total monthly bill. With granular tracking, you know:
- Which features are most expensive
- Which users consume the most tokens
- Which teams are driving costs
- Where optimization efforts will have the biggest impact

### **How to Track Costs Effectively**

**Step 1: Add metadata tags to every API call**

When you call an AI API, attach tags that identify:
- User ID
- Feature name
- Team/department
- Request type

```python
# Example: Python code with metadata tagging
response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[{"role": "user", "content": prompt}],
    metadata={
        "user_id": "user_12345",
        "feature": "customer_support_bot",
        "team": "customer_success",
        "request_type": "classification"
    }
)
```

**Step 2: Log token usage and costs**

Extract the token counts from the response and log them:

```python
# Extract token information
prompt_tokens = response.usage.prompt_tokens
completion_tokens = response.usage.completion_tokens
total_tokens = response.usage.total_tokens

# Calculate cost (example rates)
cost = (prompt_tokens * 0.0005 + completion_tokens * 0.0015) / 1000

# Log to your database
log_event({
    "timestamp": now(),
    "user_id": "user_12345",
    "feature": "customer_support_bot",
    "prompt_tokens": prompt_tokens,
    "completion_tokens": completion_tokens,
    "total_tokens": total_tokens,
    "cost": cost
})
```

**Step 3: Aggregate and analyze**

Query your logs to understand spending patterns:

```sql
-- Cost per feature (last 30 days)
SELECT 
    feature, 
    SUM(total_tokens) as tokens,
    SUM(cost) as total_cost
FROM llm_usage
WHERE timestamp > now() - 30 days
GROUP BY feature
ORDER BY total_cost DESC;

-- Cost per user (top 10 users)
SELECT 
    user_id, 
    SUM(total_tokens) as tokens,
    SUM(cost) as total_cost
FROM llm_usage
WHERE timestamp > now() - 30 days
GROUP BY user_id
ORDER BY total_cost DESC
LIMIT 10;
```

### **What to Look For**

Once you have data, ask these questions:

- **Which features are most expensive?** → Prioritize optimizing these
- **Are there power users?** → Consider special pricing or rate limits
- **What's the cost trend?** → Is it growing? Stabilizing?
- **Which models are most used?** → Can you switch cheaper models?
- **Peak usage patterns?** → When should you batch vs. use real-time?

---

## FinOps: Building a Cost-Conscious Culture

**FinOps** is the discipline of bringing financial accountability to cloud spending. It's not just a technical practice—it's a team practice.

### **The Three Pillars of FinOps**

**1. Visibility**

Everyone on the team should see costs. Not just the finance department—engineers, product managers, and executives. When people see the cost impact of their decisions, they make better choices.

**How to achieve it:**
- Share cost dashboards with the team
- Show cost per feature in your product metrics
- Make cost data as accessible as performance data

**2. Accountability**

Assign cost ownership. Each team or feature should have someone responsible for its costs.

**How to achieve it:**
- Assign cost owners to each major feature
- Include cost metrics in team goals
- Review costs in team meetings

**3. Optimization**

Make cost optimization a continuous practice, not a one-time project.

**How to achieve it:**
- Set monthly cost reduction targets (e.g., "reduce costs by 5%")
- Hold regular optimization reviews
- Celebrate wins when costs go down

### **FinOps Best Practices**

**Practice 1: Right-Sizing Resources**

Use the smallest resources that still meet your needs. Don't use an H100 GPU when an A100 works fine. Don't run development on production-grade hardware.

**Practice 2: Separate Workloads**

- **Development:** Use cheap, on-demand resources
- **Testing:** Use reserved capacity (predictable workload)
- **Production:** Use a mix of reserved (baseline) and on-demand (spikes)

**Practice 3: Automate Cost Controls**

Set up alerts and automatic limits:
- Alert when daily costs exceed threshold
- Automatically shut down idle resources
- Pause expensive experiments if they exceed budget

**Practice 4: Regular Reviews**

Monthly cost reviews with stakeholders:
- What changed?
- What worked?
- What should we try next?

### **Real-World Example: E-Commerce AI Feature**

Let's say you launched an AI feature that recommends products to customers. Initial costs: $5,000/month.

**Month 1 - Visibility:** You discover the feature uses GPT-4 for every recommendation (expensive).

**Month 2 - Optimization:** You switch to GPT-3.5 for simple recommendations and GPT-4 only for complex cases. Cost: $2,000/month (60% savings).

**Month 3 - Batching:** You batch overnight recommendations instead of real-time. Cost: $1,000/month (another 50% savings).

**Month 4 - Prompt Optimization:** You shorten your prompts. Cost: $700/month (another 30% savings).

**Result:** Went from $5,000 to $700 (86% savings) while keeping the same feature quality.

---

## Putting It All Together: A Complete Cost Optimization Strategy

Let's walk through a realistic scenario: You're building an AI customer support system. Here's how to apply everything we've learned.

### **The Scenario**

You have 10,000 customers. Each day:
- 500 customers use the AI chatbot in real-time (need instant replies)
- 5,000 customer support tickets are processed overnight (can wait for batch processing)

**Initial setup (before optimization):**
- Real-time chatbot: 500 requests/day × $0.01/request = $5/day
- Overnight tickets: 5,000 requests/day × $0.01/request = $50/day
- **Total: $55/day = $1,650/month**

### **Step 1: Choose the Right Pricing Model**

- Real-time chatbot: Use pay-per-use (unpredictable demand)
- Overnight tickets: Reserve batch capacity (predictable volume)

**Savings: 15–20%** ($250–$330/month)

### **Step 2: Optimize Prompts and Models**

- Real-time: Use GPT-3.5 for classification, GPT-4 for complex issues (split 80/20)
- Overnight: Use GPT-3.5 for everything
- Shorten prompts by 30%

**Savings: 40%** ($660/month)

### **Step 3: Implement Batching**

- Process overnight tickets in batch (50% discount)

**Savings: 25%** ($412/month)

### **Step 4: Add Caching**

- Cache answers to common questions
- 30% of requests hit cache (no API call needed)

**Savings: 15%** ($247/month)

### **Final Result**

```
Initial cost:        $1,650/month
After optimization:  $400/month
Total savings:       $1,250/month (76% reduction)
```

Same feature quality, dramatically lower cost.

---

## Storage Optimization: The Forgotten Opportunity

While tokens get most attention, storage costs compound over time. Here's how to optimize.

### **Common Storage Waste**

- **Old logs:** Keeping detailed logs forever costs money
- **Duplicate data:** Multiple copies of the same training data
- **Unused models:** Old model versions taking up space
- **Inactive user data:** Data from deleted accounts still stored

### **Storage Optimization Strategy**

**1. Implement data retention policies:**
- Keep detailed logs for 30 days
- Archive to cheaper storage after 30 days
- Delete after 1 year

**2. Use appropriate storage tiers:**
- **Hot storage:** Frequently accessed data (expensive)
- **Warm storage:** Occasionally accessed (cheaper)
- **Cold storage:** Rarely accessed (cheapest)

**3. Clean up regularly:**
- Delete old model versions
- Remove duplicate data
- Archive completed experiments

**Impact:** Can reduce storage costs by 30–50% without affecting performance.

---

## Practice Exercises

### **Exercise 1: Cost Detective**

You're looking at your monthly AI bill and notice it jumped from $3,000 to $5,000. Using the concepts from this pre-read, what are the three most likely causes?

**Hint:** Think about the three cost drivers (compute, tokens, storage). What changed?

---

### **Exercise 2: Model Selection**

You need to classify 100,000 customer reviews as positive, negative, or neutral. Each review is about 50 words. You have three options:

- **Option A:** GPT-4 at $0.03 per 1K input tokens
- **Option B:** GPT-3.5 at $0.0005 per 1K input tokens
- **Option C:** Open-source model on your own GPU ($0.50/hour, takes 2 hours)

Calculate the cost of each option. Which would you choose and why?

**Hint:** Estimate tokens (roughly 1 word = 1.3 tokens). Calculate total cost for each.

---

### **Exercise 3: Pricing Model Decision**

You're building an AI feature for your product. Usage will vary:
- Summer: 10,000 API calls/day
- Winter: 2,000 API calls/day
- API cost: $0.01 per call

Each pricing model costs:
- **Pay-per-use:** Full price ($0.01 per call)
- **Reserved (annual):** $2,000/month flat fee

Which pricing model would you choose and why? Calculate your annual cost for each.

**Hint:** Calculate costs for both models across the full year. Consider which is cheaper on average.

---

### **Exercise 4: Spot the Opportunity**

You're reviewing your token usage logs and notice:
- Feature A: 1 million tokens/day, used by 500 users
- Feature B: 2 million tokens/day, used by 50 users
- Feature C: 500,000 tokens/day, used by 10,000 users

Which feature should you prioritize for optimization and why?

**Hint:** Think about impact. Optimizing which feature saves the most money?

---

### **Exercise 5: Build Your Strategy**

Imagine you're launching a new AI feature that will process 1,000 customer requests per day. Using everything from this pre-read, outline your cost optimization strategy:

1. What pricing model would you use?
2. How would you monitor costs?
3. What optimization techniques would you apply?
4. How would you measure success?

**Hint:** Think about the complete picture: pricing, monitoring, optimization, and culture.

---

## Key Takeaways

1. **Costs have three main drivers:** Compute (GPU power), tokens (text processing), and storage. Understanding each helps you optimize effectively.

2. **Choose your pricing model strategically:** Pay-per-use for unpredictable workloads, reserved capacity for predictable ones. Hybrid approaches work best.

3. **Batching is powerful:** Processing requests in batches costs roughly 50% less. Use it for non-urgent workloads.

4. **Tokens are your biggest optimization opportunity:** Optimize prompts, use cheaper models, implement caching, and compress requests.

5. **Monitor granularly:** Track costs per feature and per user. You can't optimize what you don't measure.

6. **FinOps is a team practice:** Make cost visibility, accountability, and optimization a shared responsibility across your organization.

7. **Small optimizations compound:** A 10% savings here, a 20% savings there—they add up to 60–80% reductions when done systematically.

---

## Ready for Class?

You now understand the fundamentals of cost optimization for AI. You know the pricing models, the optimization techniques, and how to think about costs systematically. In the upcoming class, we'll dive deeper into real-world implementations, advanced monitoring tools, and how to build cost optimization into your product strategy from day one.

Come ready to ask questions and think about how these concepts apply to your specific use cases. Cost optimization isn't one-size-fits-all—it's about finding the right balance for your situation.