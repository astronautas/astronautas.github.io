---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

<style>

@media (min-width: 500px) {
.grid-container {
    display: grid;
    grid-template-columns: 1.2fr 2fr;
    gap: 30px;
}
}

@media (max-width: 499px) {
.grid-container {
    display: block;
}

.grid-item {
    margin-bottom: 30px;
}
}

</style>

<div class="grid-container">
    <img class="grid-item one" src="/assets/IMG_0762.jpeg" />
    <div class="grid-item two">
        <h1>👋 Welcome!</h1>
        <p>My name is Lukas Valatka. I am a machine learning engineer.</p>
        
        <p>For the past few years, I have been developing recommender systems at <a href="https://www.vinted.fr/" target="_blank">Vinted</a>. Here I co-created many critical systems that enable real-time personalization, such as a first in-house feature store.
        Before this, I trained <a href="https://arxiv.org/abs/1910.06658" target="_blank">deep neural networks</a> for autonomous mobile robots. I have a degree in Computer Science (Software Engineering).</p>

        <p>In my spare time, I practice <a href="2021/11/21/no-time-to-plan.html" target="_blank">Traditional Karate-do</a>. I reside in my lovely hometown Vilnius, Lithuania.</p>

        <!-- <p>Read the full CV here.</p> -->
    </div>
</div>
