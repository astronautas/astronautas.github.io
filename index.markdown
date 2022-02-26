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
        <p>My name is Lukas Valatka. I am a software engineer. I build adaptive systems with machine learning to support businesses and improve user experience in most dynamic settings.</p>
        <p>I currently develop recommender systems at <a href="https://www.vinted.fr/" target="_blank">Vinted</a>, to enable a personalized second-hand clothing shopping experience to millions of users. In the past, 
        I built autonomous mobile  <a href="https://arxiv.org/abs/1910.06658" target="_blank">robots powered by deep neural networks</a>.</p>
        <!-- <p>Read the full CV here.</p> -->
        <p>In my spare time, I practice <a href="2021/11/21/no-time-to-plan.html" target="_blank">Traditional Karate-do</a>.</p>
        
        <p>I reside in Vilnius, Lithuania, at the moment.</p>
    </div>
</div>
