---
layout: post
title: 'Thoughts on AI-generated content'
subtitle: GPT-3, DALL-E, Codex, and more
share-img: 'assets/img/thoughts-on-ai/DALL-E 2022-07-21 10.48.41 - A fox and owl in the style of American Gothic.png'
tags: [AI, opinion]
---
It was long expected that among all the practices which scientists could apply computers to, that art would remain safe: after all, how can one program a machine to have creativity?

Yet, in the last two years we have seen an explosive emergence of new AI-based tools that are not tailored to analysis or control, but advertise their utility for creation.
Already, commercial tools are coming online: like [Jasper](https://www.jasper.ai/), which produces free-form prose; [DALL-E](https://openai.com/dall-e-2/), which can create digital and photo-realistic art; and [GitHub Copilot](https://github.com/features/copilot), which writes computer programs and code.

As GPT-2/large says, "`It is the beginning of a new era: the "Artificial Creativity" era. We are beginning to see a paradigm shift in how we think about the creative process, and in how we think about computers and creativity itself.`" (I prompted it with the article contents up to the opening quote mark using [huggingface](https://transformer.huggingface.co/doc/distil-gpt2)).

In many ways these tools are awe-inspiring, capable of performance that few technologists would have predicted even five years ago. It's verifiable performance, too: many designs are published academically, and others are made free to use online. For instance, we can produce art using [craiyon](https://www.craiyon.com/) at no cost.

![craiyon art.]({{ 'assets/img/thoughts-on-ai/craiyon-2022-07-21 An AI making art.png' | relative_url }}){: .mx-auto.d-block :}
*Craiyon - An AI making art*

That said, although we might know the general structure of their architecture of the most capable models, training these AI models is an expensive task (estimates for the art-producing DALL-E version 2 range from hundreds of thousands to millions of dollars; for the prose-writing GPT-3 the range is in the tens of millions of dollars).
In addition, these models will typically require powerful computers in order to produce their outputs: there's no simply running it on your desktop PC.

![DALL-E art.]({{ 'assets/img/thoughts-on-ai/DALL-E 2022-07-21 10.20.46 - a kiwi bird programs a computer, digital art.png' | relative_url }}){: .mx-auto.d-block :}
*DALL-E - a kiwi bird programs a computer, digital art*

It is still too early to say what the industry impact of such AI-based products will be, although early signs are promising for the product designers: for instance, in one statement, [GitHub has touted that Copilot now produces 30% of all new committed code in common languages such as Java and Python, and 50% of all developers that tried the AI in their technical preview have left it switched on](https://www.axios.com/2021/10/27/copilot-artificial-intelligence-coding-github). 

![copilot art.]({{ 'assets/img/thoughts-on-ai/copilot-estimate-value.png' | relative_url }}){: .mx-auto.d-block :}
*Copilot estimates the value of generative AI.*

However, there are more kinds of impacts than just monetary, and we are yet to fully understand the full consequences of applying content-creating AI.
We have seen warnings that the text-generating models such as GPT-3 can be tailored to produce abusive and distressing content at a frightening pace (imagine an automated cyber-bully!). Or, more insidiously, they can be used to change the tone of existing text, creating, as one article introduces, [risks of 'Propaganda-as-a-Service'](https://www.computer.org/csdl/proceedings-article/sp/2022/131600b532/1CIO7BDk9sA).
Further, [as my own research](https://arxiv.org/abs/2108.09293) has separately shown, the models such as GitHub Copilot which write computer code do not have any intrinsic understanding of what it is they are producing - and as such are prone to emitting code which is problematic from a cybersecurity point of view. As such, what might the consequences be if Copilot is paired with a naive junior developer? Might we see a new rise of computer vulnerabilities produced by AI?

![copilot art 2.]({{ 'assets/img/thoughts-on-ai/copilot-prepares-an-sql-vulnerability.png' | relative_url }}){: .mx-auto.d-block :}
*Copilot prepares an SQL injection vulnerability.*

The image-generating models are amongst the newest commercial offerings, but even there we see apparent risk. [OpenAI has already stated](https://openai.com/blog/dall-e-2-pre-training-mitigations/) that they take active efforts to ensure that their DALL-E model does not propagate harmful biases (for instance, only generating 'scientists' or 'CEOs' that are white men). They also prevent the generation of images that might be utilized for 'fake news' (for instance, you cannot use DALL-E to produce an image with Barack Obama or Donald Trump). It is good that they recognize and correct for these risks, but this does not guarantee that all purveyors of such AI will have the same ethics.

![dall-e art 2.]({{ 'assets/img/thoughts-on-ai/DALL-E 2022-07-21 11.01.07 - two friends eat forbidden lasagna.png' | relative_url }}){: .mx-auto.d-block :}
*DALL-E - two friends eat forbidden lasagna.*

That said, even with the risks of mis-use, we cannot simply ignore these AI models. They are simply too powerful, too capable to disregard. As such, we should be encouraging experimentation and understanding, so that society can interact with this new technology in a healthy and productive way. We should ensure ethical oversight of the most powerful models, and encourage research to identify when AI models have been mis-used to create fictional,  divisive, or faulty content. 

![DALL-E art 2.]({{ 'assets/img/thoughts-on-ai/DALL-E 2022-07-21 10.48.41 - A fox and owl in the style of American Gothic.png' | relative_url }}){: .mx-auto.d-block :}
*DALL-E - A fox and owl in the style of American Gothic*

With careful `consideration of the potential downsides to this technology, and the ways in which society can best interact with it , we can ensure that we protect our future while maximizing its potential for progress.` *(gpt2/large)*.

*Author note:* I have not received any sponsorship or funding from any company mentioned in this blog post.