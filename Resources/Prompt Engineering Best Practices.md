---
tags:
  - domain/ai
  - type/resource
  - language/ukrainian
source: "22365_3_Prompt Engineering_v7 (1).pdf"
date_added: "2025-10-02"
---

# Best Practices

Finding the right prompt requires tinkering. Language Studio in Vertex AI is a perfect place to

play around with your prompts, with the ability to test against the various models.

Use the following best practices to become a pro in prompt engineering.

## Provide examples

The most important best practice is to provide (one shot / few shot) examples within a

prompt. This is highly effective because it acts as a powerful teaching tool. These examples

showcase desired outputs or similar responses, allowing the model to learn from them

and tailor its own generation accordingly. It’s like giving the model a reference point or

target to aim for, improving the accuracy, style, and tone of its response to better match

your expectations.

February 2025 54Prompt Engineering

## Design with simplicity

Prompts should be concise, clear, and easy to understand for both you and the model. As a

rule of thumb, if it’s already confusing for you it will likely be also confusing for the model. Try

not to use complex language and don’t provide unnecessary information.

Examples:

BEFORE:

> I am visiting New York right now, and I'd like to hear more about great
> 
> locations. I am with two 3 year old kids. Where should we go during
> 
> our vacation?

AFTER REWRITE:

> Act as a travel guide for tourists. Describe great places to visit in
> 
> New York Manhattan with a 3 year old.

Try using verbs that describe the action. Here’s a set of examples:

Act, Analyze, Categorize, Classify, Contrast, Compare, Create, Describe, Define,

Evaluate, Extract, Find, Generate, Identify, List, Measure, Organize, Parse, Pick,

Predict, Provide, Rank, Recommend, Return, Retrieve, Rewrite, Select, Show, Sort,

Summarize, Translate, Write.

February 2025 55Prompt Engineering

## Be specific about the output

Be specific about the desired output. A concise instruction might not guide the LLM enough

or could be too generic. Providing specific details in the prompt (through system or context

prompting) can help the model to focus on what’s relevant, improving the overall accuracy.

Examples:

DO:

> Generate a 3 paragraph blog post about the top 5 video game consoles.`
> 
> `The blog post should be informative and engaging, and it should be`
> 
> `written in a conversational style.

DO NOT:

Generate a blog post about video game consoles.

## Use Instructions over Constraints

Instructions and constraints are used in prompting to guide the output of a LLM.

•

•

An instruction provides explicit instructions on the desired format, style, or content of the

response. It guides the model on what the model should do or produce.

A constraint is a set of limitations or boundaries on the response. It limits what the model

should not do or avoid.

February 2025 56Prompt Engineering

Growing research suggests that focusing on positive instructions in prompting can be more

effective than relying heavily on constraints. This approach aligns with how humans prefer

positive instructions over lists of what not to do.

Instructions directly communicate the desired outcome, whereas constraints might leave the

model guessing about what is allowed. It gives flexibility and encourages creativity within the

defined boundaries, while constraints can limit the model’s potential. Also a list of constraints

can clash with each other.

Constraints are still valuable but in certain situations. To prevent the model from generating

harmful or biased content or when a strict output format or style is needed.

If possible, use positive instructions: instead of telling the model what not to do, tell it what to

do instead. This can avoid confusion and improve the accuracy of the output.

DO:

Generate a 1 paragraph blog post about the top 5 video game consoles.

Only discuss the console, the company who made it, the year, and total

sales.

DO NOT:

Generate a 1 paragraph blog post about the top 5 video game consoles.

Do not list video game names.

As a best practice, start by prioritizing instructions, clearly stating what you want the model

to do and only use constraints when necessary for safety, clarity or specific requirements.

Experiment and iterate to test different combinations of instructions and constraints to find

what works best for your specific tasks, and document these.

February 2025 57Prompt Engineering

## Control the max token length

To control the length of a generated LLM response, you can either set a max token limit in the

configuration or explicitly request a specific length in your prompt. For example:

"Explain quantum physics in a tweet length message."

## Use variables in prompts

To reuse prompts and make it more dynamic use variables in the prompt, which can be

changed for different inputs. E.g. as shown in Table 20, a prompt which gives facts about a

city. Instead of hardcoding the city name in the prompt, use a variable. Variables can save

you time and effort by allowing you to avoid repeating yourself. If you need to use the same

piece of information in multiple prompts, you can store it in a variable and then reference

that variable in each prompt. This makes a lot of sense when integrating prompts into your

