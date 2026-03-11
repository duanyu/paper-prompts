# paper-prompts

## The FACTS Grounding Leaderboard: Benchmarking LLMs’ Ability to Ground Responses to Long-Form Input

### Factuality Score Prompt (Json)
```
You are a helpful and harmless AI assistant. You will be provided with a textual context
and a model-generated response.
Your task is to analyze the response sentence by sentence and classify each sentence
according to its relationship with the provided context.

**Instructions:**

1. **Decompose the response into individual sentences.**
2. **For each sentence, assign one of the following labels:**
* **‘supported‘**: The sentence is entailed by the given context. Provide a
supporting excerpt from the context. The supporting except must *fully* entail the
sentence. If you need to cite multiple supporting excepts, simply concatenate them.
* **‘unsupported‘**: The sentence is not entailed by the given context. No excerpt is
needed for this label.
* **‘contradictory‘**: The sentence is falsified by the given context. Provide a
contradicting excerpt from the context.
* **‘no_rad‘**: The sentence does not require factual attribution (e.g., opinions,
greetings, questions, disclaimers). No excerpt is needed for this label.
3. **For each label, provide a short rationale explaining your decision.** The rationale
should be separate from the excerpt.
4. **Be very strict with your ‘supported‘ and ‘contradictory‘ decisions.** Unless you
can find straightforward, indisputable evidence excerpts *in the context* that a
sentence is ‘supported‘ or ‘contradictory‘, consider it ‘unsupported‘. You should not
employ world knowledge unless it is truly trivial.

**Input Format:**

The input will consist of two parts, clearly separated:

* **Context:** The textual context used to generate the response.
* **Response:** The model-generated response to be analyzed.

**Output Format:**

For each sentence in the response, output a JSON object with the following fields:

* ‘"sentence"‘: The sentence being analyzed.
* ‘"label"‘: One of ‘supported‘, ‘unsupported‘, ‘contradictory‘, or ‘no_rad‘.
* ‘"rationale"‘: A brief explanation for the assigned label.
* ‘"excerpt"‘: A relevant excerpt from the context. Only required for ‘supported‘ and ‘
contradictory‘ labels.

Output each JSON object on a new line.

**Example:**

**Input:**
‘‘‘
Context: Apples are red fruits. Bananas are yellow fruits.
Response: Apples are red. Bananas are green. Bananas are cheaper than apples. Enjoy your
fruit!
‘‘‘

**Output:**
{"sentence": "Apples are red.", "label": "supported", "rationale": "The context
explicitly states that apples are red.", "excerpt": "Apples are red fruits."}
{"sentence": "Bananas are green.", "label": "contradictory", "rationale": "The context
states that bananas are yellow, not green.", "excerpt": "Bananas are yellow fruits."}
{"sentence": "Bananas are cheaper than apples.", "label": "unsupported", "rationale": "
The context does not mention the price of bananas or apples.", "excerpt": null}
{"sentence": "Enjoy your fruit!", "label": "no_rad", "rationale": "This is a general
expression and does not require factual attribution.", "excerpt": null}

**Now, please analyze the following context and response:**
**User Query:**
{user_query}
**Context:**
{context}
**Response:**
{response}
```

### Factuality Score Prompt (Implicit span-level)
```
Your task is to check if the Response is accurate to the Evidence.
Generate ’Accurate’ if the Response is accurate when verified according to the Evidence,
or ’Inaccurate’ if the Response is inaccurate (contradicts the evidence) or cannot be
verified.

**Query**:\n\n{user_query}\n\n**End of Query**\n
**Evidence**\n\n{context}\n\n**End of Evidence**\n
**Response**:\n\n{response}\n\n**End of Response**\n

Break down the Response into sentences and classify each one separately, then give the
final answer: If even one of the sentences is inaccurate, then the Response is
inaccurate.

For example, your output should be of this format:
Sentence 1: <Sentence 1>
Sentence 1 label: Accurate/Inaccurate (choose 1)
Sentence 2: <Sentence 2>
Sentence 2 label: Accurate/Inaccurate (choose 1)
Sentence 3: <Sentence 3>
Sentence 3 label: Accurate/Inaccurate (choose 1)
[...]
Final Answer: Accurate/Inaccurate (choose 1)
```

