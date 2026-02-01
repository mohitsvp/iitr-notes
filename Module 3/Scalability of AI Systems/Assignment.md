# Assignment: Scalability of AI Systems

## Learning Objectives
By completing this assignment, you will:
- Understand scalability constraints in AI inference systems
- Analyze sources of latency across distributed AI pipelines
- Compare centralized, serverless, and edge inference paradigms
- Evaluate cost–latency–throughput trade-offs

---

## Section 1: Objective Questions (Multiple Choice & Multiple Select)

### Easy Questions (Knowledge-Based Understanding)

**Question 1: Multiple Choice**

In the context of AI inference systems, what does "latency" primarily refer to?

A) The total amount of data processed per second  
B) The time delay between when a request is sent and when the result is returned  
C) The cost of running an inference model on cloud servers  
D) The number of concurrent users a system can handle  

**Correct Answer:** B

**Explanation:** Latency is the time it takes for a request to travel through the inference pipeline and receive a response. This is critical for real-time applications like autonomous vehicles or interactive chatbots, where milliseconds matter. Options A, C, and D refer to throughput, cost, and concurrency respectively—all important but distinct from latency.

---

**Question 2: Multiple Choice**

Which of the following best describes the "cold-start problem" in serverless AI inference?

A) The system fails to initialize GPU memory properly  
B) The initial delay when a serverless function spins up compute resources after idle time  
C) The cost of training a model from scratch  
D) The process of downloading a model from cloud storage  

**Correct Answer:** B

**Explanation:** Cold-start refers to the delay experienced when a serverless inference endpoint hasn't been used for a while and needs to provision compute resources (like GPUs) to handle new requests. Modern serverless platforms have reduced cold-start times to 2–3 seconds, but this remains a consideration for ultra-low-latency applications. Options A, C, and D describe different infrastructure challenges, not the cold-start phenomenon.

---

**Question 3: Multiple Choice**

What is the primary advantage of running AI inference on edge devices rather than centralized cloud data centers?

A) Edge devices always have more powerful GPUs than cloud servers  
B) Reduced latency because processing happens closer to the data source  
C) Edge inference eliminates the need for model training entirely  
D) Edge devices cost significantly less than cloud infrastructure  

**Correct Answer:** B

**Explanation:** Edge inference dramatically reduces latency because data is processed locally or near the user, eliminating the network round-trip to distant cloud data centers. This is crucial for real-time applications like autonomous vehicles or smart factory sensors. While edge can be cost-effective at scale, it requires upfront hardware investment, so option D is not universally true.

---

**Question 4: Multiple Select**

Which of the following are valid sources of latency in a distributed AI inference pipeline? (Select all that apply)

A) Network transmission time between nodes  
B) Model computation time on GPUs  
C) Data serialization and deserialization overhead  
D) The color of the network cables used  
E) Queue wait time when multiple requests are pending  

**Correct Answer:** A, B, C, E

**Explanation:** Latency comes from multiple sources: network hops (A), actual computation (B), data format conversions (C), and queuing delays (E). Option D is irrelevant—the physical cable color has no impact on latency. Understanding these components helps identify optimization opportunities.

---

**Question 5: Multiple Choice**

In the context of scalability, what does "throughput" measure?

A) The number of inference requests processed per unit of time  
B) The total latency experienced by a single user  
C) The physical distance between servers  
D) The amount of GPU memory available  

**Correct Answer:** A

**Explanation:** Throughput is the volume of work completed—specifically, how many inference requests a system can process per second (or per minute). It's distinct from latency (time per request) and capacity (total resources available). High throughput systems can handle many concurrent requests efficiently.

---

### Moderate Questions (Implementation & Application)

**Question 6: Multiple Choice**

You're designing an AI inference system for a real-time video recommendation platform. Users expect responses within 100ms, and you anticipate 10,000 concurrent requests per second. Which architecture would best balance your latency and throughput requirements?

A) A single centralized cloud data center with all models running on one GPU cluster  
B) A distributed edge inference network with local caching and centralized fallback for complex queries  
C) A serverless platform with on-demand GPU scaling and no pre-provisioned resources  
D) A local-only inference system running entirely on user devices  

**Correct Answer:** B

**Explanation:** This scenario requires a hybrid approach. Edge inference handles most requests with sub-10ms latency by processing data locally. Semantic caching at the edge further reduces redundant computations. For complex queries that edge models can't handle, requests fall back to centralized cloud resources. Option A creates a bottleneck; Option C's cold-starts violate the 100ms SLA; Option D sacrifices model quality and personalization.

