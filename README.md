# A simple FastAPI Server to run XTTSv2

This project is inspired by [daswer123](https://github.com/daswer123/xtts-api-server) and utilizes [XTTSv2](https://github.com/coqui-ai/TTS).

This server was created for [Mantella](https://github.com/art-from-the-machine/Mantella) by [Haurrus](https://github.com/Haurrus) and I made it compatible with Blackwell GPUs by updating Pytorch libraries.

Deepspeed and low VRAM modes might not work.

Please note that I'm a total beginner in Python and will probably have trouble answering issues. Feel free to help if you're more experienced than I am.

## If you want to train you own dataset :

You can use the webui for fine-tuning xtts from [webui for fine-tuning xtts](https://github.com/daswer123/xtts-webui)
Or my work to train in batch using only the command line : [xtts-trainer-no-ui-auto](https://github.com/Haurrus/xtts-trainer-no-ui-auto)


## Installation

I'm using python 3.11.7

### Manual
```bash
# Clone REPO
git clone https://github.com/leosdt/xtts-api-server-mantella-blackwell/
cd xtts-api-server-mantella-blackwell
# Create virtual env
python -m venv venv
venv/scripts/activate or source venv/bin/activate
# Install deps
pip install -r requirements.txt
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu128
# Launch server
python -m xtts_api_server
 
```
## Mantella specific

Don't forget to add the folder \plugins\lip_fuz in the root folder so Mantella can find it when you pass the argument in config.ini (xtts_server_folder)

## Starting Server

`python -m xtts_api_server` will run on default ip and port (localhost:8020)

Use the `--deepspeed` flag to process the result fast ( 2-3x acceleration )

```
usage: xtts_api_server [-h] [-hs HOST] [-p PORT] [-sf SPEAKER_FOLDER] [-o OUTPUT] [-t TUNNEL_URL] [-ms MODEL_SOURCE] [--listen] [--use-cache] [--lowvram] [--deepspeed] [--streaming-mode] [--stream-play-sync]

Run XTTSv2 within a FastAPI application

options:
  -h, --help show this help message and exit
  -hs HOST, --host HOST
  -p PORT, --port PORT
  -d DEVICE, --device DEVICE `cpu` or `cuda`, you can specify which video card to use, for example, `cuda:0`
  -sf SPEAKER_FOLDER, --speaker-folder The folder where you get the samples for tts
  -o OUTPUT, --output Output folder
  -mf MODELS_FOLDERS, --model-folder Folder where models for XTTS will be stored, finetuned models should be stored in this folder
  -t TUNNEL_URL, --tunnel URL of tunnel used (e.g: ngrok, localtunnel)
  -ms MODEL_SOURCE, --model-source ["api","apiManual","local"]
  -v MODEL_VERSION, --version You can download the official model or your own model, official version you can find [here](https://huggingface.co/coqui/XTTS-v2/tree/main)  the model version name is the same as the branch name [v2.0.2,v2.0.3, main] etc. Or you can load your model, just put model in models folder
  --listen Allows the server to be used outside the local computer, similar to -hs 0.0.0.0
  --use-cache Enables caching of results, your results will be saved and if there will be a repeated request, you will get a file instead of generation
  --lowvram The mode in which the model will be stored in RAM and when the processing will move to VRAM, the difference in speed is small
  --deepspeed allows you to speed up processing by several times, automatically downloads the necessary libraries
  --streaming-mode Enables streaming mode, currently has certain limitations, as described below.
  --streaming-mode-improve Enables streaming mode, includes an improved streaming mode that consumes 2gb more VRAM and uses a better tokenizer and more context.
  --stream-play-sync Additional flag for streaming mod that allows you to play all audio one at a time without interruption
```

You can specify the path to the file as text, then the path counts and the file will be voiced

You can load your own model, for this you need to create a folder in models and load the model with configs, note in the folder should be 3 files `config.json` `vocab.json` `model.pth`

If you want your host to listen, use -hs 0.0.0.0 or use --listen

The -t or --tunnel flag is needed so that when you get speakers via get you get the correct link to hear the preview. More info [here](https://imgur.com/a/MvpFT59)

Model-source defines in which format you want to use xtts:

1. `local` - loads version 2.0.2 by default, but you can specify the version via the -v flag, model saves into the models folder and uses `XttsConfig` and `inference`.
2. `apiManual` - loads version 2.0.2 by default, but you can specify the version via the -v flag, model saves into the models folder and uses the `tts_to_file` function from the TTS api
3. `api` - will load the latest version of the model. The -v flag won't work.

All versions of the XTTSv2 model can be found [here](https://huggingface.co/coqui/XTTS-v2/tree/main)  the model version name is the same as the branch name [v2.0.2,v2.0.3, main] etc.

The first time you run or generate, you may need to confirm that you agree to use XTTS.

# About Streaming mode

Streaming mode allows you to get audio and play it back almost immediately. However, it has a number of limitations.

You can see how this mode works [here](https://www.youtube.com/watch?v=jHylNGQDDA0) and [here](https://www.youtube.com/watch?v=6vhrxuWcV3U)

Now, about the limitations

1. Can only be used on a local computer
2. Playing audio from the your pc
3. Does not work endpoint `tts_to_file` only `tts_to_audio` and it returns 1 second of silence.

You can specify the version of the XTTS model by using the `-v` flag.

Improved streaming mode is suitable for complex languages such as Chinese, Japanese, Hindi or if you want the language engine to take more information into account when processing speech.

`--stream-play-sync` flag - Allows you to play all messages in queue order, useful if you use group chats. In SillyTavern you need to turn off streaming to work correctly

# API Docs

API Docs can be accessed from [http://localhost:8020/docs](http://localhost:8020/docs)

# How to add speaker

By default the `speakers` folder should appear in the folder, you need to put there the wav file with the voice sample, you can also create a folder and put there several voice samples, this will give more accurate results

# Selecting Folder

You can change the folders for speakers and the folder for output via the API.


# Use Docker image with Docker Compose

A Dockerfile is provided to build a Docker image, and a docker-compose.yml file is provided to run the server with Docker Compose as a service.

You can build the image with the following command:

```bash
mkdir xtts-api-server
cd xtts-api-server
docker run -d daswer123/xtts-api-server

```

OR

```bash
cd docker
docker compose build
```

Then you can run the server with the following command:

```bash
docker compose up # or with -d to run in background
```

# Note on creating samples for quality voice cloning

The following post is a quote by user [Material1276 from reddit](https://www.reddit.com/r/Oobabooga/comments/1807tsl/comment/ka5l8w9/?share_id=_5hh4KJTXrEOSP0hR0hCK&utm_content=2&utm_medium=android_app&utm_name=androidcss&utm_source=share&utm_term=1)

> Some suggestions on making good samples
>
> Keep them about 7-9 seconds long. Longer isn't necessarily better.
>
> Make sure the audio is down sampled to a Mono, 22050Hz 16 Bit wav file. You will slow down processing by a large % and it seems cause poor quality results otherwise (based on a few tests). 24000Hz is the quality it outputs at anyway!
>
> Using the latest version of Audacity, select your clip and Tracks > Resample to 22050Hz, then Tracks > Mix > Stereo to Mono. and then File > Export Audio, saving it as a WAV of 22050Hz
>
> If you need to do any audio cleaning, do it before you compress it down to the above settings (Mono, 22050Hz, 16 Bit).
>
> Ensure the clip you use doesn't have background noises or music on e.g. lots of movies have quiet music when many of the actors are talking. Bad quality audio will have hiss that needs clearing up. The AI will pick this up, even if we don't, and to some degree, use it in the simulated voice to some extent, so clean audio is key!
>
> Try make your clip one of nice flowing speech, like the included example files. No big pauses, gaps or other sounds. Preferably one that the person you are trying to copy will show a little vocal range. Example files are in [here](https://github.com/oobabooga/text-generation-webui/tree/main/extensions/coqui_tts/voices)
>
> Make sure the clip doesn't start or end with breathy sounds (breathing in/out etc).
>
> Using AI generated audio clips may introduce unwanted sounds as its already a copy/simulation of a voice, though, this would need testing.

# Credit

1. Thanks to **daswer123** [daswer123](https://github.com/daswer123/xtts-api-server)
2. Thanks to the author **Kolja Beigel** for the repository [RealtimeTTS](https://github.com/KoljaB/RealtimeTTS) , I took some of its code for my project.
3. Thanks **[erew123](https://github.com/oobabooga/text-generation-webui/issues/4712#issuecomment-1825593734)** for the note about creating samples and the code to download the models
4. Thanks **lendot** for helping to fix the multiprocessing bug and adding code to use multiple samples for speakers
