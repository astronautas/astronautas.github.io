---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

<style>

@media (min-width: 500px) {
.grid-container {
    display: grid;
    grid-template-columns: 1.5fr 2fr;
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
        <p>My name is Lukas. I am a software engineer who loves using data and machine learning to tackle today's most challenging issues.</p>
        <p>I build software for the entire lifecycle of machine learning. I design models, as well as work on MLOps, to deliver accurate, scalable and reliable systems that learn and adapt, in production.</p>
        <p>I currently help <a href="https://www.vinted.fr/" target="_blank">Vinted</a> recommend the most relevant second-hand clothing to millions of users. In the past, 
        I designed <a href="https://arxiv.org/abs/1910.06658" target="_blank">autonomous mobile robots</a>.</p>
        <!-- <p>Read the full CV here.</p> -->
        <p>In my spare time, I practice <a href="2021/11/21/no-time-to-plan.html" target="_blank">Traditional Karate-do</a>.</p>
        
        <p>I reside in Vilnius, Lithuania, at the moment.</p>
    </div>
</div>
