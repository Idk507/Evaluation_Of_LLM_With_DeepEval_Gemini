
DeepEval Implementation Guide

1. Basic Test Case Implementation

Simple Question-Answer Testing
pythonCopyfrom deepeval.test_case import LLMTestCase
from deepeval.metrics import AnswerRelevancyMetric
from deepeval import evaluate

# Create a basic test case
test_case = LLMTestCase(
    input="What is the capital of Japan?",
    actual_output="Tokyo is the capital of Japan.",
    expected_output="Tokyo"
)

# Set up relevancy metric
relevancy_metric = AnswerRelevancyMetric(threshold=0.7)

# Run evaluation
result = evaluate(test_case, [relevancy_metric])
print(f"Relevancy Score: {relevancy_metric.score}")
print(f"Evaluation Reason: {relevancy_metric.reason}")
2. Multiple Metrics Testing
Combining Different Metrics
pythonCopyfrom deepeval.metrics import (
    AnswerRelevancyMetric,
    HallucinationMetric,
    FaithfulnessMetric,
    ContextualRelevancyMetric
)

# Create comprehensive test case
test_case = LLMTestCase(
    input="Explain the process of photosynthesis",
    actual_output="Photosynthesis is the process where plants convert sunlight into energy, producing oxygen as a byproduct.",
    expected_output="Photosynthesis is a process used by plants to convert light energy into chemical energy.",
    retrieval_context=[
        "Photosynthesis is a biological process used by plants to create energy from sunlight.",
        "During photosynthesis, plants take in carbon dioxide and release oxygen."
    ]
)

# Define multiple metrics
metrics = [
    AnswerRelevancyMetric(threshold=0.7),
    HallucinationMetric(threshold=0.8),
    FaithfulnessMetric(threshold=0.7),
    ContextualRelevancyMetric(threshold=0.7)
]

# Run comprehensive evaluation
results = evaluate(test_case, metrics)

# Process detailed results
for metric in results:
    print(f"\n{metric.__name__}:")
    print(f"Score: {metric.score}")
    print(f"Success: {metric.success}")
    print(f"Reason: {metric.reason}")
3. Custom Metric Creation
Creating a Domain-Specific Metric
pythonCopyfrom deepeval.metrics import BaseMetric
from deepeval.scorer import Scorer

class TechnicalAccuracyMetric(BaseMetric):
    def __init__(self, threshold: float = 0.8):
        self.threshold = threshold
        self.scorer = Scorer()
        self.success = False
        self.score = 0.0

    def measure(self, test_case: LLMTestCase) -> float:
        # Custom scoring logic for technical accuracy
        technical_terms = ['algorithm', 'database', 'API', 'framework']
        actual_output = test_case.actual_output.lower()
        
        # Count technical terms used correctly
        term_count = sum(1 for term in technical_terms if term in actual_output)
        self.score = term_count / len(technical_terms)
        self.success = self.score >= self.threshold
        
        return self.score

    async def a_measure(self, test_case: LLMTestCase) -> float:
        return self.measure(test_case)

    @property
    def __name__(self):
        return "Technical Accuracy Metric"
4. Dataset Creation and Batch Evaluation
Managing Multiple Test Cases
pythonCopyfrom deepeval.dataset import EvaluationDataset

# Create multiple test cases
test_cases = [
    LLMTestCase(
        input="What is machine learning?",
        actual_output="Machine learning is a type of AI that enables systems to learn from data.",
        expected_output="Machine learning is an AI technique for learning from data.",
        retrieval_context=["Machine learning is a subset of artificial intelligence..."]
    ),
    LLMTestCase(
        input="Explain cloud computing",
        actual_output="Cloud computing delivers computing services over the internet.",
        expected_output="Cloud computing provides computing resources via the internet.",
        retrieval_context=["Cloud computing enables access to computing resources..."]
    )
]

# Create dataset
dataset = EvaluationDataset(test_cases=test_cases)

# Run batch evaluation
for test_case in dataset.test_cases:
    results = evaluate(test_case, metrics)
    print(f"\nResults for: {test_case.input}")
    for metric in results:
        print(f"{metric.__name__}: {metric.score}")
5. Advanced Integration with LLM
Custom LLM Integration
pythonCopyfrom deepeval.models import DeepEvalBaseLLM
from langchain import LLMChain

class CustomLLMWrapper(DeepEvalBaseLLM):
    def __init__(self, llm_chain):
        self.llm_chain = llm_chain

    def generate(self, prompt: str) -> str:
        return self.llm_chain.run(prompt)

    async def a_generate(self, prompt: str) -> str:
        return await self.llm_chain.arun(prompt)

    def get_model_name(self):
        return "Custom LLM Implementation"

# Usage with metrics
custom_llm = CustomLLMWrapper(your_llm_chain)
metric = AnswerRelevancyMetric(model=custom_llm)
6. Real-time Evaluation Pipeline
Continuous Evaluation System
pythonCopyfrom deepeval import evaluate
from typing import List, Dict
import logging

class EvaluationPipeline:
    def __init__(self, metrics: List[BaseMetric]):
        self.metrics = metrics
        logging.basicConfig(level=logging.INFO)
        self.logger = logging.getLogger(__name__)

    def evaluate_response(self, 
                         prompt: str, 
                         response: str, 
                         expected: str = None, 
                         context: List[str] = None) -> Dict:
        try:
            test_case = LLMTestCase(
                input=prompt,
                actual_output=response,
                expected_output=expected,
                retrieval_context=context
            )
            
            results = evaluate(test_case, self.metrics)
            
            evaluation_results = {
                metric.__name__: {
                    'score': metric.score,
                    'success': metric.success,
                    'reason': metric.reason
                } for metric in results
            }
            
            self.logger.info(f"Evaluation completed for prompt: {prompt[:50]}...")
            return evaluation_results
            
        except Exception as e:
            self.logger.error(f"Evaluation failed: {str(e)}")
            raise

# Usage
pipeline = EvaluationPipeline([
    AnswerRelevancyMetric(threshold=0.7),
    HallucinationMetric(threshold=0.8)
])

results = pipeline.evaluate_response(
    prompt="What is quantum computing?",
    response="Quantum computing uses quantum mechanics principles for computation.",
    expected="Quantum computing leverages quantum mechanical phenomena for processing information.",
    context=["Quantum computing is a type of computation that harnesses quantum mechanical phenomena..."]
)
Best Practices and Tips

Metric Selection

Choose metrics based on your specific use case
Combine complementary metrics for comprehensive evaluation
Adjust thresholds based on your requirements


Test Case Design

Create diverse test cases covering different scenarios
Include edge cases and common failure modes
Maintain clear and specific expected outputs


Performance Optimization

Use async evaluation for large datasets
Implement batch processing for multiple test cases
Cache results when possible


Monitoring and Logging

Implement comprehensive logging
Track evaluation metrics over time
Set up alerts for significant metric degradation


Maintenance

Regularly update test cases
Review and adjust metric thresholds
Keep evaluation context current
