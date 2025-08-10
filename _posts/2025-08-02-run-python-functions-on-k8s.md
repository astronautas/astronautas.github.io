---
layout: post
title: Run Python functions on K8s
---
I love Kubernetes — for production, it’s rock-solid. But throw me a notebook that needs 0.5 TB of RAM and a GPU, and suddenly K8s feels like a chore. 

Here’s what that usually means:

1. Write a Dockerfile.
2. Build the image.
3. Push it to a registry.
4. Write a deployment spec.
5. Deploy it to the cluster.
6. Set up secrets and environment variables.
7. Wait for the pod to start.
8. Port forward to Jupyter within the pod.
9. Finally, run the notebook.

This gets even more interesting if you forget to shutdown the pod. 24 * 10 USD / h = **240 USD / day**, down the drain 🤦.

Meet [kuberun](https://github.com/astronautas/kuberun).  
Just decorate your function with `@kuberun` and it will run on a Kubernetes cluster as if it were local.  It quietly handles the Docker build, push, deployment, and wiring — you just write Python:

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

As soon as the function returns, the pod and any related resources get immediately cleaned-up.

Zooming in:
```python
@kuberun(requirements=["torch", "transformers", "pillow"], cpu=32, mem="64Gi")
def you_func(arg):
    ...
```

Under the hood, it builds a rather standard templated Docker image and generates a just enough Kubernetes Pod spec. The image is pushed to the configured registry, and the spec is applied to the cluster (within a default namespace, configurable though). Arguments and results are pickled — like in Ray or Python’s `multiprocessing`— and exchanged through a shared volume with a sidecar container.

Builds are very slow, and I haven’t figured out annotations, secrets, environment variables yet (without manual spec writing, which defeats the purpose). Still, the concept gives me energy, so give it a try—and share your feedback via an [issue](https://github.com/astronautas/run_on_k8s/issues)! 

Happy experimentation!

<hr>

<sup>1</sup>  I sort of accidentally found out that the name `kuberun` had been already used for some fancy [ssh to container tool](https://github.com/ContainerSSH/kuberun). It seems to be deprecated. Fair if I borrow the name. It's a pet project after all...