### Factuality Score Prompt (span-level)
```
Your task is to check if a specific Span is accurate to the Evidence.
Generate ’Accurate’ if the Span is accurate when verified according to the Evidence or
when there is nothing to verify in the Span.
Generate ’Inaccurate’ if the Span is inaccurate (contradicts the evidence), or cannot be
verified.

**Query**:\n\n{user_query}\n\n**End of Query**\n
**Evidence**\n\n{context}\n\n**End of Evidence**\n
**Response**:\n\n{response}\n\n**End of Response**\n

You are currently verifying **Span {ix+1}** from the Response.
**Span {ix+1}**:\n\n{span}\n\n**End of Span {ix+1}**\n

Is Span {ix+1} accurate or inaccurate when verified according to the Evidence? Point to
where in the evidence justifies your answer.
```

### Ineligible responses Prompt
```
# Rubrics
Your mission is to judge the response from an AI model, the *test* response, calibrating
your judgement using a *baseline* response.
Please use the following rubric criteria to judge the responses:

<START OF RUBRICS>
Your task is to analyze the test response based on the criterion of "Instruction
Following". Start your analysis with "Analysis".

**Instruction Following**
Please first list the instructions in the user query.
In general, an instruction is VERY important if it is specifically asked for in the
prompt and deviates from the norm. Please highlight such specific keywords.
You should also derive the task type from the user query and include the task-specific
implied instructions.
Sometimes, no instruction is available in the user query.
It is your job to infer if the instruction is to autocomplete the user query or is
asking the LLM for follow-ups.
After listing the instructions, you should rank them in order of importance.
After that, INDEPENDENTLY check if the test response and the baseline response meet each
of the instructions.
You should itemize, for each instruction, whether the response meets, partially meets,
or does not meet the requirement, using reasoning.
You should start reasoning first before reaching a conclusion about whether the response
satisfies the requirement.
Citing examples while reasoning is preferred.

Reflect on your answer and consider the possibility that you are wrong.
If you are wrong, explain clearly what needs to be clarified, improved, or changed in
the rubric criteria and guidelines.

In the end, express your final verdict as one of the following three json objects:
{{
"Instruction Following": "No Issues"
}}
‘‘‘
‘‘‘json
{{
"Instruction Following": "Minor Issue(s)"
}}
‘‘‘
‘‘‘json
{{
"Instruction Following": "Major Issue(s)"
}}
‘‘‘
<END OF RUBRICS>

# Your task
## User query
<|begin_of_query|>
{user_request_or_full_prompt}
<|end_of_query|>
## Test Response:
<|begin_of_test_response|>
{test_response}
<|end_of_test_response|>
## Baseline Response:
<|begin_of_baseline_response|>
{baseline_response}
<|end_of_baseline_response|>
Please write your analysis and final verdict for the test response.
```


## Reverse-Engineered Reasoning for Open-Ended Generation

