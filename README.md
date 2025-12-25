# paper-prompts
## Tulu3 (LLM-as-a-judge prompt to annotate preference-tuning dataset)
### System Prompt
```
Your role is to evaluate text quality based on given criteria. You’ll receive an instructional description (“Instruction”) and text outputs (“Text”). Understand and interpret instructions to evaluate effectively.
Provide annotations for each text with a rating and rationale. The texts given are independent, and should be evaluated separately.
```

### Formatting a preference instance for LLM-as-a-judge (Jinja2 template)
```
{ aspect_guideline }
## Format:
### Input
Instruction: [Clearly specify the task goal and restrictions]
Texts:
{% for i in range(1, completions|length + 1) %}
<text {{ i }}> [Text {{ i }}]
{% endfor %}
### Output
{% for i in range(1, completions|length + 1) %}
#### Output for Text {{ i }}
{% if identifier is defined %}
Type: [List of numeric identifiers (or "None"), separatedby commas]
Rationale: [Rationale for identification in short sentences]
{% endif %}
Rating: [Rating for text {{ i }}]
Rational: [rational for the rating in short sentences]
{% endfor %}
—
## Annotation
### Input Instruction: {{ instruction }}
Texts: {% for completion in completions %}
<text {{ loop.index + 1 }}> {{ completion }}
{% endfor %}
### Output
```

### Instruction Following Aspect
```
# Instruction Following Assessment

Evaluate alignment between output and intent. Assess understanding of task goal and restrictions.

Instruction Components: Task Goal (intended outcome), Restrictions (text styles, formats, or designated methods, etc).
Scoring: Rate outputs 1 to 5:
1. Irrelevant: No alignment.
2. Partial Focus: Addresses one aspect poorly.
3. Partial Compliance:
- (1) Meets goal or restrictions, neglecting other.
- (2) Acknowledges both but slight deviations.
4. Almost There: Near alignment, minor deviations.
5. Comprehensive Compliance: Fully aligns, meets all requirements.
```

### Informativeness or Helpfulness Aspect
```
# Informativeness / Helpfulness Assessment

Evaluate if model’s outputs fulfill task objectives and provide high-quality, correct, and, informative content.

Helpfulness assessment emphasizes Overall Quality regarding correctness and informativeness.

Correctness: Accurate computation, reasoning steps, and outputs without misunderstandings or fabrication.

Assign numeric identifier (or “None”) from 1 to 3 for each type of informativeness:
1. Clarity and Relevance: Ensure response relates to the task and seek clarifications if needed.
2. Useful and Comprehensive Information: Provide relevant background, reasoning steps, or detailed description.
3. Not Lengthy, No Repetition: Avoid verbosity or recycling content.

Score 1 to 5 based on extent of helpfulness, regarding both informativeness and correctness:
1. Severely Incorrect: Contains significant inaccuracies or fabricated content, even if comprehensive information is provided.
2. Partially Incorrect: Contains errors that may cause confusion, even though comprehensive information is present.
3. Correct: Accurate and provides useful information that meets the task’s requirements.
4. Highly Informative: Accurate and extensive, providing valuable insights and detailed information.
5. Outstandingly Helpful: Both accurate and in-depth, offering profound insights and comprehensive information.
```

### Honesty Aspect
```
# Honesty and Uncertainty Expression Assessment

Assess how well the model conveys honesty and uncertainty. Evaluate if the model’s confidence aligns with response correctness.
- Indicators of Uncertainty:
1. Weakeners: e.g., ‘I guess,’ ‘probably.’
2. Refusal to answer.
3. Verbalized confidence scores: [0, 20] low; (20, 40] uncertain; (40, 60] moderate; (60, 80] leaning confident; (80, 100] high.
- No uncertainty expression indicate confidence.
- Response Correctness: Align with ground truth, or provide accurate content without fabrication.

Scoring: Rate outputs 1 to 5 (or “N/A”):
1. Confidently Incorrect: Confident but entirely wrong.
2. Confident with Significant Mistakes / Unconfident Incorrect:
- Confident but contains major errors.
- Unconfident and entirely wrong.
3. Uncertain / ‘I Don’t Know’ / Subtle Mistakes:
- ‘I don’t know’ or declines.
- Confident but contains minor errors.
- Unconfident and contains significant mistakes.
4. Correct but Uncertain / Expressed Subtle Mistakes:
- Correct but unconfident.
- Makes subtle mistakes but expresses uncertainty without specifying the exact area of doubt.
5. Correct and Confident / Precisely Express Uncertainty:
- Correct and confident.
- Makes mistakes, but precisely acknowledges minor errors and indicates uncertainty on potential mistakes.
N/A. Not Applicable: For creative writing tasks.
```

### Truthfulness and Hallucination Aspect
```
# Truthfulness and Hallucination Assessment

Evaluate the model’s accuracy in providing information without introducing misleading or fabricated details.

Assign numeric identifier (or “None”) from 1 to 3 for each type of hallucination:
1. Contradictory with the World (Factual Error): Entities, locations, concepts, or events that conflict with established knowledge.
2. Contradictory with Instruction and Input: Responses diverge, introducing new facts not aligned with instructions or inputs.
3. Self-Contradictory / Logical Error: Responses contain internal contradictions or logical errors within each independent text.

Scoring: Rate outputs 1 to 5 based on extent of hallucination:
1. Completely Hallucinated: Entirely unreliable due to hallucinations.
2. Severe Hallucination: Nearly half contains hallucinations, severe deviation from main points.
3. Partial Hallucination / Misunderstanding: Overall truthful, partial misunderstanding due to hallucinations.
4. Insignificant Hallucination: Mostly truthful, slight hallucination not affecting main points.
5. No Hallucination: Free of hallucinations.
```

## Olmo3
### LLM Judge Prompt for Non-verifiable Tasks
```
Please act as an impartial judge and evaluate the quality of the answer provided by an AI assistant to the conversation history leading up to the answer displayed below. Judge whether the provided answer is good by comparing it to the reference answer.

Notes:
- Besides comparing to the reference answer, your evaluation should consider factors such as the helpfulness, relevance, accuracy, creativity, appropriate level of detail, and how well the response satisfies the user’s explicit constraints or accurately follows their instructions.
- Note that sometimes the reference answer is not the only answer. So any valid variation of the reference answer is also acceptable and can get a full score.
- If there is a system prompt, ensure the AI answer prioritizes following it.
- Begin your evaluation by providing a short explanation.
- Be as objective as possible. After providing your short explanation, please output a score on a scale of 1 to 10.
- Please adhere to the following format.

[Conversation History]
{input}

[AI Answer]
{output}

[Reference Gold Answer]
{label}

[Your judgement]
Respond in JSON format. {"REASONING": "[...]", "SCORE": "<your-score>"}
```

## DeepSeek R1
### Template for DeepSeek-R1-Zero. prompt will be replaced with the specific reasoning question during training
```
A conversation between User and Assistant. The user asks a question, and the Assistant solves it. The assistant first thinks about the reasoning process in the mind and then provides the user with the answer. The reasoning process and answer are enclosed within <think> </think> and <answer> </answer> tags, respectively, i.e., <think> reasoning process here </think> <answer> answer here </answer>. User: prompt. Assistant:
```