own applications.

guide. Tell me a fact about the city: {city}

Amsterdam is a beautiful city full of canals, bridges, and narrow streets. It’s a

great place to visit for its rich history, culture, and nightlife.

Table 20. Using variables in prompts

Prompt VARIABLES

{city} = "Amsterdam"

PROMPT

You are a travel Output February 2025 58Prompt Engineering

## Experiment with input formats and writing styles

Different models, model configurations, prompt formats, word choices, and submits can yield

different results. Therefore, it’s important to experiment with prompt attributes like the style,

the word choice, and the type prompt (zero shot, few shot, system prompt).

For example a prompt with the goal to generate text about the revolutionary video game

console Sega Dreamcast, can be formulated as a question, a statement or an instruction,

resulting in different outputs:

•

•

•

Question: What was the Sega Dreamcast and why was it such a revolutionary console?

Statement: The Sega Dreamcast was a sixth-generation video game console released by

Sega in 1999. It...

Instruction: Write a single paragraph that describes the Sega Dreamcast console and

explains why it was so revolutionary.

For few-shot prompting with classification tasks, mix up

the classes

Generally speaking, the order of your few-shots examples should not matter much. However,

when doing classification tasks, make sure you mix up the possible response classes in the

few shot examples. This is because you might otherwise be overfitting to the specific order of

the examples. By mixing up the possible response classes, you can ensure that the model is

learning to identify the key features of each class, rather than simply memorizing the order of

the examples. This will lead to more robust and generalizable performance on unseen data.

February 2025 59Prompt Engineering

A good rule of thumb is to start with 6 few shot examples and start testing the accuracy

from there.

Adapt to model updates

It’s important for you to stay on top of model architecture changes, added data, and

capabilities. Try out newer model versions and adjust your prompts to better leverage new

model features. Tools like Vertex AI Studio are great to store, test, and document the various

versions of your prompt.

## Experiment with output formats

Besides the prompt input format, consider experimenting with the output format. For non-

creative tasks like extracting, selecting, parsing, ordering, ranking, or categorizing data try

having your output returned in a structured format like JSON or XML.

There are some benefits in returning JSON objects from a prompt that extracts data. In

a real-world application I don’t need to manually create this JSON format, I can already

return the data in a sorted order (very handy when working with datetime objects), but most

importantly, by prompting for a JSON format it forces the model to create a structure and

limit hallucinations.

In summary, benefits of using JSON for your output:

•

•

Returns always in the same style

Focus on the data you want to receive

February 2025 60Prompt Engineering

•

•

•

•

Less chance for hallucinations

Make it relationship aware

You get data types

You can sort it

Table 4 in the few-shot prompting section shows an example on how to return

structured output.

### JSON Repair

While returning data in JSON format offers numerous advantages, it's not without its

drawbacks. The structured nature of JSON, while beneficial for parsing and use in

applications, requires significantly more tokens than plain text, leading to increased

processing time and higher costs. Furthermore, JSON's verbosity can easily consume the

entire output window, becoming especially problematic when the generation is abruptly cut

off due to token limits. This truncation often results in invalid JSON, missing crucial closing

braces or brackets, rendering the output unusable. Fortunately, tools like the json-repair

library (available on PyPI) can be invaluable in these situations. This library intelligently

attempts to automatically fix incomplete or malformed JSON objects, making it a crucial

ally when working with LLM-generated JSON, especially when dealing with potential

truncation issues.

February 2025 61Prompt Engineering

## Working with Schemas

Using structured JSON as an output is a great solution, as we've seen multiple times in this

paper. But what about input? While JSON is excellent for structuring the output the LLM

generates, it can also be incredibly useful for structuring the input you provide. This is where

JSON Schemas come into play. A JSON Schema defines the expected structure and data

types of your JSON input. By providing a schema, you give the LLM a clear blueprint of the

data it should expect, helping it focus its attention on the relevant information and reducing

the risk of misinterpreting the input. Furthermore, schemas can help establish relationships

between different pieces of data and even make the LLM "time-aware" by including date or

timestamp fields with specific formats.

Here's a simple example:

Let's say you want to use an LLM to generate descriptions for products in an e-commerce

catalog. Instead of just providing a free-form text description of the product, you can use a

JSON schema to define the product's attributes:

```js
{`

`"type": "object",`

`"properties": {`

`"name": { "type": "string", "description": "Product name" },`

`"category": { "type": "string", "description": "Product category" },`

