# Assignment 1 - Serverless Computing Critical Analysis

## Part 1: Paper Summary

The paper takes us through the advantages and disadvantages of serverless computing. With a lot of hype around this technology seven years ago in 2019, researchers at UC Berkeley do a deep dive and critical analysis of this subject to find out whether it actually holds up to its promises or is mostly just marketing.

The paper critiques serverless architecture, aptly titled "one step forward, two steps back", the paper addresses problems with serverless computing for many modern use-cases, by running tests like machine learning, prediction serving and distributed computing against more traditional means like virtual machines and other existing server technologies. On the other hand, tasks which consist of a multitude of independent actions which could be performed in parallel and one-off operations that hand off responsibility to other specialized systems displayed the true potential of serverless (the "one step forward").

Diving deeper into the deficiencies of this architecture, researchers found the following problems:
- Execution Time Constraints: Serverless functions are strictly time bound. Lambda, for example, forcibly kills a function after fifteen minutes of execution regardless of its current states.
- Communication and network limitations: Functions fall into the "data-shipping" trap, where the data must be pulled into where the code resides every time an instance is created. Even if the same dataset is created multiple times, it must be fetched over the network repeatedly because of the execution time constraints mentioned above.
- Limited Hardware Access: at the moment this paper was written, serverless functions were only allowed a part of the CPU and some RAM, there was no provision for tasks that could, for example, perform magnitudes better on specialized architectures (e.g. GPUs).
- Challenges for distributed computing: these functions are ephemeral and short-lived, with dynamic network addressing, which doesn't allow for them to communicate with each other through network and forces them to go through cloud storage, which is expensive and inefficient.

Revisiting the core argument: "serverless computing does not unlock the true potential of cloud computing", at least not in its current state. But it must also be addressed that explosive technologies like Google's MapReduce, that completely changed the state of computing, also started as something very limited in its capabilities, forcing developers to innovate, therefore the researchers have proposed the following improvements and future directions:

- Shipping code to data: one of the biggest limitations, as we have discovered, is network communication and latency. Instead of pulling data to where the code is, code needs to be fluid and allow for meeting on the node where the data resides.
- Specialized Hardware: while expensive, with the amount of workloads to make it profitable, cloud providers stand at a unique position to make such offerings possible.
- A universal language (common internal representation): regardless of whether the function is written in python or node, it should compile down to a standardized cloud-native language, that can efficiently execute workloads.

---
## Part 2: Azure Durable Functions Deep Dive

### Orchestration Model

Durable functions fixes the "isolated and ephemeral nature" problem of FaaS which makes it difficult to maintain state across client class without using external storage. Durable functions fixes this through orchestrator and entity functions who can effectively handle state management and checkpoints. The client function acts like the entrypoint to the function chain, it sits behind the orchestrator function, which acts as the brain of the whole workflow, which in turn controls activity functions who primarily do computation.

### State Management

By using the aforementioned durable task scheduler, you can eliminate the need for expensive and slow cloud storage, and use the "task scheduler's in-memory state store which is highly optimized for use with Durable Functions and the Durable Task SDKs, resulting in better durability and reliability and reduced latency" (Hhunter-Ms).

### Execution Timeouts

The standard timeout limits that apply to functions are bypassed. While the provided sources do no explicitly list the specific timeout duration for activity functions - with state and checkpoint management, the orchestrator can reliably handle recovery and retries.
Developers should assume standard function limitations still apply to individual activity functions and instead focus on making their functions adapt to the orchestrator. For truly long-lived operations, there are orchestrator and entity functions but they are more suited for monitoring.

### Communication between functions

Durable functions fix the "isolated and ephemeral nature" problem of FaaS which makes it difficult to maintain state across client class without using external storage. Azure's durable task scheduler internally has its own dedicated compute and memory resources optimized for:
- Dispatching orchestrator, activity, and entity work items
- Storing and querying history at scale with minimal latency
- Providing a rich monitoring experience through the Durable Task Scheduler dashboard. (Microsoft, 2026)

### Parallel execution (fan-out/fan-in)

The Berkeley paper critiques FaaS for only being suitable for "embarrassingly parallel" tasks where functions never communicate, without direct addressability the functions cannot coordinate together efficiently.
An orchestrator function can execute multiple activity functions (fan-out) and wait for their execution and aggregate their results (fan-in). This moves beyond what the paper calls "uncoordinated parallelism" to a system where tasks have clear data dependencies.

---
## Part 3: Critical Evaluation

While functions are a great innovation in themselves - with the architecture signalling to me that a lot of thought was put into the architecture, and that it was built by experts, the Berkeley paper signals that there are quite a number of drawbacks to this. As we explored each one of them and consequently explored a potential solution in azure's "Durable Functions", I see that there are two limitations that still remain unresolved.

The first and the obvious one (to me) is the "data shipping" antipattern. The way I see it, though orchestrator functions make is possible to manage state in a much more centralized and efficient way, the fundamental architectural flaw still remains the same. The fact that data is being shipped to where the code resides has not changed, and examples like the machine learning pipeline mentioned in the paper will still suffer from the same bottlenecks such as pulling the 90 gigabytes of data through the network. While EC2 offered impressive performance by "co-locating" the code and data, using caching and other memory optimizations offered by EBS, FaaS does not keep up with it, even with new innovations like the task scheduler. In brief, orchestrators provide a logical fix by providing functions that can manage state, but do not solve the physical separation of compute and storage.

The second problem that remains largely unsolved is what the paper calls as "heterogenous hardware support" - the ability to provision specialized hardware to support sophisticated operations that would do better on a different stack (like GPUs, compiling code into the most efficient form etc.). I believe this problem is exacerbated by the fact that there is not much open-source support available for functions (though it has changed significantly since the time this paper was written). Being forced to follow AWS and Azure for innovation means things will only get better when they are profiting from the transaction - which is not always in the best interest of other organizations or developers.

In my verdict, while azure durable functions tries to address various shortcomings of the first iteration of functions - it still falls fundamentally short of solving the key architectural concerns laid out by the paper. Choosing a language and framework is still as messy and limiting as before (as is with VMs as well, don't get me wrong) without a common cloud-native language it can be compiled down to, and data shipping remains almost impossible to solve until the security concerns around it are addressed.

Despite all this I remain optimistic, recent innovations in the cloud are to me a large encouragement about the future of computing (at least when it comes to the cloud itself), and I can foresee major investments being made to reduce the footprints of the software we use the most - if containers changed the game 10 years ago, surely there is something else coming soon which will address the drawbacks of serverless.

---
## References

- Hellerstein, J. M. (2019). Serverless computing: One step forward, Two steps back. https://www.cidrdb.org/cidr2019/papers/p119-hellerstein-cidr19.pdf
- Hhunter-Ms. (n.d.-a). _Durable functions overview: Stateful serverless workflows_. Durable Functions Overview: Stateful Serverless Workflows | Microsoft Learn. https://learn.microsoft.com/en-us/azure/durable-task/durable-functions/durable-functions-overview
- Hhunter-Ms. (n.d.). _Durable task scheduler - durable task_. Durable Task | Microsoft Learn. https://learn.microsoft.com/en-us/azure/durable-task/scheduler/durable-task-scheduler

---
## AI Use Disclosure

- Google's NotebookLM was used to convert the paper into a podcast for giving more context to the student.
- Due to lack of detail in documentation for certain assignment questions, NotebookLM was used to fill in the knowledge gaps.
- NotebookLM was also questioned to clarify higher concepts like "leader election" etc.
