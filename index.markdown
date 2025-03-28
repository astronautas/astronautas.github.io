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

    #logo {
        width: 33%; 
        float: right; 
        margin-left: 60px; 
        margin-bottom: 30px;
    }
}

@media (max-width: 700px) {
    .grid-container {
        display: block;
    }

    .grid-item {
        margin-bottom: 30px;
    }

    #logo {
        width: 100%;
        height: auto;
        margin-top: 30px;
        margin-left: 0;
        margin-bottom: 30px;
    }

    #logo-img {
        width: 60%;
        height: auto;
        margin-bottom: 0px;
    }
}

</style>

<div class="grid-containers">
    <h1>Hi, I am Lukas 👋</h1>

    <div id="logo">
        <center><img class="grid-item one" id="logo-img" src="/assets/IMG_0762.jpeg" /></center>
    </div>

    <div class="grid-item two">
        <p>
        I’m a software engineer. I have 6+ years of experience building production machine learning systems — think efficient model deployment stacks, high-throughput feature pipelines, low-latency model serving and monitoring — commonly known as MLOps. This involves non-trivial amount of data engineering, backend development, and some data science, primarily using Python, Go, and cloud stacks.
        </p>

        <p>
        I love good beer (thanks, Belgium), cycling (gravel most of time), and lately, I’ve been getting more into endurance competitions.
        </p>

        <h2>My Engineering Journey So Far</h2>

        <p>I grew up in vibrant Vilnius, Lithuania — thrilled to see more people now associate my beloved city with startups and fintech — where I earned a degree in Computer Science. I kickstarted my career in large-scale enterprise application development with Java. As a student job, it was more than I could’ve asked for: surrounded by senior engineers and working on a >100k LOC project. Even today, I still have plenty to say about API development and integration testing, thanks to that experience. Around 2017, as machine learning started gaining traction, my curiosity was sparked to explore new directions.</p>

        <p>Hence, early 2018, I fully pivoted to ML. For a brief period, I trained <a href="https://arxiv.org/abs/1910.06658" target="_blank">deep neural networks</a> for autonomous mobile robots. The half-year went by quickly, but I realized that research wasn’t my calling — I found myself drawn back to product development.</p>

        <p>From 2019 to 2023, I had a rewarding run building ML for high-growth scale-ups. At <a href="https://vinted.engineering/" target="_blank">Vinted</a>, I scaled the first versions of home screen recommendations, search ranking, and a millisecond-latency feature store (“I’m quite a Redis developer myself”). Rumour goes some of those components are still running, though hopefully revamped by folks better than me! Overall, these systems drove double-digit improvements in key business metrics and helped establish ML-powered ranking and recommendations as a standalone data product. Afterwards, I had a short but impactful stint at <a href="https://www.otrium.com/">Otrium</a>, where I automated model prediction flows into ERP systems (reverse ETL), saving hours of manual work, and introduced A/B testing to de-risk releases. To this day, I believe most of my engineering muscle was built in these growth-stage environments — an experience I’d recommend to anyone.</p>

        <p>
        Afterwards, my family and I moved to picturesque Leuven, Belgium, where I’ve been helping <a href="https://dataroots.io/">Dataroots</a> scale MLOps for local companies. My focus has been on building developer tooling for model deployment — think reducing deployment cycles from months to weeks, while upskilling on DevOps along the way. I soon learned building tools for experts is often harder than building for end users...
        </p>

        <p>
        Stay tuned for my next stop — news coming very soon!
        </p>
</div>
