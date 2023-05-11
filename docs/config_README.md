This a ChatGPT-4 English Conversion of kohya-ss (config_README-ja.md)


This documentation explains the configuration file that can be passed using the `--dataset_config` option.

## Overview

By providing a configuration file, users can fine-tune various settings.

* Multiple datasets can be configured.
    * For example, you can set the `resolution` for each dataset and train them together.
    * In learning methods that support both DreamBooth and fine-tuning techniques, it is possible to mix datasets using DreamBooth and fine-tuning techniques.
* Settings can be changed for each subset.
    * A dataset is a collection of subsets, which are created by dividing the dataset into separate image directories or metadata.
    * Options such as `keep_tokens` and `flip_aug` can be set for each subset. On the other hand, options such as `resolution` and `batch_size` can be set for each dataset, and the values are shared among subsets belonging to the same dataset. More details are provided later.

The configuration file can be written in JSON or TOML format. Considering ease of writing, we recommend using [TOML](https://toml.io/ja/v1.0.0-rc.2). The following explanations assume the use of TOML.

Here is an example of a configuration file written in TOML:

```toml
[general]
shuffle_caption = true
caption_extension = '.txt'
keep_tokens = 1

# This is a DreamBooth-style dataset
[[datasets]]
resolution = 512
batch_size = 4
keep_tokens = 2

  [[datasets.subsets]]
  image_dir = 'C:\hoge'
  class_tokens = 'hoge girl'
  # This subset has keep_tokens = 2 (using the value of the parent datasets)

  [[datasets.subsets]]
  image_dir = 'C:\fuga'
  class_tokens = 'fuga boy'
  keep_tokens = 3

  [[datasets.subsets]]
  is_reg = true
  image_dir = 'C:\reg'
  class_tokens = 'human'
  keep_tokens = 1

# This is a fine-tuning-style dataset
[[datasets]]
resolution = [768, 768]
batch_size = 2

  [[datasets.subsets]]
  image_dir = 'C:\piyo'
  metadata_file = 'C:\piyo\piyo_md.json'
  # This subset has keep_tokens = 1 (using the general value)
```

In this example, three directories are trained as DreamBooth-style datasets at 512x512 (batch size 4), and one directory is trained as a fine-tuning-style dataset at 768x768 (batch size 2).

## Dataset and Subset Configuration Settings

The settings for datasets and subsets are divided into several sections.

* `[general]`
    * This section specifies options that apply to all datasets or all subsets.
    * If an option with the same name exists in the dataset-specific and subset-specific settings, the dataset and subset-specific settings take precedence.
* `[[datasets]]`
    * `datasets` is the registration section for dataset settings. This section specifies options that apply individually to each dataset.
    * If subset-specific settings exist, the subset-specific settings take precedence.
* `[[datasets.subsets]]`
    * `datasets.subsets` is the registration section for subset settings. This section specifies options that apply individually to each subset.

The following is a conceptual diagram of the correspondence between the image directories and registration sections in the previous example:

```
C:\
├─ hoge  ->  [[datasets.subsets]] No.1  ┐                        ┐
├─ fuga  ->  [[datasets.subsets]] No.2  |->  [[datasets]] No.1   |->  [general]
├─ reg   ->  [[datasets.subsets]] No.3  ┘                        |
└─ piyo  ->  [[datasets.subsets]] No.4  -->  [[datasets]] No.2   ┘
```

Each image directory corresponds to one `[[datasets.subsets]]`. One or more `[[datasets.subsets]]` are combined to form a `[[datasets]]`. The `[general]` section includes all `[[datasets]]` and `[[datasets.subsets]]`.

Different options can be specified for each registration section, but if an option with the same name is specified, the value in the lower registration section takes precedence. It may be easier to understand by checking how the `keep_tokens` option is handled in the previous example.

In addition, the available options vary depending on the supported techniques of the learning method.

* DreamBooth-specific options
* Fine-tuning-specific options
* Options available when the caption dropout technique can be used

In learning methods that support both DreamBooth and fine-tuning techniques, both can be used together.
When using both, note that whether a dataset is a DreamBooth-style or fine-tuning-style is determined on a dataset-by-dataset basis, so it is not possible to mix DreamBooth-style subsets and fine-tuning-style subsets within the same dataset.
In other words, if you want to use both of these techniques, you need to set the subsets with different techniques to belong to different datasets.

Regarding the program's behavior, it is determined that a subset is a fine-tuning-style subset if the `metadata_file` option, which will be explained later, exists.
Therefore, for subsets belonging to the same dataset, there is no problem as long as they are either "all have the `metadata_file` option" or "all do not have the `metadata_file` option".

The following describes the available options. For options with the same name as command-line arguments, the basic explanation is omitted. Please refer to the other READMEs.

### Common Options for All Learning Methods

These options can be specified regardless of the learning method.

#### Dataset-specific Options

These options are related to dataset settings and cannot be written in `datasets.subsets`.

| Option Name | Example Setting | `[general]` | `[[datasets]]` |
| ---- | ---- | ---- | ---- |
| `batch_size` | `1` | o | o |
| `bucket_no_upscale` | `true` | o | o |
| `bucket_reso_steps` | `64` | o | o |
| `enable_bucket` | `true` | o | o |
| `max_bucket_reso` | `1024` | o | o |
| `min_bucket_reso` | `128` | o | o |
| `resolution` | `256`, `[512, 512]` | o | o |

* `batch_size`
    * Equivalent to the command-line argument `--train_batch_size`.

These settings are fixed for each dataset.
In other words, subsets belonging to the dataset share these settings.
For example, if you want to prepare different resolution datasets, you can set different resolutions by defining them as separate datasets, as shown in the previous example.

#### Subset-specific Options
