There may be errors in the description due to document updates.

# Common Learning Guide

In this repository, we support the fine-tuning of models, DreamBooth, and the learning of LoRA and Textual Inversion (including [XTI:P+](https://github.com/kohya-ss/sd-scripts/pull/327)). This document explains the common preparation methods for learning data and options.

# Overview

Please refer to the README of this repository beforehand and set up the environment.

The following will be explained:

1. Preparation of learning data (using the new format with configuration files)
2. A brief explanation of the terms used in learning
3. Previous specification method (specifying from the command line without using a configuration file)
4. Sample image generation during learning
5. Commonly used options in each script
6. Metadata preparation for fine-tuning method: captioning, etc.

You can learn by just executing 1 (see each script's documentation for learning). Refer to 2 and later as needed.

# Preparing Learning Data

Prepare image files for learning data in any folder (multiple folders are also acceptable). Supported formats are `.png`, `.jpg`, `.jpeg`, `.webp`, and `.bmp`. Preprocessing such as resizing is generally not necessary.

However, it is recommended to avoid using images that are significantly smaller than the learning resolution (described later) or to enlarge them in advance using super-resolution AI. Also, it seems that errors may occur with images larger than extremely large images (around 3000x3000 pixels), so please reduce their size beforehand.

When learning, you need to organize the image data to be learned by the model and specify it to the script. You can specify the learning data in several ways, depending on the number of learning data, learning target, whether captions (image descriptions) can be prepared, etc. The following methods are available (the names of each method are not general, but are unique to this repository). Regularization images will be discussed later.

1. DreamBooth, class+identifier method (regularization images can be used)

    Learn by associating the learning target with a specific word (identifier). There is no need to prepare captions. For example, when learning a particular character, it is easy to use because there is no need to prepare captions. However, since all elements of the learning data, such as hairstyle, clothing, and background, are learned by associating them with the identifier, situations such as not being able to change clothes in the generated prompt may occur.

2. DreamBooth, caption method (regularization images can be used)

    Prepare a text file with captions recorded for each image and learn. For example, when learning a specific character, by describing the details of the image in the caption (character A in white clothes, character A in red clothes, etc.), the character and other elements are separated, and the model can be expected to learn the character more precisely.

3. Fine-tuning method (regularization images cannot be used)

    The captions are collected in a metadata file in advance. It supports functions such as managing tags and captions separately and speeding up learning by pre-caching latents (explained in separate documents). (Although it is called the fine-tuning method, it can also be used for non-fine-tuning.)

The combination of what you want to learn and the available learning methods are as follows:

| Learning target or method | Script | DB / class+identifier | DB / Caption | fine-tuning |
| ----- | ----- | ----- | ----- | ----- |
| Fine-tuning the model | `fine_tune.py`| x | x | o |
| DreamBooth the model | `train_db.py`| o | o | x |
| LoRA | `train_network.py`| o | o | o |
| Textual Inversion | `train_textual_inversion.py`| o | o | o |

## Which one to choose

For LoRA and Textual Inversion, if you want to learn without having to prepare a caption file, the DreamBooth class+identifier method is a good choice. If you can prepare, the DreamBooth caption method is recommended. If you have a large number of learning data and do not use regularization images, consider the fine-tuning method as well.

For DreamBooth, the same applies, but the fine-tuning method cannot be used. In the case of fine-tuning, only the fine-tuning method is available.

# How to specify each method

Here, only typical patterns for each specification method are explained. For more detailed specification methods, please see [Dataset Configuration](./config_README-ja.md).

# DreamBooth, class+identifier method (regularization images can be used)

In this method, each image is learned as if it was learned with the caption `class identifier` (e.g., `shs dog`).

## Step 1. Determine the identifier and class

Decide on the word identifier to associate with the learning target and the class to which the target belongs.

(There are various names such as instance, but for now, we will follow the original paper.)

Here is a brief explanation (please investigate further for details).

The class is a general category of the learning target. For example, if you want to learn a specific dog breed, the class would be dog. For anime characters, depending on the model, it could be boy, girl, 1boy, or 1girl.

The identifier is used to identify and learn the learning target. Any word is acceptable, but according to the original paper, "a rare word of 3 characters or less that is 1 token in the tokenizer" is preferred.

By using the identifier and class, for example, learning the model with "shs dog" allows the learning target to be identified and learned from the class.

When generating images, specifying "shs dog" will generate images of the learned dog breed.

(For reference, here are some identifiers I've been using recently: ``shs sts scs cpc coc cic msm usu ici lvl cic dii muk ori hru rik koo yos wny``. Ideally, it should not be included in the Danbooru Tag.)

## Step 2. Decide whether to use regularization images, and if so, generate them

Regularization images are images used to prevent the entire class from being pulled by the learning target (language drift). If you do not use regularization images, for example, if you learn a specific character with `shs 1girl`, even if you generate a simple `1girl` prompt, it will resemble that character. This is because `1girl` is included in the learning caption.

By learning the target images and regularization images together, the class remains as a class, and the learning target is generated only when the identifier is included in the prompt.

If you only need a specific character to come out in LoRA or DreamBooth, you don't need to use regularization images.

Textual Inversion does not need to be used (because nothing is learned when the token string to be learned is not included in the caption).

For regularization images, it is common to use images generated with only the class name for the learning target model (e.g., `1girl`). However, if the quality of the generated image is poor, you can also use images downloaded separately from the internet or modify the prompt.

(Since the regularization images are also learned, their quality affects the model.)

Generally, it is desirable to prepare about several hundred images (if the number is small, the class images will not be generalized, and their features will be learned).

For generated images, usually, match the size of the generated images to the learning resolution (more precisely, the resolution of the bucket, described later).

## Step 2. Describe the configuration file

Create a text file and set the extension to `.toml`. For example, write as follows:

(The parts starting with `#` are comments, so you can copy and paste them as is, or delete them if you don't need them.)

```toml
[general]
enable_bucket = true                        # Whether to use Aspect Ratio Bucketing

[[datasets]]
resolution = 512                            # Learning resolution
batch_size = 4                              # Batch size

  [[datasets.subsets]]
  image_dir = 'C:\hoge'                     # Specify the folder with the learning images
  class_tokens = 'hoge girl'                # Specify identifier class
  num_repeats = 10                          # Number of repetitions for learning images
