
# Deep Speech

## Install

```bash
sudo apt install portaudio19-dev
pip3 install pyaudio
pip3 install deepspeech-gpu
```

* Model:

```bash
curl -LO https://github.com/mozilla/DeepSpeech/releases/download/v0.9.3/deepspeech-0.9.3-models.pbmm
curl -LO https://github.com/mozilla/DeepSpeech/releases/download/v0.9.3/deepspeech-0.9.3-models.scorer
```

## Usage

```python
#!/usr/env python3
import time

import deepspeech
import numpy as np
import pyaudio

model_file = './deepspeech-0.9.3-models.pbmm'
model = deepspeech.Model(model_file)
model.setScorerAlphaBeta(0.75, 1.85)
model.setBeamWidth(500)
model.enableExternalScorer('./deepspeech-0.9.3-models.scorer')

stream = model.createStream()

prev_text = ''
def process_audio(in_data, frame_count, time_info, status):
    global prev_text
    data16 = np.frombuffer(in_data, dtype=np.int16)
    stream.feedAudioContent(data16)
    text = stream.intermediateDecode()
    if text != prev_text:
        print('\rTranscript = {}'.format(text))
        prev_text = text
    return (in_data, pyaudio.paContinue)

audio = pyaudio.PyAudio()
audio_stream = audio.open(
    format=pyaudio.paInt16,
    channels=1,
    rate=16000,
    input=True,
    frames_per_buffer=1024,
    stream_callback=process_audio
)

try:
    while audio_stream.is_active():
        time.sleep(0.1)
except KeyboardInterrupt:
    pass

audio_stream.stop_stream()
audio_stream.close()
audio.terminate()
text = stream.finishStream()
print('Final text = {}'.format(text))
```
