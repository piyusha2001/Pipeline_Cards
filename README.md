# **Architectural Overview: A Five-Layer System**

Our script is structured as a modular system with five distinct layers:

1. **System Initialization Layer (`setup_environment`)**  
   Prepares all necessary assets and configurations.

2. **Problem Specification Layer (Task Definitions)**  
   Defines the "what"—the data engineering tasks to be performed and the objective ground truth for each.

3. **LLM Execution Layer (`run_llm_pipeline`)**  
   Simulates the core data engineering work by passing the task to the LLM. This layer contains the crucial logic for our experimental intervention.

4. **Verification & Evaluation Layer (`process_single_run`)**  
   Critically analyzes the LLM's output against the ground truth, applying a multi-faceted definition of "correctness."

5. **Reporting & Insights Layer (`generate_summary_card`)**  
   Aggregates the results into the "Pipeline Card," translating raw metrics into actionable insights that directly map to our paper's four pillars.

# Here’s a deeper look at each layer and its significance:

## **Layer 1: System Initialization (`setup_environment`)**

- **Purpose:**  
  This layer is responsible for creating a consistent and reproducible runtime environment. It handles loading the LLM and tokenizer from Hugging Face, loading the UCI Adult dataset into a pandas DataFrame, and performing necessary data cleaning.

- **Architectural Significance:**  
  This layer establishes the stable foundation for the entire experiment. By isolating setup, we ensure that every benchmark run starts from the exact same state, which is critical for the scientific validity of our results.

---

## **Layer 2: Problem Specification (The TASK and `gt_calculator` definitions)**

- **Purpose:**  
  This layer defines the actual data engineering problems we are testing. It consists of two key components for each task:  
  - **The Prompt (`TASK_*_PROMPT`):** The natural language instruction that is given to the LLM.  
  - **The Ground Truth Calculator (`gt_calculator_*`):** A deterministic Python function that calculates the provably correct answer using standard libraries (pandas, numpy).

- **Architectural Significance:**  
  This demonstrates a core principle of our framework: the decoupling of the problem from the solver. The ground truth calculator acts as our "oracle," representing the objective, verifiable reality. The entire purpose of the system is to measure how well the probabilistic output of the LLM (Layer 3) can align with the deterministic truth defined here.

---

## **Layer 3: LLM Execution (`run_llm_pipeline`)**

- **Purpose:**  
  This is the heart of the system where the LLM performs the data engineering task. It takes a prompt and the dataset, constructs the full context for the model, sends it to the LLM for generation, and then parses the raw text response to extract a JSON object.

- **Architectural Significance:**  
  This layer is where our experimental intervention happens, controlled by the `use_remediation` boolean. This is a critical design choice:  
  - **Baseline Mode (`use_remediation=False`):** The "naive" or direct approach. We give the LLM the prompt along with a small sample of the raw data. This simulates a common but flawed implementation.  
  - **Remediated Mode (`use_remediation=True`):** The "informed" strategy, developed directly from the insights our framework provides.  
    - We stop sending raw data and instead send a structural description of the data (schema, column values, and total row count).  
    - We also use a more detailed system prompt and lower the temperature for more deterministic outputs.  
    - This remediation tests the hypothesis that understanding the data's structure is more important than seeing a few raw examples, which can be distracting.

---

## **Layer 4: Verification & Evaluation (`process_single_run`)**

- **Purpose:**  
  This layer acts as the "quality assurance" stage. It takes the parsed JSON from the LLM Execution Layer and meticulously compares it against the ground truth from the Problem Specification Layer.

- **Architectural Significance:**  
  This layer operationalizes a nuanced definition of "correctness," which is central to our paper. It verifies:  
  - **Structural Correctness:** Is the output valid JSON? Does it contain the correct keys (with case-insensitive matching for robustness)?  
  - **Numerical Correctness:** Are the numeric values within an acceptable tolerance? This acknowledges that LLMs may have minor precision variations.  
  - **Debuggability:** The function generates a detailed debug log, which is vital for understanding why a run failed and directly informs the remediation strategy.

---

## **Layer 5: Reporting & Insights (`generate_summary_card`)**

- **Purpose:**  
  This is the final output layer. It aggregates the results from multiple iterations and formats them into the human-readable "Pipeline Card."

- **Architectural Significance:**  
  This layer directly connects our experimental results back to the four pillars of our proposed framework:  
  - **Pillar 1 (Efficiency):** Reports Latency and Est. Cost.  
  - **Pillar 2 (Reliability):** Reports the Structured Output Rate.  
  - **Pillar 3 (Adaptivity):** Reports Task Correctness Rate and Mean Absolute Percentage Error (MAPE), quantifying "how wrong" the answer was.  
  - **Pillar 4 (Governance):** Reports the Fairness Metric.  

  The Pipeline Card is the ultimate output of the system, designed to give a holistic, multi-dimensional view of the LLM's performance and expose the trade-offs between different pillars (e.g., how the remediated approach improves reliability but may increase latency).

---

# **The Overall Process: An Experimental Loop**

The `if __name__ == "__main__":` block executes the core experimental loop that validates our paper's thesis:

1. **Run the Baseline Benchmark:**  
   Uses the naive prompting strategy to generate our initial set of Pipeline Cards, which act as a diagnostic tool, revealing the LLM's weaknesses.

2. **Run the Remediated Benchmark:**  
   Based on the failures diagnosed in the baseline, we deploy our targeted intervention (schema-based context and stricter prompting).

3. **Compare the Two Sets of Pipeline Cards:**  
   This comparison provides empirical evidence that our framework successfully identified a problem and that our informed, systems-level remediation strategy fixed it.

---
This architecture proves that by treating LLM-powered data engineering as a unified system to be evaluated and improved, rather than a series of one-off prompts, we can build far more reliable, robust, and governable pipelines.


