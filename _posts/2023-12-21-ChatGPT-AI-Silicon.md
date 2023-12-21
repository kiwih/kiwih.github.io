---
layout: post
title: 'Chip-Chat: My journey making the world's first LLM-architected and taped-out silicon design'
subtitle: Using ChatGPT to write Verilog and produce a chip for my christmas tree
share-img: ''
tags: [AI, EDA, Verilog]
---

TL;DR: I architectured a microcontroller chip using ChatGPT and it was taped out. Now it controls my christmas tree.

This was the first time anyone had used an LLM to design silicon.

<video width='60%' controls>
  <source src="{{ '/assets/vid/chip-chat/chip-chat.mp4' | relative_url }}" type="video/mp4">
Your browser does not support the video tag.
</video>

# The full story: Part 1, the Idea

In March of this year [a post made it to the front page of Hacker News regarding Tiny Tapeout](https://news.ycombinator.com/item?id=35376645). Here, they had made a process where you could submit custom, miniature silicon designs to a subset of a silicon shuttle for just $100. 

At the time, I was working at NYU on my postdoc, which among other things, was exploring the use of [Large Language Models for Verilog](https://arxiv.org/abs/2212.11140). We were benchmarking a variety of different applications for the use of LLMs like ChatGPT for designing hardware, including in specification interpretation, design, and bug detection and repair. We were one of the very first movers in this space, using GPT-2 and Verilog all the way back in 2020.

I was immediately interested, then, in the HN post. We'd always been working with FPGAs and simulations due to the prohibitive cost of a real tapeout. But, there's always a simulation-reality gap, so showing that LLMs and AI really could produce chips could be a boon for our research area. Could we use Tiny Tapeout as a vehicle for this - and use an LLM to not just write Verilog, but also to design the Verilog for a real chip? 

![general idea.]({{ 'assets/img/chip-chat/chip-chat-general-idea.drawio.png' | relative_url }}){: .mx-auto.d-block :}

I spoke to my supervisors, and several other Ph.D students, and we brainstormed a few ideas. Tiny Tapeout is extremely small - just a thousand standard cells or so, meaning that the design would be very constrained, but we all really liked the idea, especially as it seemed nobody had done it yet - so if we moved quickly, we might be able to do a world's-first!

So, we decided to go for it. But now, there were many other things to consider. What should we submit, given the design space was so small? And there are other concerns too. We knew from our own prior work that LLMs can write hardware design languages like Verilog, but they just aren't very good at it, with much higher incidences of syntax or logic errors compared with more popular languages like Python - This is actually why me and my team have produced our own LLMs for Verilog, but I digress. So we needed to decide - if we did want to make a chip using an LLM, (1) which LLM should we use? (2) How much help should we give it? (3) What prompting strategy should we try?

# Part 2: Designing a methodology

We first decided on the LLMs. Instead of using the 'autocomplete' style LLMs which we had been working with up to this point, primarily OpenAI's Codex and Salesforce CodeGen, we'd use the newer and flashier 'conversational' / 'instructional' style LLMs. We picked OpenAI's ChatGPT, versions 3.5 and 4, Google's Bard, and the open-source HuggingChat. 

We then worked out two methods. The first would try and have the LLMs complete everything in a kind of feedback loop, whereby the LLM would be given a specification, then produce both the design and the tests for that design. A human would then run the tests and design in a simulator (iVerilog) and then return any errors to the LLM.

However, we knew from experience that LLMs are also fairly silly sometimes, and can get stuck in loops where they think they are fixing a problem or improving an output when really they're just iterating over the same data. So we theorized that we might need to give back 'human assistance' sometimes.

Through some preliminary experimentation, we decided on an initial process that looked like this:

![chip-chat process flow.]({{ 'assets/img/chip-chat/chip-chat-flowchart.drawio.png' | relative_url }}){: .mx-auto.d-block :}

Ideally, the human would not need to provide _much_ input, but this remained to be seen...

**To be continued...**