---

**Question 7: Multiple Select**

Which techniques can reduce latency in distributed AI inference pipelines? (Select all that apply)

A) Using larger batch sizes to process more requests simultaneously  
B) Implementing tensor parallelism to shard model weights across multiple GPUs  
C) Deploying models at the edge closer to end users  
D) Increasing the number of model parameters for better accuracy  
E) Using semantic caching to avoid redundant vector database queries  

**Correct Answer:** B, C, E

**Explanation:** Tensor parallelism (B) reduces per-request latency by distributing computation across GPUs. Edge deployment (C) cuts network round-trips. Semantic caching (E) avoids expensive retrieval operations. Option A increases throughput but not latency per request. Option D actually increases latency by requiring more computation.

---

**Question 8: Multiple Choice**

A company is comparing two inference strategies for their LLM-powered chatbot:
- **Strategy 1:** Centralized cloud inference with 150ms latency but $0.001 per request
- **Strategy 2:** Edge inference with 30ms latency but $0.003 per request (including edge hardware amortization)

At 1 million requests per day, what is the primary trade-off being made?

A) Strategy 1 sacrifices user experience (latency) to save $2,000 per day in costs  
B) Strategy 2 improves user experience by 120ms but costs $2,000 more per day  
C) Strategy 1 is always superior because it costs less  
D) There is no meaningful difference between the two strategies  

**Correct Answer:** A

**Explanation:** This is a classic cost–latency trade-off. Strategy 1 saves $2,000 daily but users experience 120ms more latency, which may degrade perceived responsiveness. Strategy 2 prioritizes user experience (4x faster) at higher cost. The "right" choice depends on business priorities: real-time applications (e.g., gaming) favor Strategy 2, while batch-processing scenarios favor Strategy 1.

---

## Section 2: Subjective Question (Synthesis & Analysis)

**Question 9: Application & Synthesis**

**Scenario:** You're an infrastructure architect at a healthcare company building a diagnostic AI system. The system processes medical imaging (X-rays, MRIs) to assist radiologists with preliminary diagnoses. The system must:
- Process images with <500ms latency (to feel real-time to radiologists)
- Handle 500 concurrent users during peak hours
- Maintain strict data privacy (medical data cannot leave on-premises infrastructure)
- Operate cost-effectively without over-provisioning

**Your Task:** Design a distributed inference architecture that satisfies all four constraints. In your response:

1. **Specify your architectural approach** (e.g., centralized cloud, edge, hybrid, serverless, or a combination). Justify each choice.

2. **Identify latency sources** in your design and explain how each is addressed. Reference specific techniques (e.g., tensor parallelism, caching, batching, model quantization).

3. **Explain the cost–latency–throughput trade-offs** you're making. What are you optimizing for, and what are you accepting as trade-offs?

4. **Address the privacy constraint** explicitly. How does your architecture ensure medical data stays on-premises?

5. **Propose one optimization technique** (not yet mentioned in your design) that could further improve either latency or cost efficiency. Explain why this technique is suitable for healthcare imaging.

---

**Sample Editorial Solution:**

**Architectural Approach:**

A **hybrid on-premises + cloud-burst architecture** is optimal:
- **Primary:** On-premises GPU cluster running inference locally (satisfies privacy constraint and achieves <500ms latency)
- **Secondary:** Cloud burst capacity for peak load spillover (ensures 500 concurrent users without over-provisioning)

**Justification:** On-premises keeps sensitive medical data within your infrastructure, eliminating compliance concerns. Local processing achieves sub-500ms latency because data never leaves the facility. Cloud burst handles traffic spikes without maintaining expensive idle GPU capacity 24/7.

---

**Latency Source Decomposition:**

| Latency Source | Duration | Mitigation |
|---|---|---|
| Image upload | 50–100ms | Optimize network stack; use progressive JPG encoding |
| Model inference (GPU compute) | 200–300ms | Implement tensor parallelism across 4 GPUs; use INT8 quantization |
| Post-processing & formatting | 50ms | Minimize CPU operations; pre-allocate output buffers |
| Network queue wait | 50–100ms | Implement continuous batching (group 8 requests, process every 50ms) |
| **Total** | **~400–550ms** | Tuning required; edge case handling may exceed 500ms |

**Optimization Techniques Applied:**
- **Tensor Parallelism:** Shard the model across 4 GPUs so each GPU processes 1/4 of the model layers. This reduces per-request compute time from 300ms to ~100ms.
- **Quantization:** Convert model weights from FP32 to INT8 (4x memory reduction). This fits the full model in GPU VRAM, avoiding slow CPU-GPU data transfers.
- **Continuous Batching:** Instead of processing requests individually, group 8 concurrent requests and process them as a mini-batch. Slightly increases latency per request (~50ms) but increases throughput and GPU utilization from 40% to 85%.

