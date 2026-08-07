---
title: "Detectrode: Training a small vision model to recognise industrial products"
description: "Training a visual language model with nanoVLM"
date: 2026-08-02
tags: [local-ai, computer-vision, vision-language-models, nanovlm, pytorch, machine-learning]
draft: false
---

I started Detectrode with a fairly narrow question: could I train a small vision model locally
to recognise products made by a particular manufacturer?

The products are by imaginary Umbrella Corp. They produce zombie protection umbrellas. Product names are
not always visible, photographs arrive from different angles, and a useful detector also needs
to know when the photograph is not a product at all.

This made the project more interesting than ordinary image classification. I did not only want
a label. I wanted a small model that accepts an image and a question, then returns a predictable
answer that another application can consume:

```json
{"isDetected": true, "product": "Zombrella"}
```

For an unrelated image, the expected answer is:

```json
{"isDetected": false, "product": "Unknown"}
```

## Why nanoVLM

I used open source [nanoVLM](https://github.com/huggingface/nanoVLM) to start the project.
It is a deliberately small and readable vision-language model, which made it much more useful
for this experiment than treating a large pretrained model as a remote black box.

The project uses two backbones. A SigLIP2 vision transformer converts the image into patch
representations. A small projection layer, including pixel shuffle, maps those representations
into the language model's embedding space. SmolLM2 then generates the textual answer. In the
configuration I settled on, the vision input is 512 pixels and the language backbone is the
135M parameter SmolLM2 Instruct model.

## Diversifying the dataset is challenging

The first version of the idea was simple: put each product's photographs in a directory named
after it, and a preparation script will standardize images, create augmentations, and generate the jsonl files.

Detectrode keeps original positive and negative images under `source_dataset`. A metadata file
maps stable, kebab-cased directory identifiers to display names. It currently describes 64
products, which avoids teaching the model accidental differences in capitalisation or spelling.
The preprocessing step normalises several input formats to JPEG, resizes large images while
preserving their proportions, and gives files deterministic names.

The population step copies those sources into a generated dataset and applies transformations (augmentations)
such as changes in brightness and contrast, blur, noise, hue shifts, small rotations,
perspective distortion and crops. These transformations are not there to make a large dataset
look impressive. They approximate the unhelpful things that happen to real photographs:
different lighting, imperfect focus, odd framing and a product that is not facing the camera
quite as politely as it did in the catalogue.

Positive examples teach the product name. Negative examples contain animals, furniture, food,
people, scenery and unrelated items, and teach the model to return `Unknown`. This second
group matters. Without it, a closed-set classifier has no reason not to identify a kitten as the
industrial controller it resembles least badly.

The pipeline produces positive-only, negative-only and combined JSONL files. I normally train
with the combined version. Keeping the source images separate from generated images also means
I can change augmentation policy and rebuild the dataset without slowly corrupting the originals.

## Training within the hardware limit

The upstream nanoVLM configuration was designed for broad multimodal training. Detectrode moved
in the other direction: a local JSONL dataset, local checkpoints for each pass, no streaming from Hugging Face, a smaller language
backbone, and general-purpose multimodal evaluation switched off.

Memory shaped the rest of the configuration. Training uses a batch size of one with eight-step
gradient accumulation, so the effective batch is less tiny without requiring all eight samples
to be resident at once. Both backbones use gradient checkpointing, sequence length is capped at
1536, and the number of images packed into a sample is restricted. These choices trade training
speed for memory. That is a reasonable exchange when the alternative is an out-of-memory error.

Training records validation loss, generated-token confidence and throughput, and saves a
checkpoint at evaluation intervals. Past a certain number of passes, the loss ratio becomes negligible which means I can stop the training as subsequent runs will not improve quality further.

## Utilities

Leveraging the excellent [Albumentations](https://pypi.org/project/albumentations) library, I was able to take a small sample set,
generate a dozens of variations and provide a meaningfully large sample set around 10k from around 1000 images. I've used `nvidia-smi` to limit max voltage for the GPU, so it can stay cool during the training.
The configuration should be scaled based on your particular hardware, but on an RTX3090 it usually takes about 2 hours. The resulting model identifies an image under 1 second on the same hardware.


## Evaluation

The test runner loads a checkpoint, performs inference for each image, parses the generated JSON
and compares both `isDetected` and `product`. It continuously writes an HTML report with the total,
pass rate, failures per label and individual predictions, so a long run still leaves useful
evidence if it stops halfway through.

The report currently stored in the repository records 17,327 passes from 18,026 samples, or
96.1%. But a more meaningful test is a separate folder of images that did not participate in dataset
generation. Failures grouped by product are especially useful there. They reveal whether the
problem is a particular pair of visually similar items, insufficient viewpoints, an output
formatting failure, or a detector that has learned the background of the photographs rather than
the object in them.

## What I learned

Detectrode is not a general vision system, and that is the point. It is a small model trained
for a bounded vocabulary on hardware I control. The interesting engineering work was not merely
getting an image through a transformer. It was defining what “not one of ours” means, preserving
clean source data, making model output deterministic enough for software, and building an
evaluation loop that makes mistakes visible.

The model architecture made the experiment possible, but the dataset contract made it useful.
For narrow local AI projects, I increasingly think that is where most of the design lives.
