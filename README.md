# AMG-Embedding

A self-supervised method for feature extraction from audio.

![image-20240314145922070](https://github.com/syh200626/AMG-Embedding/assets/83171204/6de00f03-83d8-427d-969e-187a1bbaada1)

## Dataset

### Fingerprinting stage

At this stage,  we have trained a fingerprint generator by utilizing the code and dataset provided in the paper ["Neural Audio Fingerprint for High-specific Audio Retrieval based on Contrasive Learning."](https://arxiz.org/abs/2010.11910) 

 For further information regarding the implementation of this step, you can refer to the link https://mimbres.github.io/neural-audio-fp.

### Embedding stage

The dataset utilized for this stage consists  of 1 million fingerprint features, derived from 100,000 audio clips, each 30 seconds in length. These clips are a subset of the fma_large dataset. For each audio clip, 10 distinct fingerprint features are generated. One feature directly represents the original clip, while the remaining nine are extracted after applying various data augmentation techniques to the original audio. This approach ensures the diversity of the dataset. 

The  data naming format is shown below:

```
# "songId-clipId", where clipId is '00', incicates that this feature is generated by an original audio clip.
data/train/data-10w/1-00.npy
data/train/data-10w/1-01.npy
······
data/train/data-10w/100000-09.npy
```

We publish a dataset generated from 1K 30s audio clips with which you can try to complete the training. Download data-1k.zip from https://drive.google.com/file/d/1uR4JRk1jkYyQhxk3puTnp3wrHsAPqwdu/view?usp=drive_link and unzip to `./data/data-1k`.

### Structure of dataset

```
  dataset/
  ├── data
      └── data-1k     <=== 1k original clips' features and 9k augmented clips' features
```

## Train

```
python train.py --checkpoint_name CHECKPOINT_NAME
```

## Generate fingerprint

Before the evaluate step, you need to split the raw fingerprints according to the duration_max you use in the training process. You can create a Python file and execute the following code:

```
from utils import split_database

# max_len = 2 * duration_max - 1, for a 30s clip, max_len = 59.
split_database('database/dummy_db/src_dir', 'database/dummy_db/dst_dir', max_len)
split_database('database/db/src_test_db_dir', 'database/db/dst_test_db_dir', max_len)
```

 ## Evaluate

```
python evaluate.py --checkpoint_path CHECKPOINT_PATH
```

## Build DB & Search

![image-20240314164854170](https://github.com/syh200626/AMG-Embedding/assets/83171204/80242c87-3139-4b6c-83c5-76379b28c715)

This image describes how to create a search database and complete the search. The set of ground truth IDs for q[i] will be (i + len(dummy_db)).

### Structure of database

```
  database/
  ├── dummy_db
  │   ├── fma_part     <=== subset of fma_full, including features generated from 10K audio clips
  │   ├── fma_part_5s  <=== split the feature sequences into 9 in length per file
  │   ├── fma_part_10s <=== split the feature sequences into 19 in length per file
  │   ├── fma_part_15s <=== split the feature sequences into 29 in length per file
  │   └── fma_part_30s <=== split the feature sequences into 59 in length per file
  ├── db
  │   ├── test_5s     <=== split the feature sequences into 9 in length per file
  │   ├── test_10s     <=== split the feature sequences into 19 in length per file
  │   ├── test_15s     <=== split the feature sequences into 29 in length per file
  │   └── test_30s     <=== 500 songs (30s) for testing, generated from raw audio, corresponds to query
  └── query    <=== 500 songs(30s) for testing, generated from raw audio after data augmentation
      
data format:embeddings extracted by fingerprinting stage
```