`"price": { "type": "number", "format": "float", "description": "Product`

`price" },`

`"features": {`

`"type": "array",`

`"items": { "type": "string" },`

`"description": "Key features of the product"`

`},`

`"release_date": { "type": "string", "format": "date", "description":`

`"Date the product was released"}`

`},
```

Snippet 5. Definition of the structured output schema

February 2025 62Prompt Engineering

Then, you can provide the actual product data as a JSON object that conforms to

this schema:

{

"name": "Wireless Headphones",

"category": "Electronics",

"price": 99.99,

"features": ["Noise cancellation", "Bluetooth 5.0", "20-hour battery life"],

"release_date": "2023-10-27"

}

Snippet 6. Structured output from the LLM

By preprocessing your data and instead of providing full documents only providing both the

schema and the data, you give the LLM a clear understanding of the product's attributes,

including its release date, making it much more likely to generate an accurate and relevant

description. This structured input approach, guiding the LLM's attention to the relevant fields,

is especially valuable when working with large volumes of data or when integrating LLMs into

complex applications.

Experiment together with other prompt engineers

If you are in a situation where you have to try to come up with a good prompt, you might

want to find multiple people to make an attempt. When everyone follows the best practices

(as listed in this chapter) you are going to see a variance in performance between all the

different prompt attempts.

February 2025 63Prompt Engineering

## CoT Best practices

For CoT prompting, putting the answer after the reasoning is required because the

generation of the reasoning changes the tokens that the model gets when it predicts the

final answer.

With CoT and self-consistency you need to be able to extract the final answer from your

prompt, separated from the reasoning.

For CoT prompting, set the temperature to 0.

Chain of thought prompting is based on greedy decoding, predicting the next word in a

sequence based on the highest probability assigned by the language model. Generally

speaking, when using reasoning, to come up with the final answer, there’s likely one single

correct answer. Therefore the temperature should always set to 0.

## Document the various prompt attempts

The last tip was mentioned before in this chapter, but we can’t stress enough how important

it is: document your prompt attempts in full detail so you can learn over time what went well

and what did not.

Prompt outputs can differ across models, across sampling settings, and even across different

versions of the same model. Moreover, even across identical prompts to the same model,

small differences in output sentence formatting and word choice can occur. (For example, as

mentioned previously, if two tokens have the same predicted probability, ties may be broken

randomly. This can then impact subsequent predicted tokens.).

February 2025 64Prompt Engineering

We recommend creating a Google Sheet with Table 21 as a template. The advantages of

this approach are that you have a complete record when you inevitably have to revisit your

prompting work–either to pick it up in the future (you’d be surprised how much you can

forget after just a short break), to test prompt performance on different versions of a model,

and to help debug future errors.

Beyond the fields in this table, it’s also helpful to track the version of the prompt (iteration),

a field to capture if the result was OK/NOT OK/SOMETIMES OK, and a field to capture

feedback. If you’re lucky enough to be using Vertex AI Studio, save your prompts (using the

same name and version as listed in your documentation) and track the hyperlink to the saved

prompt in the table. This way, you’re always one click away from re-running your prompts.

When working on a retrieval augmented generation system, you should also capture the

specific aspects of the RAG system that impact what content was inserted into the prompt,

including the query, chunk settings, chunk output, and other information.

Once you feel the prompt is close to perfect, take it to your project codebase. And in the

codebase, save prompts in a separate file from code, so it’s easier to maintain. Finally, ideally

your prompts are part of an operationalized system, and as a prompt engineer you should

rely on automated tests and evaluation procedures to understand how well your prompt

generalizes to a task.

Prompt engineering is an iterative process. Craft and test different prompts, analyze,

and document the results. Refine your prompt based on the model’s performance. Keep

experimenting until you achieve the desired output. When you change a model or model

configuration, go back and keep experimenting with the previously used prompts.

February 2025 65Prompt Engineering

Name [name and version of your prompt]

Goal [One sentence explanation of the goal of this attempt]

Model [name and version of the used model]

Temperature [value between 0 - 1] Token Limit [number]

Top-K [number] Top-P [number]

Prompt [Write all the full prompt]

Output [Write out the output or multiple outputs]

Table 21. A template for documenting prompts

Prompt Engineering UA

**Джерело:** 22365_3_Prompt Engineering_v7 (1).pdf  
**Перекладено:** 2 жовтня 2025 року  
**Мова:** Українська

---

**Теги:** #prompt-engineering #ai #ukraine #nlp #machine-learning #best-practices