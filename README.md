# TorToiSe

<h3> How to run locally </h3>
<br>
First go to Experiment.ipynb then select voice what type of voice you want to use for tts , you can give your custom voice by inserting your voice in ->tortoise->voice->sample
<br>
then you can type the text what you want to process.<br>
Tortoise is a text-to-speech program built with the following priorities:

1. Strong multi-voice capabilities.
2. Highly realistic prosody and intonation.
   
This repo contains all the code needed to run Tortoise TTS in inference mode.


### Adding a new voice

To add new voices to Tortoise, you will need to do the following:

1. Gather audio clips of your speaker(s). Good sources are YouTube interviews (you can use youtube-dl to fetch the audio), audiobooks or podcasts. Guidelines for good clips are in the next section.
2. Cut your clips into ~10 second segments. You want at least 3 clips. More is better, but I only experimented with up to 5 in my testing.
3. Save the clips as a WAV file with floating point format and a 22,050 sample rate.
4. Create a subdirectory in voices/
5. Put your clips in that subdirectory.
6. Run tortoise utilities with --voice=<your_subdirectory_name>.

### Picking good reference clips

As mentioned above, your reference clips have a profound impact on the output of Tortoise. Following are some tips for picking
good clips:

1. Avoid clips with background music, noise or reverb. These clips were removed from the training dataset. Tortoise is unlikely to do well with them.
2. Avoid speeches. These generally have distortion caused by the amplification system.
3. Avoid clips from phone calls.
4. Avoid clips that have excessive stuttering, stammering or words like "uh" or "like" in them.
5. Try to find clips that are spoken in such a way as you wish your output to sound like. For example, if you want to hear your target voice read an audiobook, try to find clips of them reading a book.
6. The text being spoken in the clips does not matter, but diverse text does seem to perform better.

## Advanced Usage

### Generation settings

Tortoise is primarily an autoregressive decoder model combined with a diffusion model. Both of these have a lot of knobs
that can be turned that I've abstracted away for the sake of ease of use. I did this by generating thousands of clips using
various permutations of the settings and using a metric for voice realism and intelligibility to measure their effects. I've
set the defaults to the best overall settings I was able to find. For specific use-cases, it might be effective to play with
these settings (and it's very likely that I missed something!)

These settings are not available in the normal scripts packaged with Tortoise. They are available, however, in the API. See
```api.tts``` for a full list.

### Prompt engineering

Some people have discovered that it is possible to do prompt engineering with Tortoise! For example, you can evoke emotion
by including things like "I am really sad," before your text. I've built an automated redaction system that you can use to
take advantage of this. It works by attempting to redact any text in the prompt surrounded by brackets. For example, the
prompt "\[I am really sad,\] Please feed me." will only speak the words "Please feed me" (with a sad tonality).

### Playing with the voice latent

Tortoise ingests reference clips by feeding them through individually through a small submodel that produces a point latent,
then taking the mean of all of the produced latents. The experimentation I have done has indicated that these point latents
are quite expressive, affecting everything from tone to speaking rate to speech abnormalities.

This lends itself to some neat tricks. For example, you can combine feed two different voices to tortoise and it will output
what it thinks the "average" of those two voices sounds like.

#### Generating conditioning latents from voices

Use the script `get_conditioning_latents.py` to extract conditioning latents for a voice you have installed. This script
will dump the latents to a .pth pickle file. The file will contain a single tuple, (autoregressive_latent, diffusion_latent).

Alternatively, use the api.TextToSpeech.get_conditioning_latents() to fetch the latents.

#### Using raw conditioning latents to generate speech

After you've played with them, you can use them to generate speech by creating a subdirectory in voices/ with a single
".pth" file containing the pickled conditioning latents as a tuple (autoregressive_latent, diffusion_latent).

## Tortoise-detect

Out of concerns that this model might be misused, I've built a classifier that tells the likelihood that an audio clip
came from Tortoise.

This classifier can be run on any computer, usage is as follows:

```commandline
python tortoise/is_this_from_tortoise.py --clip=<path_to_suspicious_audio_file>
```

This model has 100% accuracy on the contents of the results/ and voices/ folders in this repo. Still, treat this classifier
as a "strong signal". Classifiers can be fooled and it is likewise not impossible for this classifier to exhibit false
positives.

## Model architecture

Tortoise TTS is inspired by OpenAI's DALLE, applied to speech data and using a better decoder. It is made up of 5 separate
models that work together. I've assembled a write-up of the system architecture here:
[https://nonint.com/2022/04/25/tortoise-architectural-design-doc/](https://nonint.com/2022/04/25/tortoise-architectural-design-doc/)



## Notice

Tortoise was built entirely by me using my own hardware. My employer was not involved in any facet of Tortoise's development.

If you use this repo or the ideas therein for your research, please cite it! A bibtex entree can be found in the right pane on GitHub.
