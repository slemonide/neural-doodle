Neural Doodle
=============

.. image:: docs/Workflow.gif
    :align: right

Use a deep neural network to borrow the skills of real artists and turn your two-bit doodles into masterpieces! This project is an implementation of `Semantic Style Transfer <http://arxiv.org/abs/1603.01768>`_ (Champandard, 2016), based on the `Neural Patches <http://arxiv.org/abs/1601.04589>`_ algorithm (Li, 2016).

The ``doodle.py`` script generates an image by using three or four images as inputs: the original style and its annotation, and a target content image (optional) with its annotation (a.k.a. your doodle). The algorithm then extracts annotated patches from the style image, and incrementally transfers them over to the target image based on how closely they match.

1. `Examples & Usage <#examples--usage>`_
2. `Installation <#installation-setup>`_
3. `Troubleshooting <#troubleshooting-problems>`_
4. `Frequent Questions <#frequent-questions>`_

**NOTE**: This project is possible thanks to the `nucl.ai Conference <http://nucl.ai/>`_ on **July 18-20**. Join us in **Vienna**!

|Python Version| |License Type| |Project Stars|

----

.. image:: docs/Landscape_example.png

Examples & Usage
================

Image Analogy
-------------

The algorithm is built for style transfer, but can also generate image analogies that we call a ``#NeuralDoodle``; use the hashtag if you post your images!  Example files are included in the ``#/samples/`` folder. Execute with these commands:

.. code:: bash

    # Synthesize a coastline as if painted by Monet. This uses "*_sem.png" masks for both images.
    python3 doodle.py --style samples/Monet.jpg --output samples/Coastline.png \
                      --device=cpu --iterations=40

    # Generate a scene around a lake in the style of a Renoir painting.
    python3 doodle.py --style samples/Renoir.jpg --output samples/Landscape.png \
                      --device=gpu0 --iterations=80

Note the ``--device`` argument that lets you specify which GPU or CPU to use. For the samples above, here are the performance results:

* **GPU Rendering** — Assuming you have CUDA and enough on-board RAM, the process should complete in less than 10 minutes, even with twice the iterations.
* **CPU Rendering** — This will take hours and hours, even up to 12h on older hardware. To match quality it'd take twice the time. Do multiple runs in parallel!

The default is to use ``cpu``, if you have NVIDIA card setup with CUDA already try ``gpu0``. On the CPU, you can also set environment variable to ``OMP_NUM_THREADS=4``, but we've found the speed improvements to be minimal.


Installation & Setup
====================

This project requires Python 3.4+ and you'll also need ``numpy`` and ``scipy`` (numerical computing libraries) as well as ``python3-dev`` installed system-wide. Afterward fetching the repository, you can run the following commands from your terminal to setup a local environment:

.. code:: bash

    # Create a local environment for Python 3.x to install dependencies here.
    python3 -m venv pyvenv --system-site-packages

    # If you're using bash, make this the active version of Python.
    source pyvenv/bin/activate

    # Setup the required dependencies simply using the PIP module.
    python3 -m pip install --ignore-installed -r requirements.txt

After this, you should have ``scikit-image``, ``theano`` and ``lasagne`` installed in your virtual environment.  You'll also need to download this `pre-trained neural network <https://github.com/alexjc/neural-doodle/releases/download/v0.0/vgg19_conv.pkl.bz2>`_ (VGG19, 80Mb) for the script to run. Once you're done you can just delete the ``#/pyvenv/`` folder.

.. image:: docs/Coastline_example.png


Troubleshooting Problems
========================

It's running out of GPU Ram, throwing ``MemoryError``. Help!
------------------------------------------------------------

You'll need a good NVIDIA card with CUDA to run this software on GPU, ideally 2Gb / 4Gb or better still, 8Gb to 12Gb for larger resolutions.  The code does work on CPU by default, so use that as fallback since you likely have more system RAM!

To improve memory consumption, you can also install NVIDIA's ``cudnn`` library version 3.0 or 4.0. This allows convolutional neural networks to run faster and save space in GPU RAM.

**FIX** Use ``--device=cpu`` to use main system memory.


NotImplementedError: AbstractConv2d theano optimization failed.
---------------------------------------------------------------

This happens when you're running without a GPU, and the CPU libraries were not found (e.g. ``libblas``).  The neural network expressions cannot be evaluated by Theano and it's raising an exception.

**FIX** ``sudo apt-get install libblas-dev libopenblas-dev``


TypeError: max_pool_2d() got an unexpected keyword argument 'mode'
------------------------------------------------------------------

You need to install Lasagne and Theano directly from the versions specified in ``requirements.txt``, rather than from the PIP versions.  These alternatives are older and don't have the required features.

**FIX** ``python3 -m pip install -r requirements.txt``


ValueError: unknown locale: UTF-8
---------------------------------

It seems your terminal is misconfigured and not compatible with the way Python treats locales. You may need to change this in your ``.bash_rc`` or other startup script. Alternatively, this command will fix it once for this shell instance.

**FIX** ``export LC_ALL=en_US.UTF-8``


ERROR: The optimization diverged and NaN numbers were encountered.
------------------------------------------------------------------

It's possible there's a platform bug in the underlying libraries or compiler, which has been reported on MacOS El Capitan.  It's not clear how to fix it, but you can try to disable optimizations to prevent the bug. (See `Issue #8 <https://github.com/alexjc/neural-doodle/issues/8>`_.)

**FIX** Use ``--safe-mode`` flag to disable optimizations.


Frequent Questions
==================

Q: How is semantic style transfer different to neural analogies?
----------------------------------------------------------------

It's still too early to say definitively, both approaches were discovered independently in 2016 by `@alexjc <https://twitter.com/alexjc>`_ and `@awentzonline <https://twitter.com/awentzonline>`_ (respectively). Here are some early impressions:

1. One algorithm is style transfer that happens to do analogies, and the other is analogies that happens to do style transfer now. Adam extended his implementation to use a content loss after the `Semantic Style Transfer <http://arxiv.org/abs/1603.01768>`_ paper was published, so now they're even more similar under the hood!

2. Both use a `patch-based approach <http://arxiv.org/abs/1601.04589>`_ (Li, 2016) but semantic style transfer imposes a "prior" via the patch-selection process and neural analogies has an additional prior on the convolution activations.  The outputs for both algorithms are a little different, it's not yet clear where each one is best.

3. Semantic style transfer is simpler, it has fewer loss components.  This means somewhat less code to write and there are **fewer parameters involved** (not necessarily positive or negative).  Neural analogies is a little more complex, with as many parameters as the combination of two algorithms.

4. Neural analogies is designed to work with images, and can only support the RGB format for its masks. Semantic style transfer was designed to **integrate with other neural networks** (for pixel labeling and semantic segmentation), and can use any format for its maps, including RGBA or many channels per label masks.

5. Semantic style transfer is **about 25% faster and uses less memory** too.  For neural analogies, the extra computation is effectively the analogy prior — which could improve the quality of the results in theory. In practice, it's hard to tell at this stage and more testing is needed.

If you have any comparisons or insights, be sure to let us know!

----

|Python Version| |License Type| |Project Stars|

.. |Python Version| image:: http://aigamedev.github.io/scikit-neuralnetwork/badge_python.svg
    :target: https://www.python.org/

.. |License Type| image:: https://img.shields.io/badge/license-New%20BSD-blue.svg
    :target: https://github.com/alexjc/neural-doodle/blob/master/LICENSE

.. |Project Stars| image:: https://img.shields.io/github/stars/alexjc/neural-doodle.svg?style=flat
    :target: https://github.com/alexjc/neural-doodle/stargazers