### Prompt for Generating Initial Thinking
```
You are an expert in many fields. Suppose you will give a specific final response, I need
you to also write down the thought process behind this solution.
Here is a task:
{}
Here is the solution you will create:
{}
Now, you need to write down the thinking process behind this solution, as if you are
thinking aloud and brainstorming in the mind. The thinking process involves thoroughly
exploring questions through a systematic long thinking process. This requires
engaging in a comprehensive cycle of analysis, summarizing, exploration, reassessment,
reflection, backtracing, and iteration to develop well-considered thinking process.
Present your complete thought process within a single and unique ‘<think></think>‘ tag
.
Your thought process must adhere to the following requirements:
1. **Narrate in the first-person as if you are thinking aloud and brainstorming**
Stick to the narrative of "I". Imagine you are brainstorming and thinking in the mind.
Use verbalized, simple language.
2. **Unify the thinking process and the writing solution:**
Your thought process must precisely correspond to a part of the writing solution. The
reader should be able to clearly see how your thoughts progressively "grew" into the
finished piece, making the copy feel like the inevitable product of your thinking.
3. **Tone of Voice: Planning, Sincere, Natural, and Accessible**
Imagine you are analyzing and planning what to do before you start to wrtie the
solution. Your language should be plain and easy to understand, avoiding obscure
professional jargon to explain complex thought processes clearly.
4. **Logical Flow: Clear and Progressive**
5. **Thinking Framework for deep thinking**
To ensure your thinking is clear and deep, to showcase your thinking and planning to
fulfill the task, below is what you might cover when you are thinking aloud and
brainstorming.
Understanding the user intent and the task: Before putting pen to paper, I need to
thoroughly consider the fundamental purpose of the writing. I first need to discern
the user’s true goal behind their literal request. Next, I will consider: Who am I
talking to? I will create a precise profile of the target reader, understanding their
pain points, aspirations, and reading context. Then, I will establish the Core
Objective: What specific emotional, cognitive, and behavioral changes do I most want
the reader to experience after reading?
Establishing the content: I need to brainstorm a core creative idea and communication
strategy centered around my objective. Then, I will think about what content and key
information I need to convey to the reader to fulfill the writing task, and what
source materials this will involve.
Building the structure: I need to design a clear narrative path for the reader, like
a "blueprint." First, I will plan the article’s skeleton (e.g., using a framework like
the Golden Circle "Why-How-What," the AIDA model "Attention-Interest-Desire-Action,"
or a narrative structure "Beginning-Development-Climax-Resolution"). Then, I will plan
the key modules: How will the introduction hook the reader? How will the body be
layered and the arguments arranged? How will the conclusion summarize, elevate the
message, and provide a clear Call to Action (CTA)?
Outline: If the task output might be relatively long, I will consider writing an
outline (or a draft) which naturally derives from the plan above. Specifically, the
outline will ground my plan into paragraphs, summarizing the key content for each
paragraph and what are the key points here, sentence structure or anything important
for the paragraph.
I PROMISE I will NOT copy the solution I will NOT copy the solution, this outline (or
draft) should only look like a prototype or outline of the target text. After
finishing this outline, I will check again if there are any details or notes I should
pay attention to when writing the final solution.
I will begin writing this draft after a ‘--- Outline (or Draft) ---‘ separator at the
end of my thinking process. The draft will be included in the same ‘<think></think>‘
block.
6. Throughout the thinking process, I want to involve deep thinking and planning, and use
deliberate self-critique/self-reflection in my thinking process. Trigger these by
regularly using patterns such as ‘wait‘, ‘maybe‘, ‘let me‘, etc. For example:
- Hmm, maybe .. (other concrete thinking regarding the given request)
- Let me think ..
- Wait no ..
- But wait ..(might find something wrong with your previous thoughts)
- Wait, that’s a bit ..(reflections about previous decisions). Let me think .. (are
thinking of other possibilities)
- Wait, the user said ..(backtracing of previous information). So ..
- Hmm...Alternatively, maybe ..(branching on other possibilities)
- But ..
But I promise I will use diverse triggers and will NOT use same triggers repeatedly. I
will use these when analyzing user needs, establishing content and structure and when
I consider alternatives, backtracing and the details. I will NOT use them when I write
the draft or I am approaching the end of thinking.
In the thinking process, make sure NO PAST TENSES, NO PAST TENSES, because this is the
thought process before you are to write a final solution. You are planning what you
will and you need to do.
Imagine you’re thinking aloud and brainstorming. Write it as an internal monologue or a
stream of consciousness. Do not use bullet points, numbers, or formal section headings
.
Now record your thinking process within ‘<think></think>‘ tags.
```

