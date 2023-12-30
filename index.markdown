---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

<style>

@media (min-width: 500px) {
.grid-container {
    display: grid;
    grid-template-columns: 2fr 1.2fr;
    gap: 30px;
}
}

@media (max-width: 499px) {
.grid-container {
    display: block;
}

.grid-item {
    margin-bottom: 30px;
    text-align: justify;
}
}

</style>

<div class="grid-containers">
    <h1>Hi, I am Lukas Valatka</h1>
    <img class="grid-item one" src="/assets/IMG_0762.jpeg" style="width: 33%; float: right; margin-left: 60px; margin-bottom: 30px;" />

    <div class="grid-item two" style="text-align: justify;">
        <p>I live in Leuven, Belgium and I am a Software Engineer. I currently help <a href="https://dataroots.io/">Dataroots</a> - a Belgian scale-up within data and A.I. - with MLOps.</p>

        <p>In 2023, I joined  <a href="https://www.otrium.nl/">Otrium</a>, a Dutch start-up in the online fashion outlet marketplace, to create the dynamic pricing engine. Beyond development, I influenced the pricing team's engineering culture, promoting iterative development through A/B testing. I also reduced manual labor by leading a multi-team project to seamlessly integrate pricing model updates into our internal inventory management system.
        </p>
        
        <p>From 2019 to 2022, I was at <a href="https://www.vinted.fr/" target="_blank">Vinted</a>. I have built
        the first version of home screen recommendations, search ranking and a milisecond-latency feature store - some of the initial components still operate up to this day. These implementations marked a significant achievement in key metrics, firmly establishing machine learning-powered ranking and recommendations as a standalone data product within Vinted. Most of my muscle had been built there.</p>
        
        <p>Before this, I trained <a href="https://arxiv.org/abs/1910.06658" target="_blank">deep neural networks</a> for autonomous mobile robots, and even built some JAVA backends (Spring boot, anyone?)... I have a degree in Computer Science (Software Engineering).</p>

        <p>I usually write Python. Most of what I've been doing has been data science, data engineering and backend development. I am currently exploring Rust and Go.</p>

        <p>In my spare time, I practice <a href="2021/11/21/no-time-to-plan.html" target="_blank">Traditional Karate-do</a> and cycle.</p>

        <!-- <p>Read the full CV here.</p> -->
    </div>
</div>
