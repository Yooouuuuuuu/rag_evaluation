# RAG Evaluation Pipeline

A smart, cost-effective system for evaluating and improving RAG (Retrieval-Augmented Generation) performance through automated testing and strategic human review.

## 🎯 What This Does

This pipeline helps you **systematically evaluate your RAG system** by:

- **Automatically classifying** query difficulty to focus human effort where it matters
- **Catching failures fast and cheap** with a 3-step evaluation process  
- **Diagnosing root causes** - is it a search problem or generation problem?
- **Scaling efficiently** from hundreds to millions of queries

## 🏗️ System Architecture 

```
New Queries → Phase 1 (Sort & Label) → Phase 2 (Live Monitor) → Phase 3 (Multi-turn)
                      ↓                        ↓                      ↓
                Human Review              Cost-Optimized          Conversation
                Excel Interface           Real-time Eval           Tracking
```

**The Big Idea**: Most queries are either obviously good or obviously bad. Only spend expensive human/LLM time on the tricky middle cases.

## 📊 Dataset Structure

Your data lives in a **Master XLSX file** with a smart column layout:

### Core Columns (A-U) - Always There
- **Query Info**: ID, text, conversation context
- **Labels**: Status (clear/ambiguous/challenge), challenge types  
- **Answers**: Gold standard answers and retrieved chunks
- **Scores**: Automated quality metrics
- **Review**: Human reviewer notes and timestamps

### Extension Columns (V+) - Added When Needed
- **Phase 1**: Full chunk text for human review (temporary)
- **Phase 2+**: Additional evaluation scores (future)

**Smart Design**: We only add the heavy columns (full text) when humans need to see them, then remove them to keep files manageable.

## 🔍 Phase 1: Pre-Evaluation (Smart Sorting)

**Goal**: Automatically sort queries so humans focus on the right things.

### The Process

1. **Run your query** through the RAG system
2. **Auto-evaluate** with three tools:
   - LLM-as-Judge (accuracy/relevance)
   - Embedding similarity (retrieval quality) 
   - Semantic consistency (hallucination detection)
3. **Auto-classify** into three buckets:
   - 🟢 **Clear** (60%) - high confidence, minimal review
   - 🟡 **Challenge** (15%) - known hard cases, needs special handling
   - 🔴 **Ambiguous** (25%) - unclear, needs human judgment

### Human Review Interface

- **Excel-based** - familiar interface for reviewers
- **Full context visible** - see exactly what chunks were retrieved
- **Smart workflows** - different actions for each query type
- **Quality tracking** - reviewer notes and timestamps

## ⚡ Phase 2: Real-Time Evaluation (Cost-Optimized)

**Goal**: Monitor live system performance without breaking the budget.

### 3-Step Smart Filter

#### Step 1: Quick Check (0.1s, $0.0001)
- **What**: Compare answers with embeddings
- **Catches**: Obviously wrong answers  
- **Runs**: Every query (100%)

#### Step 2: Double Check (3s, $0.004)
- **What**: LLM compares the two answers
- **Catches**: False positives from Step 1
- **Runs**: When Step 1 passes (~75%)

#### Step 3: Context Analysis (3s, $0.1) 
- **What**: Deep dive into retrieved chunks
- **Catches**: Root cause - search vs generation issue
- **Runs**: When either step fails (~20%)

### Smart Economics

- **Average cost**: ~$0.03 per query (vs $0.10+ for naive approach)
- **80% cost reduction** through smart filtering
- **Detailed diagnosis** only when needed

### What You Learn

| Pattern | Frequency | What It Means | How to Fix |
|---------|-----------|---------------|------------|
| Pass both steps | 60% | System working | Keep monitoring |
| Pass 1, fail 2 | 15% | Sounded right but wrong | Tune similarity threshold |
| Fail 1, bad chunks | 20% | Search problem | Fix embeddings/chunking |
| Fail 1, good chunks | 5% | Generation problem | Fix prompts/model |

## 🚀 Phase 3: Multi-Turn RAG (Coming Soon)

Future expansion for conversational RAG systems:
- Conversation flow tracking
- Memory strategy evaluation  
- Context window management
- Turn-level performance metrics

## 🛠️ Quick Start

### 1. Setup
```bash
pip install pandas openpyxl sentence-transformers openai anthropic
export OPENAI_API_KEY="your-key"
export SIMILARITY_THRESHOLD="0.7"
```

### 2. Phase 1 - Sort Your Queries
```python
from src.phase1 import Phase1Pipeline

# Run automated evaluation
p1 = Phase1Pipeline()
results = p1.run_evaluation("your_queries.xlsx")

# Export for human review  
p1.export_for_review("review.xlsx", include_chunks=True)

# Import human updates
p1.import_from_review("review_completed.xlsx")
```

### 3. Phase 2 - Monitor Live System
```python
from src.phase2 import Phase2Pipeline

# Setup real-time evaluator
p2 = Phase2Pipeline("your_labeled_dataset.xlsx")

# Evaluate single query
result = p2.evaluate_query(
    query_id="q001", 
    rag_response="Your system's answer here"
)

print(f"Pass: {result['passed']}, Cost: ${result['cost']:.4f}")
```

## 📈 What You Get

### Immediate Benefits
- **Faster human review** - focus on the 25% that actually need attention
- **Cost control** - spend evaluation budget where it matters
- **Root cause analysis** - know whether to fix search or generation
- **Quality tracking** - measure improvement over time

### Long-term Value
- **Systematic improvement** - data-driven optimization
- **Scalable process** - works from prototype to production
- **Knowledge building** - understand your system's failure modes
- **Team efficiency** - clear roles for humans vs automation

## 🤔 When to Use This

### Perfect For:
- RAG systems with 100+ queries to evaluate
- Teams that need systematic quality improvement
- Projects where evaluation cost matters
- Systems with mix of easy and hard queries

### Maybe Overkill For:
- Quick prototypes with <50 test queries
- Single-domain systems with very consistent performance
- Teams with unlimited evaluation budget

## 📚 File Organization

```
your-project/
├── data/
│   ├── master_dataset.xlsx     # Your main data file
│   ├── exports/                # JSON/CSV for tools
│   └── backups/               # Automatic backups
├── config/
│   ├── phase1_config.yaml     # Classification thresholds
│   └── phase2_config.yaml     # Cost and performance settings
└── reports/                   # Generated analysis
```

## 🔧 Configuration

Key settings you can tune:

```yaml
# phase1_config.yaml
classification:
  clear_threshold: 0.8      # How confident to auto-approve
  challenge_threshold: 0.4   # How low to auto-challenge
  
# phase2_config.yaml  
evaluation:
  similarity_threshold: 0.7  # Step 1 pass/fail line
  max_daily_cost: 50.0      # Budget protection
  enable_step3: true        # Deep analysis on/off
```

## 🆘 Need Help?

- **Getting started**: Check the detailed docs in your original files
- **Configuration issues**: Look at example configs in `config/`
- **Performance tuning**: See the monitoring section for key metrics
- **Custom challenge types**: Extend the rule-based classifier

---

**Remember**: This system is designed to make evaluation smarter, not harder. Start simple, then add complexity as you learn what works for your specific use case.