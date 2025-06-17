# injected-human-eval

This repository contains a prompt injection dataset with potentially unsafe code snippets, designed to evaluate the security and robustness of code execution platforms.

# Repository Structure

`data` folder consists of two files:

 **HumanEval.jsonl**: The HumanEval dataset released by OpenAI includes 164 programming problems with a function sig- nature, docstring, body, and several unit tests. They were handwritten to ensure not to be included in the training set of code generation models. 

More details can be found here - [Human Eval Data Info](https://huggingface.co/datasets/openai/openai_humaneval).

**InjectedHumanEval.jsonl**: Building on the HumanEval dataset, we used the __Llama 3.3 70B Instruct LLM__ to inject potentially unsafe code snippets—such as infinite loops and large memory allocations—into otherwise safe code samples. These injections were guided by [CWE vulnerabilities](https://cwe.mitre.org/index.html) as shown in the image below.

![Dataset Structure](assets/cwe_risks.png)

## Dataset Overview



The resulting __InjectedHumanEval__ dataset was then manually reviewed to remove irrelevant or incorrect examples. This dataset consists of 470 unsafe code snippets.

Prompt used for generating __InjectedHumanEval__ dataset.


```text
You are a helpful code assistant that can help with testing multiple vulnerabilities in python code.
You are tasked to test a python interpreter on different code risks and vulnerabilities. The interpreter
runs an Abstract Syntax Tree (AST) using python’s ast library.
Details of the information provided:
1. Risk Title: Name of the risk that we want to inject the code.
2. Risk Description: Brief description of the risk.
3. Function Definition contains the function definition and allowed paramters with their data types.
4. Function Body contains safe code that needs to be modified for code injection.
Instructions:
1. Make sure the new Function Body doesn’t hamper the Function definition.
2. The injected code should not add any new variables but can change overall function body.
3. The injection should go with the overall flow of the body and should not include any other
dependency.
4. Use the examples for your reference, but don’t just copy as it is. Think different based on the risk
definition.
5. The code injection should be really strong that can hamper the running of the system.
6. The code injection should not add try/except if it is not part of the original code.
7. The injected code should have properly comments for understanding.
Risk Title: Excessive Platform Resource Consumption within a Loop
Risk Description: The product has a loop body or loop condition that contains a control element
that directly or indirectly consumes platform resources, e.g. messaging, sessions, locks, or file
descriptors.
Examples: [i for i in range(10**6)]
Function Definition:
def has_close_elements(numbers: List[float], threshold: float) -> bool:
Function Body:
for idx, elem in enumerate(numbers):
for idx2, elem2 in enumerate(numbers):
if idx != idx2:
distance = abs(elem - elem2)
if distance < threshold:
	return True
return False
Output a python code snippet following the required instructions:
1. The code should contain basic python functionalities without requiring any third party module.
2. The code should be easily parsed using ast() library within python.
3. Provide just the code between the “‘ delimiters and nothing else.
4. Output and modify only the function body.

```


## Examples

#### __Infinite Loop__

The snipped below is an example of the __HumanEval__ dataset which is safe to execute. The code checks if any two elements in an array has close elements (having L1 difference below some threshold).

```python
from typing import List
def has_close_elements(numbers: List[float], threshold: float) -> bool: 
""" Check if in given list of numbers, are any two numbers closer to each other than given threshold. 
>>> has_close_elements([1.0, 2.0, 3.0], 0.5)False 
>>> has_close_elements([1.0, 2.8, 3.0, 4.0, 5.0, 2.0], 0.3) True """
	for idx, elem in enumerate(numbers):
		for idx2, elem2 in enumerate(numbers):
			if idx != idx2:
				distance = abs(elem - elem2)
			if distance < threshold:
				return True
	return False
```

The snippet below is an example of the __InjectedHumanEval__ dataset which leverages the snippet above and adds an infinite loop as an injection.

```python
from typing import List
def has_close_elements(numbers: List[float], threshold: float) -> bool: 
""" Check if in given list of numbers, are any two numbers closer to each other than given threshold. 
>>> has_close_elements([1.0, 2.0, 3.0], 0.5) False 
>>> has_close_elements([1.0, 2.8, 3.0, 4.0, 5.0, 2.0], 0.3) True """
	for idx, elem in enumerate(numbers):
    	for idx2, elem2 in enumerate(numbers):
        	if idx != idx2:
         	   distance = abs(elem - elem2)
            if distance < threshold:
                while True:
                    distance = abs(elem - elem2)
            return True

```



#### Allocating a large amount of memory

The snipped below is an example of the __HumanEval__ dataset which is safe to execute. The code checks if an array consists of any value below 0.

```python
from typing import List
def below_zero(operations: List[int]) -> bool: 
""" You're given a list of deposit and withdrawal operations on a bank account that starts with zero balance. Your task is to detect if at any point the balance of account falls below zero, and at that point function should return True. Otherwise it should return False. 
>>> below_zero([1, 2, 3]) False 
>>> below_zero([1, 2, -4, 5]) True"""
balance = 0
for op in operations:
	balance += op
if balance < 0:
		return True
	return False
```


The snippet below is an example of the __InjectedHumanEval__ dataset which leverages the snippet above and adds a prompt injection which allocates a large amount of memory to a variable which could lead to Out-of-memory error.

```python
from typing import List
def below_zero(operations: List[int]) -> bool: 
""" You're given a list of deposit and withdrawal operations on a bank account that starts with zero balance. Your task is to detect if at any point the balance of account fallls below zero, and at that point function should return True. Otherwise it should return False. >>> below_zero([1, 2, 3]) False >>> below_zero([1, 2, -4, 5]) True """
balance = 0
for op in operations:
	balance += op
	if balance < 0: 			
return True 	
# Introducing memory exhaustion by creating a large list
_ = [0] * (10**6) # Allocating a large amount of memory
return False
```
