# tts-arabic-tacotron2

Tacotron2 from `torchaudio`, trained on Nawar Halabi's [Arabic Speech Corpus](http://en.arabicspeechcorpus.com/), including the [HiFi-GAN vocoder](https://github.com/jik876/hifi-gan) for direct TTS inference.

![tts](https://user-images.githubusercontent.com/28433296/178712707-264d1310-4162-4f34-b336-56e6be944e2d.png)

Papers:

Tacotron2 | Natural TTS Synthesis by Conditioning WaveNet on Mel Spectrogram Predictions ([arXiv](https://arxiv.org/abs/1712.05884))

HiFi-GAN  | HiFi-GAN: Generative Adversarial Networks for Efficient and High Fidelity Speech Synthesis ([arXiv](https://arxiv.org/abs/2010.05646))

## Audio Samples

You can listen to some audio samples [here](https://nipponjo.github.io/tacotron2-arabic-samples).

## Quick Setup
Required packages:
`torch torchaudio pyyaml`

~ for training: `librosa matplotlib tensorboard`

~ for the demo app: `fastapi "uvicorn[standard]"`

Download the pretrained weights for the Tacotron2 model ([link](https://drive.google.com/u/1/uc?id=18a20eVu0bLlws7h3TA0xZse68aSr7Bg9&export=download)).

Download the [HiFi-GAN vocoder](https://github.com/jik876/hifi-gan) weights and config file ([direct link](https://drive.google.com/drive/folders/1YuOoV3lO2-Hhn1F2HJ2aQ4S0LC1JdKLd)). Either put them into `pretrained/hifigan-universal-v1` or edit the following lines in `configs/basic.yaml`.

```yaml
# vocoder
vocoder_state_path: pretrained/hifigan-universal-v1/g_02500000
vocoder_config_path: pretrained/hifigan-universal-v1/config.json
```

## Using the model

This repo uses the [Tacotron2](https://pytorch.org/audio/stable/models.html#tacotron2) model from `torchaudio.models`. The `Tacotron2` from `model.networks` wraps the model in order to simplify text-to-mel inference. The `Tacotron2Wave` model includes the [HiFi-GAN vocoder](https://github.com/jik876/hifi-gan) for direct text-to-speech inference.

## Inferring the Mel spectrogram

```python
from model.networks import Tacotron2
model = Tacotron2('pretrained/tacotron2_ar.pth')
model = model.cuda()
mel_spec = model.ttmel("???????????????????? ???????????????? ?????? ????????????????")
```

## End-to-end Text-to-Speech



```python
from model.networks import Tacotron2Wave
model = Tacotron2Wave('pretrained/tacotron2_ar.pth')
model = model.cuda()
wave = model.tts("???????????????????? ???????????????? ?????? ????????????????")

wave_list = model.tts(["????????" ,"??????????" ,"????????????", "??????????????" ,"????????????????" ,"????????????", "????????????" ,"????????????" ,"??????????????????", "????????????" ,"??????????????"])
```

By default, Arabic letters are converted using the [Buckwalter transliteration](https://en.wikipedia.org/wiki/Buckwalter_transliteration). The transliteration can also be used directly. If no Arabic script is expected to be used you can set `arabic_in=False`.

```python
model = Tacotron2Wave('pretrained/tacotron2_ar.pth')
wave = model.tts(">als~alAmu Ealaykum yA Sadiyqiy")


model = Tacotron2Wave('pretrained/tacotron2_ar.pth', arabic_in=False)
wave = model.tts(">als~alAmu Ealaykum yA Sadiyqiy")

wave_list = model.tts(["Sifr", "wAHid", "<i^nAn", "^alA^ap", ">arbaEap", "xamsap", "sit~ap", "sabEap", "^amAniyap", "tisEap", "Ea$arap"])
```

### Inference from text file
```bash
python inference.py
# default parameters:
python inference.py --list data/infer_text.txt --checkpoint pretrained/tacotron2_ar.pth --out_dir samples/results --batch_size 8 --speed 1
```

## Testing the model
To test the model run:
```bash
python test.py
# default parameters:
python test.py --checkpoint pretrained/tacotron2_ar.pth --out_dir samples/test
```

## Processing details
This repo uses Nawar Halabi's [Arabic-Phonetiser](https://github.com/nawarhalabi/Arabic-Phonetiser) but simplifies the result such that different contexts are ignored (see `text/symbols.py`). Further, a doubled consonant is represented as consonant + doubling-token.

The model can sometimes struggle to pronounce the last phoneme of a sentence when it ends in an unvocalized consonant. The pronunciation is more reliable if one appends a word-separator token at the end and cuts it off using the alignments weights (details in `models.networks`). This option is implemented as a default postprocessing step that can be disabled by setting `postprocess_mel=False`.


## Training the model
Before training, the audio files must be resampled. The pretrained model was trained after preprocessing the files using `scripts/preprocess_audio.py`.

To train the model with options specified in the config file run:
```bash
python train.py
# default parameters:
python train.py --config configs/nawar.yaml
```


## Web app

The web app uses the FastAPI library. To run the app you need the following packages:

fastapi: for the backend api | uvicorn: for serving the app

Install with: `pip install fastapi "uvicorn[standard]"`

Run with: `python app.py`

Preview:

<div align="center">
  <img src="https://user-images.githubusercontent.com/28433296/178459733-6b6610df-9ad3-4d9a-8e5f-1282956a65ec.png" width="66%"></img>
</div>

## Acknowledgements

I referred to NVIDIA's [Tacotron2 implementation](https://github.com/NVIDIA/tacotron2) for details on model training. 