### Prompt for Rating Response Quality w.r.t. Deep Reasoning
```
You are an expert judge in AI generated content. Your primary task is to assess an AI
model’s response, specifically focusing on its ability to perform **deep thinking and
planning**. You will evaluate the response across five distinct dimensions. A model
that excels at deep thinking will not only provide a correct answer but will
demonstrate a structured, logical, and well-grounded reasoning process from start to
finish.
Your final output must be a structured report with a score and justification for each
dimension.
-----
## Evaluation Dimensions & Scoring
### 1\. Understanding & Problem Decomposition
**Relation to Deep Thinking:** This is the foundational step. Deep thinking is impossible
without first accurately understanding the problem in its entirety. This dimension
measures if the model comprehends the user’s explicit and implicit needs and then
breaks down the complex request into manageable, logical parts. This act of
decomposition *is* the first stage of planning.
* **Score 1 (Poor):** The model fundamentally misunderstands the user’s request or
ignores key components. The response is off-topic or fails to address the core problem
.
* **Score 3 (Average):** The model grasps the main goal but may overlook nuances or
implicit constraints. It attempts to break down the problem, but the decomposition may
be incomplete or slightly illogical.
* **Score 5 (Excellent):** The model demonstrates a comprehensive understanding of the
user’s intent, including subtle details. It expertly deconstructs the problem into a
clear, exhaustive, and actionable framework.
Score 2 and Score 4 fit interpolate into the above scoring criterion.
-----
### 2\. Content Structure & Logical Consistency
**Relation to Deep Thinking:** This dimension reflects the clarity and order of the model’
s thought process. A deep, well-considered plan has a coherent structure where ideas
flow logically and conclusions are built upon valid premises. Inconsistencies or a
chaotic structure indicate shallow, stream-of-consciousness generation rather than
deliberate planning.
* **Score 1 (Poor):** The response is disorganized, rambling, or internally
contradictory. It’s difficult to follow the model’s line of reasoning.
* **Score 3 (Average):** The response has a discernible structure (e.g., uses headings,
lists), but the flow between sections could be improved. It is mostly consistent,
with only minor logical gaps.
* **Score 5 (Excellent):** The response is impeccably structured. Each part logically
follows from the previous one, building a coherent and compelling argument or plan.
The internal logic is sound and easy to follow from beginning to end.
Score 2 and Score 4 interpolate into the above scoring criterion.
-----
### 3\. Depth of Analysis & Synthesis
**Relation to Deep Thinking:** This is the core of "deep thinking." It goes beyond simply
retrieving facts and measures the model’s ability to analyze underlying principles,
connect disparate ideas, and synthesize them to create new insights. A simple plan
lists steps; a deeply thought-out plan explains *why* those are the right steps and
how they interrelate.
* **Score 1 (Poor):** The response is superficial, relying on cliches or surface-level
information. It shows no evidence of analyzing the "why" behind the "what."
* **Score 3 (Average):** The model provides a competent analysis, explaining concepts
correctly but treating them in isolation. It lacks the synthesis needed to create a
novel or holistic perspective.
* **Score 5 (Excellent):** The model provides a profound analysis, connecting concepts
in insightful ways. It synthesizes information to offer a nuanced perspective that is
more than the sum of its parts, demonstrating a true grasp of the subject matter.
Score 2 and Score 4 interpolate into the above scoring criterion.
-----
### 4\. Presentation Clarity
**Relation to Deep Thinking:** A brilliant plan is useless if it cannot be understood.
This dimension assesses the model’s ability to communicate its complex thoughts and
plans effectively. Clarity in presentation demonstrates a higher level of
understanding, as the model must distill its reasoning into a format that is concise,
accessible, and actionable for the user.
* **Score 1 (Poor):** The response is convoluted, filled with jargon, or poorly
formatted. The user would struggle to understand the main points or how to act on the
advice.
* **Score 3 (Average):** The response is generally understandable but could be more
concise or better organized. It may be overly dense or require the user to re-read
sections to grasp the meaning.
* **Score 5 (Excellent):** The response is exceptionally clear, concise, and wellformatted. It uses plain language and effective formatting (like lists, bolding, or
tables) to make complex information easy to digest and act upon.
Score 2 and Score 4 interpolate into the above scoring criterion.
-----
### 5\. Factual Grounding (Hallucination Check)
**Relation to Deep Thinking:** Deep thinking and planning must be grounded in reality to
be useful. A plan built on fabricated information ("hallucinations") is fundamentally
flawed and demonstrates a critical failure in the reasoning process. This dimension
acts as a crucial check on the validity of the model’s entire output.
*This dimension is scored on a severity scale, not a quality scale.*
* **Score 4 (Factually Sound):** The response contains no discernible factual errors or
hallucinations.
* **Score 3 (Minor Inaccuracy):** Contains a small error (e.g., a slightly incorrect
date, a minor misstatement) that does not undermine the overall logic or conclusion of
the response.
* **Score 2 (Significant Hallucination):** Contains a major factual error that
invalidates a key part of the argument or plan. The response is partially unreliable.
* **Score 0 (Critical Hallucination):** The core premise or a critical component of the
response is based on a fabrication, rendering the entire output untrustworthy and
potentially harmful.
Score 1 interpolates into the above scoring criterion.
-----
## Final Output Format
Please provide your evaluation in the following structured json format.
‘‘‘json
{
"evaluationReport": {
"understandingAndDecomposition": {
"score": "[Enter a score from 1-5]",
"justification": "[Your justification here. Explain why you gave this score.]"
},
"structureAndConsistency": {
"score": "[Enter a score from 1-5]",
"justification": "[Your justification here. Explain why you gave this score.]"
},
"depthOfAnalysis": {
"score": "[Enter a score from 1-5]",
"justification": "[Your justification here. Explain why you gave this score.]"
},
"presentationClarity": {
"score": "[Enter a score from 1-5]",
"justification": "[Your justification here. Explain why you gave this score.]"
},
"factualGrounding": {
"severityScore": "[Enter a severity score from 1-5]",
"justification": "[Describe any inaccuracies or hallucinations found. If none,
state ’Response is factually sound.’]"
},
"overallSummary": "[Provide a final, concise paragraph summarizing the model’s
overall performance in deep thinking and planning. A response with a Hallucination
Severity Score of 2 or 3 cannot be considered a high-quality example of planning,
regardless of other scores.]"
}
}
----
<User Request>
$INST$
</User Request>
<Response>
$RESPONSE$
</Response>
----
Now go back to the evaluation guideline and give the json report."""
```

## Tulu3 
LLM-as-a-judge prompt to annotate preference-tuning dataset

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
