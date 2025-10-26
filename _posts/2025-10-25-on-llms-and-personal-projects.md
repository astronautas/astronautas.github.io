---
layout: post
title: LLMs are the missing link to shipping personal projects
---
I've been procrastinating on shipping a small UI improvement to my blog, and an LLM made it happen.

<!--more-->

Some time ago, I was toying around with an idea for a tech product. I was procrastinating on making a landing page. Pixel pushing, as I used to call it, is an excruciating process. So, I asked an LLM<sup>1</sup> to do it for me. "Surprise me," I thought. In a few minutes of prompting, I was amazed. Here it was — an interactive website with callouts, a contact form, and more. But that joy vanished quickly. Looking deeper, it felt too generic. I tried multiple times, but nothing felt right. I didn't really know what made a good landing page, so LLMs couldn't help me much either. I parked the idea of an autonomous coding assistant then.

Later, I found myself procrastinating again. I'd been postponing an idea to give a breath of fresh air to my personal blog. I had borrowed a simple idea from [Andrej Karpathy's](https://karpathy.github.io/) blog: move posts into the landing page and the bio section into a separate about page. It meant some yawn-inducing HTML and CSS. Once more, I called on an LLM<sup>1</sup>. It nailed it. With some interim feedback and a quick code review, I had a good result in a few minutes instead of hours.

![img](/assets/image-new.png)

Makes me think. In the first situation, I had no <u>vision</u> of the end result—I didn't know what makes a good landing page. I could barely give the agent useful feedback. It saved some typing time but produced no good results. In the second, I had a <u>clear end state</u> in mind, so a bit of guiding brought us to a satisfactory end result, in no time.

It seems I can finally ship <u>well-scoped</u> features with obvious (so <u>boring</u>) implementations. Previously, I would put them aside for undetermined periods. Exciting times ahead - until agents become higher-goal-directed and start asking six figures and equity for side projects.

<small><sup>1</sup> In both cases, I used Claude Sonnet 3.5 and 4, via GitHub's Copilot VSCode extension. The latter experiment obviously benefited from a more powerful model, but I doubt it was a significant factor.</small>