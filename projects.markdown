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
        Back then, I wanted to stay up-to-date with ML news from Hacker News, but Algolia's search missed some ML-related posts. Reaping the opportunity to learn Go-lang as well, I created my own lightweight semantic search with HuggingFace and Hacker News APIs. It fetched top scoring 24h posts based on upvotes and ranked based on the upvotes themselves and the similarity score between the query and post titles. I even added subscription management with MailJet! A fun little language learning project! 🚀
      </li>
    </ul>
</div>