---

**Cost–Latency–Throughput Trade-offs:**

| Metric | Trade-off | Decision |
|---|---|---|
| **Latency** | On-premises is slower than cloud-optimized inference but satisfies privacy | Accept: Privacy > Speed |
| **Throughput** | Continuous batching slightly increases latency but enables 500 concurrent users on smaller GPU cluster | Accept: Batching overhead is worth the throughput gain |
| **Cost** | On-premises requires upfront capital (GPUs, cooling, maintenance) vs. cloud's pay-per-use | Accept: Amortized cost is lower than cloud at this scale; privacy justifies capital investment |

---

**Privacy Implementation:**

- **Network Isolation:** On-premises cluster has no internet egress. Radiologists upload images via encrypted VPN tunnel.
- **Data Residency:** All images and intermediate model activations remain on-premises GPUs. No data is cached, logged, or transmitted to external systems.
- **Audit Trail:** Implement local logging for compliance (HIPAA, GDPR). Cloud burst (if used) processes anonymized data only.

---

**Proposed Additional Optimization: Semantic Caching**

**Technique:** Implement semantic caching at the application layer. Store embeddings of previously processed images. When a radiologist uploads a new image, compute its embedding and check if a semantically similar image was recently processed.

**Why It's Suitable:**
- **Medical Context:** Radiologists often review similar cases (e.g., multiple chest X-rays of the same patient, or similar pathologies). Semantic caching avoids redundant inference for near-duplicate images.
- **Latency Improvement:** Cache hits return results in <50ms instead of 400–500ms.
- **Privacy-Safe:** Embeddings can be stored locally without exposing raw image data.
- **Cost Benefit:** Reduces GPU compute by ~15–20% during typical clinic hours when similar cases cluster.

**Implementation:** Use a local vector database (e.g., Milvus) to store image embeddings. Set a similarity threshold (e.g., cosine similarity > 0.95). If a match is found, retrieve the cached diagnosis and present it with a "Similar case found" flag for radiologist review.

---

## Answer Key and Solutions

### Objective Questions Answer Summary

| Question | Type | Answer | Difficulty |
|---|---|---|---|
| 1 | MCQ | B | Easy |
| 2 | MCQ | B | Easy |
| 3 | MCQ | B | Easy |
| 4 | MSQ | A, B, C, E | Easy |
| 5 | MCQ | A | Easy |
| 6 | MCQ | B | Moderate |
| 7 | MSQ | B, C, E | Moderate |
| 8 | MCQ | A | Moderate |

---

### Detailed Explanations for Objective Questions

**Question 1 – Latency Definition**
- **Why B is correct:** Latency measures response time, fundamental to user experience.
- **Common misconception:** Students often confuse latency with bandwidth (data volume per time) or throughput (requests per second).

**Question 2 – Cold-Start Problem**
- **Why B is correct:** Cold-start is the delay when serverless functions provision resources after idle periods. Modern platforms have reduced this to 2–3 seconds, but it remains a consideration.
- **Why others are wrong:** A refers to memory initialization (separate issue); C refers to training (not related to inference); D refers to model loading (a component of cold-start but not the full phenomenon).

**Question 3 – Edge Inference Advantage**
- **Why B is correct:** Edge reduces latency by eliminating network round-trips to distant data centers. This is the primary advantage, especially for real-time applications.
- **Why others are wrong:** A is false (cloud often has more powerful hardware); C is false (inference still requires trained models); D is context-dependent (edge hardware has upfront costs).

**Question 4 – Latency Sources (Multiple Select)**
- **Why A, B, C, E are correct:**
  - A: Network transmission is a major latency source in distributed systems.
  - B: GPU computation is the primary latency source for model inference.
  - C: Serialization/deserialization (e.g., converting tensors to JSON) adds overhead.
  - E: Queue wait time is often overlooked but significant under high load.
- **Why D is wrong:** Physical cable color has zero impact on latency (trick option to test understanding).

