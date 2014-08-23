TinySR: Tiny Speech Recognizer
==============================

**TinySR** is a light weight real-time small-vocabulary speaker-independent speech recognizer written in portable C.
The entire library fits in a single pair of files, `tinysr.c` and `tinysr.h`.
You can either statically link, dynamically link, or directly include the pair of files in your project, as TinySR is entirely public domain.
No special libraries are used or dependencies required; everything is here.

The goal is to provide the following features:
* Lean, readable source with absolutely no external dependencies.
* Very easy API for both real-time and one-shot recognition.
* Real-time low latency performance, and utterance detection.
* User friendly vocabulary training, and good speaker independence.

Currently the recognition pipeline looks like:
* Generic audio processing (resampling filter, bringing input to 16 kHz)
* ES 201 108 feature extraction, to log energy + 13 cepstral components.
* Utterance detection, followed by utterance level Cepstral Mean Normalization.
* Maximum likelihood multivariate Gaussian models.
* Dynamic Time Warping to match against the vocabulary.

If you use my code, I'd love it if you dropped me a line at <snp@mit.edu>.
There's a LaTeX file in `docs` describing how TinySR works, and the compiled PDF is here: http://www.mit.edu/~snp/tinysr.pdf

The code is divided into the following directories:
* `apps`: Contains programs that link against `tinysr.o`. The makefile is set up to automatically compile anything matching `*.c` in `apps` against `tinysr.o`.
* `playground`: Contains non-critical throw-away programs that were written in the course of creating TinySR.
* `scripts`: Contains utility scripts, such as for speaker training, or computing important tables of constants.

To Train
--------

Before you can start training, you are going to need to produce a bunch of raw 16-bit 16 kHz mono audio.
On systems supporting ALSA, I recommend the following command to produce such raw audio, and stream it to stdout:

	arecord -r 16000 -c 1 -f S16_LE

We will assume you have decided upon such a command, and will refer to it as `<audio>`.
To build a speech model for TinySR, begin by deciding on your vocabulary.
Once you've decided, make a directory for each word.
In this example, ``up'' and ``down'' are the vocabulary.
First, run:

	<audio> | ./apps/store_utters.app data/up

Say the word ``up'' approximately one hundred times, with long enough spaces in between that the app doesn't get confused.
Naturally, repeat this process for ``down'' into a separate directory.
Once you're done with this, run:

	python scripts/model_gen.py data/up up_model

And again, the same for ``down.''
To produce an overall speech model, you need merely concatenate the files for the individual words in the vocabulary.
Thus:

	cat up_model down_model > speech_model

You can now test the recognizer on your model by running:

	<audio> | ./apps/full_reco.app speech_model

