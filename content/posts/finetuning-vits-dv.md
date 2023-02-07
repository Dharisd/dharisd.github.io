---
title: "Finetuning VITS on Common voice Dhivehi"
date: 2023-02-07T08:22:22+05:00
draft: False
---

Text to speech has been something that interested and involved me for sometime, And the last publically available models are from 2021 era, Since then there has been lots of improvements in the area .

I decided to cook up some and see how it goes 

The model we are gonna be training the is the VITS implementaion from [coqui](https://coqui.ai/) (check them out they are the coolest fr)

## Whats VITS and why VITS out of all

VITS (Conditional Variational Autoencoder with Adversarial Learning for End-to-End Text-to-Speech) is a autoregressive end to end model, while most other architectures such as tacotron rely on a two-stage method where a mel spectogram is first generated, which is then converted to speech using a vocoder , VITS uses a single model architecture which takes in text and returns the raw audio waveforms. 

* **Single model architecture**
The model does not need to generate immediate representations and then load them again to another model for inference, this geatly reduces full end to end inference speed, this also helps to make deploynet easier

* **Multispeaker Support**
Supports training and infernce of audio for multi speakers from a single model throigh speaker embeddings.

* **Tooling**
[Coqui/TTS](https://github.com/coqui-ai/TTS) have done an absolutely amazing job, developing the tooling and the tools to train TTS models
 

### Setting up the dataset 

Setting up the dataset would have been pretty straight forward since i was using the common voice dataset, all I needed to do was to remove the voices and entries of the voices i didnt want to train on

**Why remove some voices?**

The Common Voice dataset isnt and wasnt meant to be used for Text to speech purposes as it is more or less focused on Speech to text applications, as such the voices necessasrily dont focus on sounding like TTS and most speakers sound like they are talking casually to their parents or reading out something 

**What do?**

Although most sound like this I noticed that some speakers infact actually have recorded with good setups and put care into sounding the best they can 

I isolated the best sounding speakers with the most audio (5 of them) each with atleast 3 hoiurs of audio to seperate dataset and then used this for training, This dataset was about x hours long 

```python
#the client_id for the 5 speakers which sounded the best to me and had enough data

speakers = ["MCV_d0c4ff6121bfdbbf162900de8e68c2f5ea1e1d08391e4928e9f2febf82869a5aae10cb3e9d3f6b77487aad40da413e27ebf451dd980c01937dd7476c01df330a",
"MCV_e3de19abdf1e78628e9d0b9eceb1022db3270200e46cc30440f7318eef8c93e8fa2f52baab585edbdbb55bd14edcf67dab85d0af8248d3b0a58ea68ffbf421d8",
"MCV_0a15c5b1210dabdccec22077ac63c2b2833c170baf61e7b81a7aad821a97f4415afbd00ae54aa633c7f691eed09683948f27e7ffe6416411b3e4b6ceffaff16d",
"MCV_52bc327bd5bde02bfe73b0321673eaf6734ad583270c0d316a9409b779f3ee3817c55744164fe9925c70e29ce3fd6c2f322f856cb256957e27eb8345c9dcb62b",
"MCV_f5a79d21b4ee20e7008f784477fae663361e7cb5d1f06ea88a7d246502364c6c1b1d273db9334ba1feb1ee712e349c9a50425c52345c95a3d2d2b5c9d59ee6b4"
]
```


**Dataset done right?**
Just set Common voice as formatter and we move right? right?
```python
dataset_config = BaseDatasetConfig(
formatter="common_voice", meta_file_train="phonemise_fixed.tsv", path=os.path.join(output_path, "dv_filtered/")
)
```

**NO** 
This just hanged infinetly when i tried setting up the Trainer
**but why?**
Turns out the formatter wasnt updated for the latest dataset format and was hanging due to an infinite loop

after I fixed this bug , dataset was done 


## LETS TRAIN LETS GO BRO 

Initially I trained from scratch but the performance at 20k steps was literally this 

*insert this* 

so I decided to pull my favoruite trick, finetune on top of an english model 

**just pull out the model** 
```tts --model_name tts_models/en/vctk/vits --text "this should be the epitome" --speaker_idx "p341"```

**copy to good location**

```!cp -r "/root/.local/share/tts" "~/tts"```

**setup Tensorboard for the cool visuals and important stats**
![[tensorboard]](https://raw.githubusercontent.com/Dharisd/dharisd.github.io/main/assets/images/tensorboard.png)

**Set it as the the restore path**

```bash
python train.py --restore_path "~/tts/vctk_model.pth" --config_path "path_to_your_config_file"
```


yes , now we train, 

*LET IT COOK*
![[walter]](https://raw.githubusercontent.com/Dharisd/dharisd.github.io/main/assets/images/hard.jpeg)
At around 40k steps we have usable results

![](https://raw.githubusercontent.com/Dharisd/dharisd.github.io/main/assets/audio/madness_0.wav)
![](https://raw.githubusercontent.com/Dharisd/dharisd.github.io/main/assets/audio/madness_1.wav)
![](https://raw.githubusercontent.com/Dharisd/dharisd.github.io/main/assets/audio/madness_2.wav)
![](https://raw.githubusercontent.com/Dharisd/dharisd.github.io/main/assets/audio/madness_3.wav)
![](https://raw.githubusercontent.com/Dharisd/dharisd.github.io/main/assets/audio/madness_4.wav)

## How do i generate my own audio?

First install the TTS library 
```pip install tts```

Download the models from [here](https://drive.google.com/drive/folders/1OGVBHlttIjkRyKyxRklv-0CE8vACO3H3?usp=sharing)

```bash
tts --text "މިހާރު ވާހަކަ ދެއްކޭނެ ވަރަށް ރަނގަޅު ވަރަކަށް" \
    --model_path "checkpoint_1020000.pth"\
    --config_path "config.json" \
    --speaker_idx "MCV_f5a79d21b4ee20e7008f784477fae663361e7cb5d1f06ea88a7d246502364c6c1b1d273db9334ba1feb1ee712e349c9a50425c52345c95a3d2d2b5c9d59ee6b4" \
    --out_path output.wav
```


or you can use my synth [notebook](https://colab.research.google.com/drive/1TMLBAcr-T9dSfItwvl6e_svl0FWi_JZn)


### Limitaions

* Noticable background noise in some voices, this is manily due to ground truth also having noise 
* Pronounciation and tone are not as good as the TransformerTTS models from prior
* The models are large at around 1 GB

### How to improve 

* Way better data needed
The models are only good as the data i give it, if the data is noisy the output is going to be noisy 

* Look into phonemiser 
Could reduce complexity for the model and reduce the model size

* More data 
model did not improve much after 30k steps, could be due to lack of data 