**Question 5 – Throughput Definition**
- **Why A is correct:** Throughput measures the volume of work completed per unit time.
- **Common misconception:** Students often confuse throughput with latency (they're inversely related but distinct).

**Question 6 – Real-Time Video Recommendation Architecture**
- **Why B is correct:** Hybrid edge + cloud architecture balances latency (edge handles most requests in <10ms), throughput (caching reduces redundant queries), and cost (cloud fallback avoids over-provisioning).
- **Why others are wrong:**
  - A: Single GPU cluster creates a bottleneck; can't achieve 100ms SLA at 10k RPS.
  - C: Serverless cold-starts (even 2–3 seconds) violate the 100ms SLA.
  - D: On-device inference sacrifices model quality and personalization.

**Question 7 – Latency Reduction Techniques**
- **Why B, C, E are correct:**
  - B: Tensor parallelism distributes computation across GPUs, reducing per-request latency.
  - C: Edge deployment cuts network round-trips dramatically.
  - E: Semantic caching avoids expensive retrieval operations.
- **Why others are wrong:**
  - A: Larger batches increase throughput but increase latency per request (batching overhead).
  - D: More parameters increase computation time, worsening latency.

**Question 8 – Cost–Latency Trade-off Analysis**
- **Why A is correct:** Strategy 1 saves money ($2,000/day) but sacrifices latency (120ms slower). This is a classic trade-off where the business must decide if cost savings justify user experience degradation.
- **Why others are wrong:**
  - B: Reverses the framing (both strategies have trade-offs; B is factually correct but misses the economic analysis).
  - C: False; cost is not the only factor; latency impacts user satisfaction.
  - D: The strategies are significantly different in both cost and performance.

---

### Subjective Question Evaluation Rubric

**Question 9 – Healthcare Diagnostic AI Architecture**

**Scoring Criteria (out of 100 points):**

| Criterion | Points | Evaluation Notes |
|---|---|---|
| **Architectural Design** | 25 | Student clearly specifies hybrid on-premises + cloud-burst (or justified alternative). Explains why this satisfies privacy, latency, and throughput constraints. |
| **Latency Source Analysis** | 25 | Identifies 4+ latency sources (network, compute, serialization, queue). Quantifies each source (e.g., "GPU compute: 200–300ms"). Proposes specific mitigations (parallelism, quantization, batching). |
| **Trade-off Discussion** | 20 | Explicitly acknowledges cost–latency–throughput trade-offs. Explains why certain choices were made (e.g., "batching increases latency slightly but enables 500 concurrent users"). |
| **Privacy Implementation** | 15 | Clearly addresses data residency constraint. Explains how medical data stays on-premises (network isolation, local logging, no cloud transmission). |
| **Additional Optimization** | 10 | Proposes a novel technique (e.g., semantic caching, knowledge distillation, model pruning) with clear justification for healthcare context. |
| **Clarity & Communication** | 5 | Writing is clear, well-organized, and easy to follow. Uses technical terminology correctly. |

**Acceptable Alternative Architectures:**

While the editorial solution proposes on-premises + cloud-burst, other valid approaches include:
1. **Full On-Premises:** If the company has capital to invest, a fully on-premises solution avoids cloud costs and guarantees privacy.
2. **VPC-Isolated Cloud:** If cloud provider offers isolated VPC with data residency guarantees (e.g., AWS PrivateLink), this could work with appropriate compliance certifications.
3. **Federated Edge:** Deploy inference models to radiologist workstations (local GPUs). Aggregate results centrally for audit/compliance.

Each alternative must justify how it satisfies the four constraints. Students should be rewarded for thoughtful trade-off analysis, not penalized for choosing a different architecture.

---

## Key Concepts Summary

**Latency Decomposition:** Understanding that inference latency comes from multiple sources (network, compute, serialization, queue) enables targeted optimizations.

**Distributed Inference Paradigms:**
- **Centralized Cloud:** Low upfront cost; high latency; potential privacy concerns.
- **Edge Computing:** Sub-10ms latency; data privacy; higher upfront hardware cost.
- **Serverless:** Pay-per-use; cold-start delays; good for bursty workloads.
- **Hybrid:** Combines benefits; most complex to operate.

**Cost–Latency–Throughput Trade-offs:** No architecture is optimal for all metrics. Businesses must prioritize based on use case. Real-time applications (e.g., autonomous vehicles) prioritize latency. Batch processing prioritizes throughput. Cost-sensitive applications may accept higher latency.

**Optimization Techniques:**
- **Tensor Parallelism:** Reduce per-request latency by distributing model computation.
- **Model Quantization:** Reduce model size and memory bandwidth requirements.
- **Continuous Batching:** Improve throughput and GPU utilization at the cost of slight latency increase.
- **Semantic Caching:** Avoid redundant computations by caching similar inputs.
- **Edge Preprocessing:** Tokenization, embedding, and filtering at the edge reduce cloud compute load.