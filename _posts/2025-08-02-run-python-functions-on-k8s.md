---
layout: post
title: Run Python functions on K8s
---
I respect Kubernetes. In the past five years, the majority of my production ML workflows have been running on a company's K8s cluster. Deploying on K8s is also pretty OK for the gains you reap — build an image, write and apply a spec — and your app lands on a best-fit node, kept alive, horizontally (auto)scaled, load-balanced — heck, you even get the fancy stuff like custom controllers to select the best-fit node (my [current employer](https://cast.ai/) does some fancy stuff like that).

That said, for offline experimentation — think notebooks, ad-hoc scripts — I rarely see people using K8s. DS spend most of their time in experimentation phase! When you suddenly need half a terabyte of RAM or a GPU or two, you'd often skip K8s — it's a bit of a hassle: containerizing and writing a deployment spec all over again. Interactive notebooks are god-sent, and K8s's strictly start-end jobs don't offer that experience. Instead, we turn to always-alive managed notebooks, serverless SDKs (like SageMaker), or large VMs. The tradeoffs are significant — forgotten VMs burning cash, every serverless millisecond costing times the regular compute. Even the remote IDEs are a downgrade compared to a local development environment.

To illustrate the issue, let's say you have a function that uses a pre-trained image captioning model, to generate descriptions. 

```python
def image_contents(image_bytes):
    from transformers import pipeline
    from PIL import Image
    import io
    import torch

    # Load and convert the image bytes to PIL
    image = Image.open(io.BytesIO(image_bytes))
    if image.mode != "RGB":
        image = image.convert("RGB")

    # Choose a captioning model—e.g. BLIP or ViT‑GPT2
    captioner = pipeline("image-to-text", model="Salesforce/blip-image-captioning-base")
    # Alternatively: model="nlpconnect/vit-gpt2-image-captioning"

    # Run the pipeline
    outputs = captioner(image)

    # outputs is a list of dicts like [{'generated_text': "..."}]
    caption = outputs[0]["generated_text"]

    return {"description": caption}


...

caption = image_contents(image_bytes)["description"]
```

If you suddenly need a much beefier machine for your function, you have to either restructure your project — build a Docker image, write a spec - make sure the secrets are on the cluster, configure the env vars - and deploy on K8s—or spin up an expensive large VM with no easy VSCode access. Or worse, slowly lose your sanity wrestling with SageMaker functions. We should be able to keep local development seamless _and_ leverage K8s cost-efficiency.

We need to make it easy to schedule Python functions on K8s. Inspired by [Modal](https://modal.com) and Airflow’s [KubernetesPodOperator](https://airflow.apache.org/docs/apache-airflow-providers-cncf-kubernetes/stable/operators.html#kubernetespodoperator) ,  I built a small package over the week: [`kuberun`](https://github.com/astronautas/kuberun), a decorator that runs Python functions on cluster as if they were local functions.
## Meet @kuberun

You can simply decorate your function with `@kuberun`, and it will execute on a Kubernetes cluster — passing in the arguments and retrieving the result all behind the scenes:

```python
@kuberun(requirements=["torch", "transformers", "pillow"], cpu=32, mem="64Gi")
def image_contents(image_bytes):
    from transformers import pipeline
    from PIL import Image
    import io
    import torch

    # Load and convert the image bytes to PIL
    image = Image.open(io.BytesIO(image_bytes))
    if image.mode != "RGB":
        image = image.convert("RGB")

    # Choose a captioning model—e.g. BLIP or ViT‑GPT2
    captioner = pipeline("image-to-text", model="Salesforce/blip-image-captioning-base")
    # Alternatively: model="nlpconnect/vit-gpt2-image-captioning"

    # Run the pipeline
    outputs = captioner(image)

    # outputs is a list of dicts like [{'generated_text': "..."}]
    caption = outputs[0]["generated_text"]

    return {"description": caption}


...

caption = image_contents(image_bytes)["description"]
```

Under the hood, it builds a rather standard templated Docker image and generates a just enough Kubernetes Pod spec. The image is pushed to the configured registry, and the spec is applied to the cluster (within a default namespace, configurable though). Arguments and results are pickled — like in Ray or Python’s `multiprocessing`— and exchanged through a shared volume with a sidecar container.

Builds are very slow, and I haven’t figured out support for annotations or environment variables yet (without manual spec writing, which defeats the purpose). Still, the concept gives me energy, so give it a try—and share your feedback via an [issue](https://github.com/astronautas/run_on_k8s/issues)! 

<hr>

<sup>1</sup>  I sort of accidentally found out that the name `kuberun` had been already used for some fancy [ssh to container tool](https://github.com/ContainerSSH/kuberun). It seems to be deprecated. Fair if I borrow the name. It's a pet project after all...