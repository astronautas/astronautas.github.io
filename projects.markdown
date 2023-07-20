---
layout: page
title: Projects
permalink: /projects/
---

<div class="home">
    <!-- <h2 class="post-list-heading">{{ page.list_title | default: "Posts" }}</h2> -->
    <ul class="post-list">
      <li>
        <h3>
          <a class="post-link" href="https://astronautas.github.io/your-hnews/">
            Your Hacker News
          </a>
        </h3>
        At the time, I wanted to subscribe to ML-specific topics at Hacker News. 
        <a href="https://hn.algolia.com/">
            Algolia's HN search
        </a> would skip quite some ML-related posts. Since it does some form of full-text search, so titles with keywords such as "NLP" would never match queries such as "machine learning", and wouldn't be that precise e.g. articles about learning in general would be retrieved at the top. I was also learning Go-lang at the time, so I toyed around and composed a lightweight semantic similarity-based search using HuggingFace APIs and Hacker News APIs. For multiple topics of your interests, it retrieves highest scoring 24h posts, based on a combination of upvotes and similarity score of the query and a post title. It leverages MailJet, to manage subscriptions. Overall, a fun little new language learning project!
      </li>
    </ul>
</div>