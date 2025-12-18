# paper-prompts

## DeepSeek R1
### Template for DeepSeek-R1-Zero. prompt will be replaced with the specific reasoning question during training
```
A conversation between User and Assistant. The user asks a question, and the Assistant solves it. The assistant first thinks about the reasoning process in the mind and then provides the user with the answer. The reasoning process and answer are enclosed within <think> </think> and <answer> </answer> tags, respectively, i.e., <think> reasoning process here </think> <answer> answer here </answer>. User: prompt. Assistant:
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
