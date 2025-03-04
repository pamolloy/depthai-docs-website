FAQs & How-To
=============

Why Does DepthAI Exist?
#######################

In trying to solve an Embedded :ref:`Spatial AI<spatialai>` problem (details `here <https://discuss.luxonis.com/d/8-it-works-working-prototype-of-commute-guardian>`__),
we discovered that although the perfect chip existed, there was no platform (hardware, firmware, or software) which
allowed the chip to be used to solve such an Embedded Spatial AI problem.

So we built the platform.

What is DepthAI?
################

DepthAI is *the* Embedded, Performant, Spatial AI+CV platform, composed of an open-source hardware, firmware, software ecosystem that
provides turnkey embedded :ref:`Spatial AI+CV<spatialai>` and hardware-accelerated computer vision.

It gives embedded systems the super-power of human-like perception in real-time: what an object is and where it is in physical space.

It can be used with off-the-shelf AI models (how-to :ref:`here <Use a Pre-trained OpenVINO model>`)
or with custom models using our completely-free training flow (how-to `here <https://colab.research.google.com/github/luxonis/depthai-ml-training/blob/master/colab-notebooks/Easy_Object_Detection_With_Custom_Data_Demo_Training.ipynb>`__).

An example of a custom-trained model is below, where DepthAI is used by a robot to autonomously pick and sort strawberries by ripeness.

.. image:: /_static/images/faq/strawberry.png
  :alt: Spatial AI Strawberry Example

It was trained to do so over the course of a weekend, by a student (for a student project), using our free online training resources.

DepthAI is also open-source (including hardware).  This is done so that companies (and even individuals) can prototype
and productize solutions quickly, autonomously, and at low risk.

See the summary of our (MIT-Licensed) Githubs :ref:`below <githubs>`, which include open-source hardware, firmware, software, and machine-learning training.


.. _spatialai:

What is SpatialAI?  What is 3D Object Localization?
###################################################

First, it is necessary to define what '`Object Detection <https://pjreddie.com/darknet/yolo/>`__' is:

.. image:: https://www.crowdsupply.com/img/7c80/depthai-dog-porch-ai_png_project-body.jpg
  :alt: Object Detection

It is the technical term for finding the bounding box of an object of interest, in pixel space (i.e. pixel coordinates), in an image.

3D Object Localization (or 3D Object Detection), is all about finding such objects in physical space, instead of pixel space.
This is useful when trying to real-time measure or interact with the physical world.

Below is a visualization to showcase the difference between Object Detection and 3D Object Localization:

.. image:: https://i.imgur.com/ABacp7x.png
  :target: https://www.youtube.com/watch?v=2J5YFehJ3N4
  :alt: Spatial AI Visualization

Spatial AI is then the super-set of such 2D-equivalent neural networks being extended with spatial information to give them 3D context.
So in other words, it's not limited to object detectors being extended to 3D object localizers.
Other network types can be extended as well, including any network which returns results in pixel space.

An example of such an extension is using a facial landmark detector on DepthAI.  With a normal camera this network returns
the 2D coordinates of all 45 facial landmarks (countours of eyes, ears, mouth, eybrows, etc.)  Using this same network
with DepthAI, each of these 45 facial landmarks is now a 3D point in physical space instead of 2D points in pixel space.


How Does DepthAI Provide Spatial AI Results?
############################################

There are two ways to use DepthAI to get Spatial AI results:

#. **Monocular Neural Inference fused with Stereo Depth.**
    In this mode the neural network is run on a single camera and fused with disparity depth results.  The left, right, or RGB camera can be used to run the neural inference.

#. **Stereo Neural Inference.**
    In this mode the neural network is run in parallel on both the left and right stereo cameras to produce 3D position data directly with the neural network.

In both of these cases, standard neural networks can be used.  There is no need for the neural networks to be trained with 3D data.

DepthAI automatically provides the 3D results in both cases using standard 2D-trained networks, as detailed :ref:`here <nodepthrequired>`.
These modes have differing minimum depth-perception limits, detailed :ref:`here <mindepths>`.


Monocular Neural Inference fused with Stereo Depth
**************************************************

In this mode, DepthAI runs object detection on a single cameras (user's choice: left, right, or RGB) and the results are
fused with the stereo disparity depth results.  The stereo disparity results are produced in parallel and in real-time
on DepthAI (based on semi global matching (SGBM)).

DepthAI automatically fuses the disparity depth results with the object detector results and uses this depth data for
each object in conjunction with the known intrinsics of the calibrated cameras to reproject the 3D position of the
detected object in physical space (X, Y, Z coordinates in meters).

And all of these calculations are done onboard to DepthAI without any processing load to any other systems.
This technique is great for object detectors as it provides the physical location of the centroid of the object -
and takes advantage of the fact that most objects are usually many pixels so the disparity depth results can be
averaged to produce a more accurate location.

A visualization of this mode is below.

.. image:: https://i.imgur.com/zTSyQpo.png
  :target: https://www.youtube.com/watch?v=sO1EU5AUq4U
  :alt: Monocular AI plus Stereo Depth for Spatial AI

In this case the neural inference (20-class object detection per :ref:`here <Run DepthAI Default Model>`)
was run on the RGB camera and the results were overlaid onto the depth stream.
The depthai reference Python script can be used to show this out (:code:`python3 depthai_demo.py -s metaout depth -bb` is the command used to produce the video above).

And if you'd like to know more about the underlying math that DepthAI is using to perform the stereo depth, see this excellent blog post here `here <https://www.learnopencv.com/introduction-to-epipolar-geometry-and-stereo-vision/>`__.  And if you'd like to run the same example run in that blog, on DepthAI, see this  `depthai-experiment <https://github.com/luxonis/depthai-experiments/tree/master/gen2-camera-demo#depth-from-rectified-host-images/>`__.


.. _stereo_inference:

Stereo Neural Inference
***********************

In this mode DepthAI runs the neural network in parallel on both the left and right stereo cameras.
The disparity of the results are then trianglulated with the calibrated camera intrinsics (programmed into the
EEPROM of each DepthAI unit) to give 3D position of all the detected features.

This **stereo neural inference** mode affords accurate 3D Spatial AI for networks which produce single-pixel locations
of features such as facial landmark estimation, pose estimation, or other meta-data which provides feature locations like this.

Examples include finding the 3D locations of:

 - Facial landmarks (eyes, ears, nose, edges of mouth, etc.)
 - Features on a product (screw holes, blemishes, etc.)
 - Joints on a person (e.g. elbow, knees, hips, etc.)
 - Features on a vehicle (e.g. mirrors, headlights, etc.)
 - Pests or disease on a plant (i.e. features that are too small for object detection + stereo depth)

Again, this mode does not require the neural networks to be trained with depth data.  DepthAI takes standard, off-the-shelf 2D networks (which are significantly more common) and uses this stereo inference to produce accurate 3D results.

An example of stereo neural inference is below.

.. image:: https://i.imgur.com/3kjFMt6.png
  :target: https://www.youtube.com/watch?v=eEnDW0WQ3bo
  :alt: DepthAI parallel multi-stage inference

And this is actuall an interesting case as it demonstrates two things on DepthAI:

#. Stereo inference (i.e. running the neural network(s) running on both the left and right cameras in parallel)
#. Multi-stage inference (i.e. face detection flowed directly into facial landmark directly on DepthAI)

The command used to run this on DepthAI is

.. code-block:: bash

  python3 depthai_demo.py -cnn face-detection-retail-0004 -cnn2 landmarks-regression-retail-0009 -cam left_right -dd -sh 12 -cmx 12 -nce 2 -monor 400 -monof 30


Where :code:`cam` specifies to run the neural network on both cameras, :code:`-cnn` specifies the first-stage network to
run (face detection, in this case), :code:`-cnn2` specifies the second-stage network (facial landmark detection, in this case),
and :code:`-dd` disables running disparity depth calculations (since they are unused in this mode).

Notes
*****

It is worth noting that monocular neural inference fused with stereo depth is possible for networks like facial-landmark
detectors, pose estimators, etc. that return single-pixel locations (instead of for example bounding boxes of
semantically-labeled pixels), but stereo neural inference is advised for these types of networks better results as
unlike object detectors (where the object usually covers many pixels, typically hundreds, which can be averaged for an
excellent depth/position estimation), landmark detectors typically return single-pixel locations.
So if there doesn't happen to be a good stereo-disparity result for that single pixel, the position can be wrong.

And so running stereo neural inference excels in these cases, as it does not rely on stereo disparity depth at all,
and instead relies purely on the results of the neural network, which are robust at providing these single pixel results.
And triangulation of the parallel left/right outputs results in very-accurate real-time landmark results in 3D space.


What is megaAI?
###############

The monocular (single-camera) version of DepthAI is megaAI.  Because not all solutions to embedded AI/CV problems require spatial information.

We named it :code:`mega` because it's tiny:

.. image:: https://www.crowdsupply.com/img/8182/megaai-quarter-original_png_project-body.jpg
  :alt: megaAI

megaAI uses all the same hardware, firmware, software, and training stacks as DepthAI (and uses the same DepthAI Githubs), it is simply the tiny single-camera variant.

You can buy megaAI from our distributors and also our online store `here <https://shop.luxonis.com/products/bw1093>`__.

Which Model Should I Order?
###########################

Embedded CV/AI requires all sorts of different shapes/sizes/permutations.  And so we have a variety of options to meet these needs.  Below is a quick/dirty summary for the ~10,000-foot view of the options:

- **USB3C with Onboard Cameras** (`BW1098OBC <https://shop.luxonis.com/products/bw10980bc>`__) - Great for quickly using DepthAI with a computer.
  All cameras are onboard, and it has a USB3C connection which can be used with any USB3 or USB2 host.

- **USB3C with Modular Cameras** (`BW1098FFC <https://shop.luxonis.com/products/depthai-usb3-edition>`__) - Great for prototyping flexibility.
  Since the cameras are modular, you can place them at various stereo baselines.  This flexibility comes with a trade -
  you have to figure out how/where you will mount them, and then once mounted, do a stereo calibration.
  This is not a TON of work, but keep this in mind, that it's not 'plug and play' like other options -
  it's more for applications that require custom mounting, custom baseline, or custom orientation of the cameras.

- **MegaAI Single Camera** (`BW1093 <https://shop.luxonis.com/products/bw1093>`__) - This is just like the BW1098OBC,
  but for those who don't need depth information.  Single, small, plug-and-play USB3C AI/CV camera.

- **Raspberry Pi Compute Module Edition** (`BW1097 <https://shop.luxonis.com/products/depthai-rpi-compute-module-edition>`__) -
  this one has a built-in Raspberry Pi Compute Module 3B+.  So you literally plug it into power and HDMI, and it boots up showing off the power of DepthAI.

- **Embedded DepthAI with WiFi/BT** (`BW1092 <https://shop.luxonis.com/products/bw1092>`__) - Currently this is in Alpha testing.
  So only buy it if you are comfortable with working with bleeding-edge tech and want to help us refine this product.
  It is the first Embedded (i.e. SPI-interface) version of DepthAI - so it has additional 128MB NOR flash, so it can boot
  on its own out of the NOR flash, and not host needs to be present to run.  In contrast, the BW1097 can also run on its own,
  but it is still booting over USB from the Raspberry Pi.  This BW1092, the Myriad X can run completely standalone and with no other devices.
  The built-in ESP32 then provides easy/convenient WiFi/BT support as well as popular integrations like plug-and-play AWS-IoT support, great iOS/Android BT examples, etc.

System on Modules
*****************

For designing products around DepthAI, we offer system on modules.  You can then design your own variants, leveraging our
`open source harware <https://github.com/luxonis/depthai-hardware>`__.  There are three system on modules available:

#. `BW1099 <https://shop.luxonis.com/collections/all/products/bw1099>`__ - USB-boot system on module. For making devices which interface over USB to a host processor running Linux, MacOS, or Windows. In this case, the host processor stores everything, and the BW1099 boots up over USB from the host.
#. `BW1099EMB <https://shop.luxonis.com/collections/all/products/bw1099emb>`__ - NOR-flash boot (also capable of USB-boot). For making devices that run standalone, or work with embedded MCUs like ESP32, AVR, STM32F4, etc.  Can also USB-boot if/as desirable.
#. `BW2099 <https://drive.google.com/file/d/13gI0mDYRw9-yXKre_AzAAg8L5PIboAa4/view?usp=sharing>`__ - NOR flash, eMMC, SD-Card, and USB-boot (selectable via IO on the 2x 100-pin connectors).  For making devices that run standalone and require onboard storage (16GB eMMC) and/or Ethernet Support (the onboard PCIE inferface through one of the 2x 100-pin connectors, paired with an Ethernet-capable base-board provides Ethernet support).

How hard is it to get DepthAI running from scratch?  What Platforms are Supported?
##################################################################################

Not hard.  Usually DepthAI is up/running on your platform within a couple minutes (most of which is download time).
The requirements are Python and OpenCV (which are great to have on your system anyway!). see
:ref:`here <Supported Platforms>` for supported platforms and how to get up/running with them.

**Raspbian, Ubuntu, macOS, Windows,** and many others are supported and are easy to get up/running.
For Install on various platforms are :ref:`here <Other installation methods>`.

It's a matter of minutes to be up and running with the power of Spatial AI, on the platform of your choice.  Below is DepthAI running on my Mac.

.. image:: https://i.imgur.com/9C9zOx5.png
  :alt: DepthAI on Mac
  :target: https://www.youtube.com/watch?v=SWDQekolM8o

(Click on the imageabove to pull up the YouTube video.)

The command to get the above output is

.. code-block:: bash

  python3 depthai_demo.py -s metaout previewout depth -ff -bb

Here is a single-camera version (megaAI) running with :code:`pytyon3 depthai_demo.py -dd` (to disable showing depth info):

.. image:: /_static/images/faq/lego.png
  :alt: DepthAI on Mac
  :target: https://www.youtube.com/watch?v=SWDQekolM8o


Is DepthAI and MegaAI easy to use with Raspberry Pi?
####################################################

Very. It's designed for ease of setup and use, and to keep the Pi CPU not-busy.

See :ref:`here <raspbian>` to get up and running quickly!


Can all the models be used with the Raspberry Pi?
#################################################

Yes, every model can be used, including:

- Raspberry Pi Compute Module Edition (`BW1097 <https://shop.luxonis.com/collections/all/products/depthai-rpi-compute-module-edition>`__ - this one has a built-in Raspberry Pi Compute Module 3B+
- Raspberry Pi HAT (`BW1094 <https://github.com/luxonis/depthai-hardware/tree/master/BW1094_DepthAI_HAT>`__) - this can also be used with other hosts as its interface is USB3
- USB3C with Onboard Cameras `BW1098OBC <https://shop.luxonis.com/collections/all/products/bw10980bc>`__
- USB3C with Modular Cameras `BW1098FFC <https://shop.luxonis.com/products/depthai-usb3-edition>`__
- MegaAI Single Camera `BW1093 <https://shop.luxonis.com/collections/all/products/bw1093>`__

We even have some basic ROS support going as well which can be used on the Pi also.


Does DepthAI Work on the NVIDIA Jetson Series?
##############################################

Yes, DepthAI and megaAI work cleanly on all the Jetson/Xavier series, and installation is easy.
Jetson Nano, Jetson Tx1, Jetson Tx2, Jetson Xavier NX, Jetson AGX Xavier, etc. are all supported.

See below for DepthAI running on a Jetson Tx2 I have on my desk:

.. image:: https://user-images.githubusercontent.com/32992551/93289854-a4cbcd00-f79c-11ea-8f37-4ea36d523dd2.png
  :alt: Jetson Tx2

Can I use multiple DepthAI with one host?
#########################################

Yes.  DepthAI is architected to put as-little-as-possible burden on the host.
So even with a Raspberry Pi you can run a handful of DepthAI with the Pi and not burden the Pi CPU.

See :ref:`here <Multiple DepthAI per Host>` for instructions on how to do so.

Is DepthAI OpenVINO Compatible?
###############################

Yes.  As of this writing, DepthAI is fully compatible with OpenVINO 2020.1.  We are in the process of upgrading to have compatibility with newer OpenVINO versions.

Can I train my own Models for DepthAI?
######################################

Yes.

We have a tutorial around Google Colab notebooks you can even use for this.  See `here <https://colab.research.google.com/github/luxonis/depthai-ml-training/blob/master/colab-notebooks/Easy_Object_Detection_With_Custom_Data_Demo_Training.ipynb>`__

.. _nodepthrequired:

Do I need Depth data to train my own custom Model for DepthAI?
##############################################################

No.

That's the beauty of DepthAI.  It takes standard object detectors (2D, pixel space) and fuses
these neural networks with stereo disparity depth to give you 3D results in physical space.

Now, could you train a model to take advantage of depth information?  Yes, and it would likely be even
more accurate than the 2D version.  To do so, record all the streams (left, right, and color) and
retrain on all of those (which would require modifying the front-end of say MobileNet-SSD to allow 5
layers instead of 3 (1 for each grayscale, 3 for the color R, G, B).

If I train my own network, which Neural Operations are supported by DepthAI?
############################################################################

See the :code:`VPU` section `here <https://docs.openvinotoolkit.org/latest/openvino_docs_IE_DG_supported_plugins_Supported_Devices.html>`__.

Anything that's supported there under :code:`VPU` will work on DepthAI.  It's worth noting that we haven't tested all of these
permutations though.

What network backbones are supported on DepthAI?
################################################

All the networks listed `here <https://docs.openvinotoolkit.org/latest/openvino_docs_IE_DG_supported_plugins_MYRIAD.html>`__ are supported by DepthAI.

We haven't tested all of them though.  So if you have a problem, contact us and we'll figure it out.

My Model Requires Pre-Processing (normalization, for example).  How do I do that in DepthAI?
############################################################################################

The OpenVINO toolkit allows adding these pre-processing steps to your model, and then these steps are performed automatically by DepthAI.  See `here <https://docs.openvinotoolkit.org/latest/openvino_docs_MO_DG_prepare_model_convert_model_Converting_Model_General.html#when_to_specify_mean_and_scale_values>`__ for how to take advantage of this.

How do I Integrate DepthAI into Our Product?
############################################

How to integrate DepthAI/megaAI depends on whether the product you are building includes

#. a processor running an operating system (Linux, MacOS, or Windows) or
#. a microcontroller (MCU) with no operating system (or an RTOS like FreeRTOS) or
#. no other processor or microcontroller (i.e. DepthAI is the only processor in the system).

We offer hardware to support all 3 use-cases, but firmware/software mauturity varies across the 3 modes:

#. the most mature, using our :ref:`Python API <Python API>`
#. initially released by actively in development (see `here <https://discuss.luxonis.com/d/56-initial-bw1092-esp32-proof-of-concept-code>`__),
#. supported in December 2020 (as part of Pipeline Builder Gen2 `here <https://github.com/luxonis/depthai/issues/136>`__).

In all cases, DepthAI (and megaAI) are compatible with OpenVINO for neural models.  The only thing that changes between
the modalities is the communication (USB, Ethernet, SPI, etc.) and what (if any) other processor is involved.

.. _withos:

Use-Case 1: DepthAI/megaAI are a co-processor to a processor running Linux, MacOS, or Windows.
**********************************************************************************************

In this case, DepthAI can be used in two modalities:

 - NCS2 Mode (USB, :ref:`here <ncsmode>`) - in this mode, the device appears as an NCS2 and the onboard cameras are not used and it's as if they don't exist.  This mode is often use for initial prototyping, and in some cases, where a product simply needs an 'integrated NCS2' - accomplished by integrating a `BW1099 <https://shop.luxonis.com/collections/all/products/bw1099>`__.
 - DepthAI Mode (USB, using our USB API, :ref:`here <Python API>`) - this uses the onboard cameras directly into the Myriad X, and boots the firmware over USB from a host processor running Linux, Mac, or Windows.  This is the main use-case of DepthAI/megaAI when used with a host processor capable of running an operating system (e.g Raspberry Pi, i.MX8, etc.).

.. _withmicrocontroller:

Use-Case 2: Using DepthAI with a MicroController like ESP32, ATTiny8, etc.
**************************************************************************

In this case, DepthAI boot off of internal flash on the `BW1099EMB <https://shop.luxonis.com/collections/all/products/bw1099emb>`__ and communicates over SPI, allowing DepthAI to be used with microcontroller such as the STM32, MSP430, ESP32, ATMega/Arduino, etc.  We even have an embedded reference design for ESP32 (`BW1092 <https://github.com/luxonis/depthai-hardware/issues/10>`__) available on our `store <https://shop.luxonis.com/collections/all/products/bw1092-pre-order>`__.  We will also be open-sourcing this design after it is fully verified (contact us if you would like the design files before we open source it).

The code-base/API for this is in active development, and a pre-release/Alpha version is available `here <https://discuss.luxonis.com/d/56-initial-bw1092-esp32-proof-of-concept-code>`__ as of this writing.

.. _standalone:

Use-Case 3: Using DepthAI as the Only Processor on a Device.
************************************************************

This will be supported through running microPython directly on the `BW1099EMB <https://shop.luxonis.com/collections/all/products/bw1099emb>`__ as nodes in the `Gen2 Pipeline Builder <https://github.com/luxonis/depthai/issues/136>`__.

The microPython nodes are what will allow custom logic, driving I2C, SPI, GPIO, UART, etc. controls, allowing direct controls of actuators, direct reading of sensors, etc. from/to the pipeline of CV/AI functions.
A target example is making an entire autonomous, visually-controlled robotic platform with DepthAI as the only processor in the system.

The target date for this mode is December 2020.

Hardware for Each Case:
***********************

- BW1099: USB boot. So it is intended for working with a host processor running Linux, Mac, or Windows and this host processor boots the BW1099 over USB
- BW1099EMB: USB boot or NOR-flash boot. This module can work with a host computer just like the BW1099, but also has a 128MB NOR flash built-in and boot switches onboard - so that it can be programmed to boot off of NOR flash instead of of USB. So this allows use of the DepthAI in pure-embedded applications where there is no operating system involved at all. So this module could be paired with an ATTiny8 for example, communicating over SPI, or an ESP32 like on the BW1092 (which comes with the BW1099EMB pre-installed).

Getting Started with Development
********************************

Whether intending to use DepthAI with an :ref:`OS-capable host <withos>`, a :ref:`microcontroller over SPI <withmicrocontroller>`
(in development), or :ref:`completely standalone <standalone>` (targeted support December 2020) - we recommend starting with either
:ref:`NCS2 mode <ncsmode>` or with the :ref:`DepthAI USB API <Python API>` for prototype/test/etc. as it allows faster iteration/feedback on
neural model performance/etc.  And in particular, with NCS2 mode, all the images/video can be used directly from the host (so that you don't have to point the camera at the thing you want to test).

In DepthAI mode, theoretically anything that will run in NCS2 mode will run - but sometimes it needs host-side processing if it's a network we've never run before - and for now it will run only off of the image sensors (once the `Gen2 pipeline builder <https://github.com/luxonis/depthai/issues/136>`__ is out, which is scheduled for December 2020, there will exist the capability to run everything off of host images/video with the DepthAI API).  And this work is usually not heavy lifting... for example we had never run semantic segmentation networks before via the DepthAI API (and therefore had no reference code for doing so), but despite this one of our users actually got it working in a day without our help (e.g here).

For common object detector formats (MobileNet-SSD, tinyYOLOv1/2/3, etc.) there's effectively no work to go from NCS2 mode to DepthAI mode.  You can just literally replace the classes in example MobileNet-SSD or tinyYOLO examples we have.  For example for tinyYOLOv3, you can just change the labels from "mask", "no mask" and "no mask 2" to whatever your classes are from this example `here <https://github.com/luxonis/depthai/blob/main/resources/nn/tiny-yolo/tiny-yolo.json>`__ and just change the blob file `here <https://github.com/luxonis/depthai/tree/main/resources/nn/tiny-yolo>`__ to your blob file.  And the same thing is true for MobileNet-SSD `here <https://github.com/luxonis/depthai/tree/main/resources/nn/mobilenet-ssd>`__.


What Hardware-Accelerated Capabilities Exist in DepthAI and/or megaAI?
######################################################################

Available in DepthAI API Today:
*******************************

- Neural Inference (e.g. object detection, image classification, etc., including two-stage, e.g. `here <https://youtu.be/uAfGulcDWSk>`__)
- Stereo Depth (including median filtering) (e.g. `here <https://youtu.be/sO1EU5AUq4U>`__)
- Stereo Inference (with two-stage, e.g. `here <https://youtu.be/eEnDW0WQ3bo>`__)
- 3D Object Localization (augmenting 2D object detectors with 3D position in meters, e.g. `here <https://youtu.be/cJr4IpGMSLA>`__ and `here <https://youtu.be/SWDQekolM8o>`__)
- Object Tracking (e.g. `here <https://vimeo.com/422965770>`__, including in 3D space)
- H.264 and H.265 Encoding (HEVC, 1080p & 4K Video, e.g. `here <https://youtu.be/vEq7LtGbECs>`__)
- JPEG Encoding
- MJPEG Encoding
- Warp/Dewarp
- Enhanced Disparity Depth Modes (Sub-Pixel, LR-Check, and Extended Disparity), `here <https://github.com/luxonis/depthai/issues/163>`__
- SPI Support, `here <https://github.com/luxonis/depthai/issues/140>`__
- Arbitrary crop/rescale/reformat and ROI return (`here <https://github.com/luxonis/depthai/issues/249>`__)

The above features are available in the Luxonis Pipeline Builder Gen1 (see example :ref:`here <API Reference>`).  See :ref:`Pipeline Builder Gen2 <pipelinegen2>` for in-progress additional functionality/flexibility which will come with the next generation Luxonis pipeline builder for DepthAI.

On our Roadmap (planned delivery December 2020)
***********************************************

- Pipeline Builder Gen2 (arbitrary series/parallel combination of neural nets and CV functions, details `here <https://github.com/luxonis/depthai/issues/136>`__)
- Improved Stereo Neural Inference Support (`here <https://github.com/luxonis/depthai/issues/216>`__)
- microPython Support, `here <https://github.com/luxonis/depthai/issues/207>`__
- Feature Tracking (including IMU-assisted feature tracking, `here <https://github.com/luxonis/depthai/issues/146>`__)
- Integrated IMU Support (`here <https://github.com/luxonis/depthai-hardware/issues/8>`__)
- Motion Estimation (`here <https://github.com/luxonis/depthai/issues/245>`__)
- Background Subtraction (`here <https://github.com/luxonis/depthai/issues/136>`__)
- Lossless zoom (from 12MP full to 4K, 1080p, or 720p, `here <https://github.com/luxonis/depthai/issues/135>`__)
- Edge Detection (`here <https://github.com/luxonis/depthai/issues/247>`__)
- Harris Filtering (`here <https://github.com/luxonis/depthai/issues/248>`__)
- AprilTags (PR `here <https://github.com/luxonis/depthai/pull/139>`__)
- Integrated Text Detection (`here <https://github.com/luxonis/depthai/issues/124>`__)
- OpenCL Support (supported through OpenVINO (`here <https://docs.openvinotoolkit.org/latest/openvino_docs_IE_DG_Extensibility_DG_VPU_Kernel.html>`__))

And see our Github project `here <https://github.com/orgs/luxonis/projects/2>`__ to follow along with the progress of these implementations.

.. _pipelinegen2:

Pipeline Builder Gen2
*********************

We have been working on a 2nd-generation pipeline builder which will incorporate many of the features below on our roadmap into a graphical drag/drop AI/CV pipeline which will then run entirely on DepthAI and return results of interest to the host.

This allows multi-stage neural networks to be pieced together in conjunction with CV functions (such as motion estimation or Harris filtering) and logical rules, all of which run on DepthAI/megaAI without any load on the host.

Are CAD Files Available?
########################

Yes.

The full designs (including source Altium files) for all the carrier boards are in our `depthai-hardware <https://github.com/luxonis/depthai-hardware>`__ Github


.. _mindepths:

What are the Minimum Depths Visible by DepthAI?
###############################################

There are two ways to use DepthAI for 3D object detection and/or using neural information to get real-time 3D position of features (e.g. facial landmarks):

#. Monocular Neural Inference fused with Stereo Depth
#. Stereo Neural Inference

Monocular Neural Inference fused with Stereo Depth
**************************************************

In this mode, the AI (object detection) is run on the left, right, or RGB camera, and the results are fused with stereo disparity depth, based on semi global matching (SGBM).  The minimum depth is limited by the maximum disparity search, which is by default 96, but is extendable to 192 in extended disparity modes (see :ref:`Extended Disparity <Extended Disparity Depth Mode>` below).

To calculate the minimum distance in this mode, use the following formula, where base_line_dist and min_distance are in meters [m]:
.. code-block:: python

  min_distance = focal_length * base_line_dist / 96

Where 96 is the standard maximum disparity search used by DepthAI and so for extended disparity (192 pixels), the minimum distance is:

.. code-block:: python

  min_distance = focal_length * base_line_dist / 192

For DepthAI, the HFOV of the the grayscale global shutter cameras is 73.5 degrees (this can be found on your board, see
`here <https://docs.luxonis.com/faq/#what-are-the-minimum-and-maximum-depth-visible-by-depthai>`__, so the focal length is

.. code-block:: python

  focal_length = 1280/(2*tan(73.5/2/180*pi)) = 857.06

Calculation `here <https://www.google.com/search?safe=off&sxsrf=ALeKk01DFgdNHlMBEkcIJdWmArcgB8Afzg%3A1607995029124&ei=lQ7YX6X-Bor_-gSo7rHIAg&q=1280%2F%282*tan%2873.5%2F2%2F180*pi%29%29&oq=1280%2F%282*tan%2873.5%2F2%2F180*pi%29%29&gs_lcp=CgZwc3ktYWIQAzIECCMQJzoECAAQR1D2HljILmDmPWgAcAJ4AIABywGIAZMEkgEFNC4wLjGYAQCgAQGqAQdnd3Mtd2l6yAEFwAEB&sclient=psy-ab&ved=0ahUKEwjlnIuk6M7tAhWKv54KHSh3DCkQ4dUDCA0&uact=5>`__
(and for disparity depth data, the value is stored in :code:`uint16`, where the max value of :code:`uint16` of 65535 is a special value, meaning that that distance is unknown.)

Stereo Neural Inference
***********************

In this mode, the neural inference (object detection, landmark detection, etc.) is run on the left *and* right cameras to produce stereo inference results.  Unlike monocular neural inference fused with stereo depth - there is no max disparity search limit - so the minimum distance is purely limited by the greater of (a) horizontal field of view (HFOV) of the stereo cameras themselves and (b) the hyperfocal distance of the cameras.

The hyperfocal distance of the global shutter synchronized stereo pair is 19.6cm.  So objects closer than 19.6cm will appear out of focus.
This is effectively the minimum distance for this mode of operation, as in most cases (except for very wide stereo baselines with the :ref:`BW1098FFC <BW1098FFC - USB3 with Modular Cameras>`),
this **effective** minimum distance is higher than the **actual** minimum distance as a result of the stereo camera field of views.
For example, the objects will be fully out of the field of view of both grayscale cameras when less than `5.25cm <https://www.google.com/search?ei=GapBX-y3BsuxtQa3-YaQBw&q=%3Dtan%28%2890-71%2F2%29*pi%2F180%29*7.5%2F2&oq=%3Dtan%28%2890-71%2F2%29*pi%2F180%29*7.5%2F2&gs_lcp=CgZwc3ktYWIQAzoECAAQR1DZkwxYmaAMYPilDGgAcAF4AIABS4gB1AKSAQE1mAEAoAEBqgEHZ3dzLXdpesABAQ&sclient=psy-ab&ved=0ahUKEwisqPat-6_rAhXLWM0KHbe8AXIQ4dUDCAw&uact=5>`__
from the :ref:`BW1098OBC <BW1098OBC - USB3 with Onboard Cameras>`), but that is closer than the hyperfocal distance of the grayscale cameras (which is 19.6cm), so the actual minimum depth is this hyperfocal distance.

Accordingly, to calculate the minimum distance for this mode of operation, use the following formula:

.. code-block:: python

  min_distance = max(tan((90-HFOV/2)*pi/2)*base_line_dist/2, 19.6)

This formula implements the maximum of the HFOV-imposed minimum distance, and 19.6cm, which is the hyperfocal-distance-imposed minimum distance.

Onboard Camera Minimum Depths
*****************************

Below are the minimum depth perception possible in the disparity depth and stereo neural inference modes.

Monocular Neural Inference fused with Stereo Depth Mode
-------------------------------------------------------

For DepthAI units with onboard cameras, this works out to the following minimum depths:

- DepthAI RPi Compute Module Edition (:ref:`BW1097 <BW1097 - RaspberryPi Compute Module>`) the minimum depth is **0.827** meters for full 1280x800 stereo resolution and  **0.414** meters for 640x400 stereo resolution:

.. code-block:: python

  min_distance = 857.06.15 * 0.09 / 96 = 0.803 # m

calculation `here <https://www.google.com/search?safe=off&sxsrf=ALeKk00zuPUIqtKg9E4O1fSrB4IFp04AQw%3A1607995753791&ei=aRHYX57zL9P9-gTk5rmADA&q=857.06*.09%2F96&oq=857.06*.09%2F96&gs_lcp=CgZwc3ktYWIQAzIECCMQJ1CqJ1i8OmDlPGgAcAB4AIABX4gB9ASSAQE4mAEAoAEBqgEHZ3dzLXdpesABAQ&sclient=psy-ab&ved=0ahUKEwjey9H96s7tAhXTvp4KHWRzDsAQ4dUDCA0&uact=5>`__

- USB3C Onboard Camera Edition (:ref:`BW1098OBC <BW1098OBC - USB3 with Onboard Cameras>`) is **0.689** meters:

.. code-block:: python

  min_distance = 857.06*.075/96 = 0.669 # m

calculation `here <https://www.google.com/search?safe=off&sxsrf=ALeKk03HLvlfCWau-bIGeYWJk_S6PBSnqw%3A1607995818683&ei=qhHYX4yeKZHr-gSv2JqoAw&q=857.06*.075%2F96&oq=857.06*.075%2F96&gs_lcp=CgZwc3ktYWIQAzIECCMQJ1CIFliUGmDvHGgAcAB4AIABUIgBrwKSAQE0mAEAoAEBqgEHZ3dzLXdpesABAQ&sclient=psy-ab&ved=0ahUKEwiMm8qc687tAhWRtZ4KHS-sBjUQ4dUDCA0&uact=5>`__

Stereo Neural Inference Mode
----------------------------

For DepthAI units with onboard cameras, all models (:ref:`BW1097 <BW1097 - RaspberryPi Compute Module>` and :ref:`BW1098OBC <BW1098OBC - USB3 with Onboard Cameras>`) are
limited by the hyperfocal distance of the stereo cameras, so their minimum depth is **0.196** meters.

Modular Camera Minimum Depths:
******************************

Below are the minimum depth perception possible in the disparity disparity depth and stereo neural inference modes.

Monocular Neural Inference fused with Stereo Depth Mode
-------------------------------------------------------

For DepthAI units which use modular cameras, the minimum baseline is 2.5cm (see image below) which means the minimum perceivable depth **0.229** meters  for full 1280x800 resolution and **0.196** meters for 640x400 resolution (limited by the minimum focal distance of the grayscale cameras, as in stereo neural inference mode).

The minimum baseline is set simply by how close the two boards can be spaced before they physically interfere:

.. image:: /_static/images/faq/modular-stereo-cam-min-dist.png
  :alt: Jetson Tx2

Stereo Neural Inference Mode
----------------------------

For any stereo baseline under 29cm, the minimum depth is dictacted by the hyperfocal distance (the distance above which objects are in focus) of 19.6cm.

For stereo baselines wider than 29cm, the minimum depth is limited by the horizontal field of view (HFOV):

.. code-block:: python

  min_distance = tan((90-HFOV/2)*pi/2)*base_line_dist/2


.. _extended_disparity:

Extended Disparity Depth Mode
*****************************

If it is of interest in your application, we can implement a system called :code:`extended disparity` which affords a closer minimum distance for the given baseline.  This increases the maximum disparity search from 96 to 192.  So this cuts the minimum perceivable distance in half (given that the minimum distance is now :code:`focal_length * base_line_dist / 192` instead of :code:`focal_length * base_line_dist / 96`).

- DepthAI RPi Compute Module Edition (`BW1097 <https://docs.luxonis.com/products/bw1097/>`__): **0.414** meters
- USB3C Onboard Camera Edition (`BW1098OBC <https://docs.luxonis.com/products/bw1098obc/>`__) is **0.345** meters
- Modular Cameras at Mimumum Spacing (e.g. `BW1098FFC <https://docs.luxonis.com/products/bw1098ffc/>`__) is **0.115** meters

So if you have the need for this shorter minimum distance when using monocular neural inference fused with disparity depth, reach out to us on discord, email, or discuss.luxonis.com to let us know.  It's on our roadmap but we haven't yet seen a need for it, so we haven't prioritized implementing it (yet!).


What Are The Maximum Depths Visible by DepthAI?
###############################################

The maximum depth perception for 3D object detection is is practically limited by how far the object detector (or other neural network) can detect what it's looking for. We've found that OpenVINO people detectors work to about 22 meters or so. But generally this distance will be limited by how far away the object detector can detect objects, and then after that, the minimum angle difference between the objects.

So if the object detector is not the limit, the maximum distance will be limited by the physics of the baseline and the number of pixels. So once an object is less than 0.056 degrees (which corresponds to 1 pixel difference) difference between one camera to the other, it is past the point where full-pixel disparity can be done.  The formula used to calculate this distance is an approximation, but is as follows:

.. code-block:: python

  Dm = (baseline/2) * tan_d((90 - HFOV / HPixels)*pi/2)

For DepthAI HFOV = 73.5(+/-0.5) degrees, and HPixels = 1280.  And for the BW1098OBC, the baseline is 7.5cm.

So using this formula for existing models the *theoretical* max distance is:

- BW1098OBC (OAK-D; 7.5cm baseline): 38.4 meters
- BW1097 (9cm baseline): 46 meters
- Custom baseline: Dm = (baseline/2) * tan_d(90 - 73.5 / 1280)

But these theoretical maximums are not achievable in the real-world, as the disparity matching is not perfect, nor are the optics, image sensor, etc., so the actual maximum depth will be application-specific depending on lighting, neural model, feature sizes, baselines, etc.

After the `KickStarter campaign <https://www.kickstarter.com/projects/opencv/opencv-ai-kit/description>`__ we will also be supporting sub-pixel, which will extend this theoretical max, but again this will likely not be the -actual- limit of the max object detection distance, but rather the neural network itself will be.  And this subpixel use will likely have application-specific benefits.

What Is the Format of the Depth Data in depth stream?
#####################################################

The output array is in uint16, so 0 to 65,535 with direct mapping to millimeters (mm).

So a value of 3,141 in the array is 3,141 mm, or 3.141 meters.  So this whole array is the z-dimension of each pixel off of the camera plane, where the :code:`center of the universe` is the camera marked :code:`RIGHT`.

And the specific value of 65,535 is a special value, meaning an invalid disparity/depth result.

How Do I Calculate Depth from Disparity?
########################################

DepthAI does convert to depth onboard for both the :code:`depth` stream and also for object detectors like MobileNet-SSD, YOLO, etc.

But we also allow the actual disparity results to be retrieved so that if you would like to use the disparity map directly, you can.

To calculate the depth map from the disparity map, it is (approximately) :code:`baseline * focal / disparity`.  Where the baseline is 7.5cm for BW1098OBC, 4.0cm for BW1092, and 9.0cm for BW1097, and the focal length is :code:`883.15` (:code:`focal_length = 1280/(2*tan(73.5/2/180*pi)) = 857.06`) for all current DepthAI models.

So for example, for a BW1092 (stereo baseline of 4.0cm), a disparity measurement of 60 is a depth of 58.8cm (:code:`depth = 40 * 857.06 / 60 = 571 mm (0.571m)`).

How Do I Display Multiple Streams?
##################################

To specify which streams you would like displayed, use the :code:`-s` option.  For example for metadata (e.g. bounding box results from an object detector), the color stream (:code:`previewout`), and for depth results (:code:`depth`), use the following command:

.. code-block:: bash

  python3 depthai_demo.py -s metaout previewout depth


The available streams are:
  - :code:`metaout` - Meta data results from the neural network
  - :code:`previewout` - Small preview stream from the color camera
  - :code:`color` - 4K color camera, biggest camera on the board with lens
  - :code:`left` - Left grayscale camera (marked `L` or `LEFT` on the board)
  - :code:`right` - Right grayscale camera (marked `R` or `RIGHT` on the board)
  - :code:`rectified_left` - `Rectified <https://en.wikipedia.org/wiki/Image_rectification>`__ left camera frames
  - :code:`rectified_right` - `Rectified <https://en.wikipedia.org/wiki/Image_rectification>`__ right camera frames
  - :code:`depth` - Depth in `uint16` (see `here <https://docs.luxonis.com/faq/-what-are-the-minimum-and-maximum-depth-visible-by-depthai>`__ for the format.
  - :code:`disparity` - Raw disparity
  - :code:`disparity_color` - Disparity colorized on the host (:code:`JET` colorized visualization of depth)
  - :code:`meta_d2h` - Device die temperature (max temp should be < 105C)
  - :code:`object_tracker` - Object tracker results

Is It Possible to Have Access to the Raw Stereo Pair Stream on the Host?
************************************************************************

Yes, to get the raw stereo pair stream on the host use the following command:

.. code-block:: bash

  python3 depthai_demo.py -s left right

This will show the full RAW (uncompressed) 1280x720 stereo synchronized pair, as below:

.. image:: https://i.imgur.com/oKVrZAV.jpg
  :alt: RAW Stereo Pair Streams

How Do I Limit The FrameRate Per Stream?
########################################

So the simple way to select streams is to just use the :code:`-s` option.  But in some cases (say when you have a slow host or only USB2 connection -and- you want to display a lot of streams) it may be necessary to limit the framerate of streams to not overwhelm the host/USB2 with too much data.

So to set streams to a specific framerate to reduce the USB2 load and host load, simply specify the stream with :code:`-s streamname` with a comma and FPS after the stream name like :code:`-s streamname,FPS`.

So for limiting `depth` to 5 FPS, use the following command:

.. code-block:: bash

  python3 depthai_demo.py -s depth,5

And this works equally for multiple streams:

.. code-block:: bash

  python3 depthai_demo.py -s left,2 right,2 previewout depth,5

It's worth noting that the framerate limiting works best for lower rates.  So if you're say trying to hit 25FPS, it's best to just leave no frame-rate specified and let the system go to full 30FPS instead.

Specifying no limit will default to 30FPS.

One can also use the following override command structure, which allows you to set the framerate per stream.

The following example sets the :code:`depth` stream to 8 FPS and the :code:`previewout` to 12 FPS:

.. code-block:: bash

  python3 depthai_demo.py -co '{"streams": [{"name": "depth", "max_fps": 8.0},{"name": "previewout", "max_fps": 12.0}]}'

You can pick/choose whatever streams you want, and their frame rate, but pasting in additional :code:`{"name": "streamname", "max_fps": FPS}` into the expression above.

How do I Synchronize Streams and/or Meta Data (Neural Inference Results)
########################################################################

The :code:`-sync` option is used to synchronize the neural inference results and the frames on which they were run.  When this option is used, the device-side firmware makes a best effort to send metadata and frames in order of metadata first, immediately followed by the corresponding image.

When running heavier stereo neural inference, particularly with high host load, this system can break down, and there are two options which can keep synchronization:

#. Reduce the framerate of the cameras running the inference to the speed of the neural inference itself, or just below it.
#. Or pull the timestamps or sequence numbers from the results (frames or metadata) and match them on the host.

Reducing the Camera Frame Rate
******************************

In the case of neural models which cannot be executed at the full 30FPS, this can cause lack of synchronization, particularly if stereo neural inference is being run using these models in parallel on the left and right grayscale image sensors.

A simple/easy way to regain synchronization is to reduce the framerate to match, or be just below, the framerate of the neural inference.  This can be accomplished via the command line with the using :code:`-rgbf` and :code:`-monof` commands.

So for example to run a default model with both the RGB and both grayscale cameras set to 24FPS, use the following command:

.. code-block:: bash

  ./depthai_demo.py -rgbf 24 -monof 24 -sync

Synchronizing on the host
*************************

The two methods :func:`FrameMetadata.getTimestamp` and :func:`FrameMetadata.getSequenceNum` can be used to guarantee the synchronization on host side.

The NNPackets and DataPackets are being sent separately from device side, and get into individual queues per stream on host side.
The function :func:`CNNPipeline.get_available_nnet_and_data_packets` returns what's available in the queues at the moment the function is called (it could be that just one NN packet is unread, or just one frame packet).

With the :code:`-sync` CLI option from depthai.py, we are doing a best effort on the device side (i.e. on the Myriad X) to synchronize NN and previewout, and send them in order: first the NN packet is being sent (and in depthai.py it gets  saved as the latest), then the previewout frame is being sent (and when received in depthai_demo.py, the latest saved NN data is overlaid on).

In most cases this works well, but there is a risk (especially under high system load on host side), that the packets may still get desynchronized, as the queues are handled by different threads (in the C++ library).

So in that case, :code:`getMetadata().getTimestamp()` returns the device time (in seconds, as float) and is also `used in the stereo calibration script <https://github.com/luxonis/depthai/blob/f26f8c6/calibrate.py#L114>`__ to synchronize the Left and Right frames.

The timestamp corresponds to the moment the frames are captured from the camera, and is forwarded through the pipeline.  And the method :code:`getMetadata().getSequenceNum()` returns an incrementing number per camera frame. The same number is associated to the NN packet, so it could be an easier option to use, rather than comparing timestamps. The NN packet and Data packet sequence numbers should match.

Also, the left and right cameras will both have the same sequence number (timestamps will not be precisely the same, but few microseconds apart -- that's because the timestamp is assigned separately to each from different interrupt handlers. But the cameras are started at the same time using an I2C broadcast write, and also use the same MCLK source, so shouldn't drift).

In this case we also need to check the camera source of the NN and Data packets. Currently, depthai.py uses getMetadata().getCameraName() for this purpose, that returns a string: :code:`rgb`, :code:`left` or :code:`right` .

It is also possible to use :code:`getMetadata().getInstanceNum()`, that returns a number: 0, 1 or 2 , respectively.

How do I Record (or Encode) Video with DepthAI?
###############################################

DepthAI suppots h.264 and h.265 (HEVC) and JPEG encoding directly itself - without any host support.  The `depthai_demo.py` script shows and example of how to access this functionality.

To leverage this functionality from the command line, use the `-v` (or `--video`) command line argument as below:

.. code-block:: bash

  python3 depthai_demo.py -v [path/to/video.h264]

To then play the video in mp4/mkv format use the following muxing command:

.. code-block:: bash

  ffmpeg -framerate 30 -i [path/to/video.h264] -c copy [outputfile.mp4/mkv]

By default there are keyframes every 1 second which resolve the previous issues with traversing the video as well as provide the capability to start recording anytime (worst case 1 second of video is lost if just missed the keyframe)

When running depthai_demo.py, one can record a jpeg of the current frame by hitting :code:`c` on the keyboard.

An example video encoded on DepthAI `BW1097 <https://shop.luxonis.com/collections/all/products/depthai-rpi-compute-module-edition>`__ (Raspberry Pi Compute Module Edition) is below.  All DepthAI and megaAI units have the same 4K color camera, so will have equivalent performance to the video below.

.. image:: https://i.imgur.com/xjBEPKc.jpg
  :alt: 4K Video in 3.125MB/s on DepthAI with Raspberry Pi 3B
  :target: https://www.youtube.com/watch?v=vEq7LtGbECs

Video Encoding Options
**********************

Additional options can be configured in the video encoding system by adding a :code:`video_config` section to the JSON config of the DepthAI pipeline builder, `here <https://github.com/luxonis/depthai/blob/d357bbda64403f69e3f493f14999445b46214264/depthai.py#L342>`__, an example of which is `here <https://github.com/luxonis/depthai/blob/dd42668f02fb3ba4e465f29915c8ca586dfc99cc/depthai.py#L342>`__.

.. code-block:: python

  config = {
    ...
    'video_config':
    {
        'rateCtrlMode': 'cbr', # Options: 'cbr' / 'vbr' (constant bit rate or variable bit rate)
        'profile': 'h265_main', # Options: 'h264_baseline' / 'h264_main' / 'h264_high' / 'h265_main'
        'bitrate': 8000000, # When using CBR
        'maxBitrate': 8000000, # When using CBR
        'keyframeFrequency': 30, # In number of frames
        'numBFrames': 0,
        'quality': 80 # (0 - 100%) When using VBR
    }
    ...
  }

The options above are all current options exposed for video encoding and not all must be set.

If :code:`video_config` member is **NOT** present in config dictionary then default is used: H264_HIGH, constant bitrate 8500Kbps, keyframe every 30 frames (once per second), num B frames: 0

What Are The Stream Latencies?
##############################

When implementing robotic or mechatronic systems it is often quite useful to know how long it takes from a photo hitting an image sensor to when the results are available to a user, the :code:`photon-to-results` latency.

So the following results are an approximation of this :code:`photon-to-results` latency, and are likely an over-estimate
as we tested by actually seeing when results were updated on a monitor, and the monitor itself has some latency, so the
results below are likely overestimated by whatever the latency of the monitor is that we used during the test.
And we have also since done several optimizations since these measurements, so the latency could be quite a bit lower than these.

.. list-table:: Worst-case estimates of stream latencies
  :widths: 25 50 25
  :header-rows: 1
  :align: center

  * - measured
    - requested
    - avg latency, ms
  * - left
    - left
    - 100
  * - left
    - left, right
    - 100
  * - left
    - left, right, depth
    - 100
  * - left
    - left, right, depth, metaout, previewout
    - 100
  * - previewout
    - previewout
    - 65
  * - previewout
    - metaout, previewout
    - 100
  * - previewout
    - left, right, depth, metaout, previewout
    - 100
  * - metaout
    - metaout
    - 300
  * - metaout
    - metaout, previewout
    - 300
  * - metaout
    - left, right, depth, metaout, previewout
    - 300


Is it Possible to Use the RGB camera and/or the Stereo Pair as a Regular UVC Camera?
####################################################################################

Yes, but currently not currently implemented in our API.  It's on our roadmap, `here <https://github.com/luxonis/depthai/issues/283>`__ 

The :code:`why` of our DepthAI API provides more flexibility in formats (unencoded, encoded, metadata, processing, frame-rate, etc.) and already works on any operating system (see :ref:`here <Supported Platforms>`).  So what we plan to do is to support UVC as part of our Gen2 Pipeline builder, so you can build a complex spatial AI/CV pipeline and then have the UVC endpoints output the results, so that DepthAI could then work on any system without drivers.  For our embedded variants, this could then be flashed to the device so that the whole pipeline will automatically run on boot-up and show up to a computer a UVC device (a webcame).

Theoretically we can implement support for 3 UVC endpoints (so showing up as 3 UVC cameras), on for each of the 3 cameras. 

We've prototyped 2x w/ internal proof of concept (but grayscale) but have not yet tried 3 but it would probably work.
We could support a UVC stream per camera if it is of interest.

So if you would like this functionality please feel subscribe to the Github feature request `here <https://github.com/luxonis/depthai/issues/283>`__.

How Do I Force USB2 Mode?
#########################

USB2 Communication may be desirable if you'd like to use extra-long USB cables and don't need USB3 speeds.

To force USB2 mode, simply use the `-fusb2` (or `--force_usb2`) command line option as below:

.. code-block:: bash

  python3 depthai_demo.py -fusb2

Note that if you would like to use DepthAI at distances that are even greater than what USB2 can handle, we do have DepthAI PoE variants coming, see `here <https://discuss.luxonis.com/d/30-luxonis-depthai-for-raspberry-pi-overview-and-status/29>`__, which allow DepthAI to use up to a 328.1 foot (100 meter) cable for both data and power - at 1 gigabit per second (1gbps).

.. _ncsmode:

What is "NCS2 Mode"?
####################

All variants of DepthAI/megaAI come supporting what we call 'NCS2 mode'.  This allows megaAI and DepthAI to pretend to be an NCS2.

So in fact, if you power your unit, plug it into a computer, and follow the instructions/examples/etc. of an NCS2 with OpenVINO, DepthAI/megaAI will behave identically.

This allows you to try out examples from OpenVINO directly as if our hardware is an NCS2.  This can be useful when
experimenting with models which are designed to operate on objects/items that you may not have available locally/physically.
It also allows running inference in programmatic ways for quality assurance, refining model performance, etc.,
as the images are pushed from the host, instead of pulled from the onboard camera in this mode.

DepthAI/megaAI will also support an additional host-communication mode in the `Gen2 Pipeline Builer <https://github.com/luxonis/depthai/issues/136>`__, which will be available in December 2020.

What Information is Stored on the DepthAI Boards
################################################

Initial Crowd Supply backers received boards which had literally nothing stored on them.  All information was loaded
from the host to the board.  This includes the BW1097 (:ref:`BW1097 <BW1097 - RaspberryPi Compute Module>`), which had the calibration stored on the included microSD card.

So each hardware model which has stereo cameras (e.g. :ref:`BW1097 <BW1097 - RaspberryPi Compute Module>`,
:ref:`BW1098FFC <BW1098FFC - USB3 with Modular Cameras>`, :ref:`BW1098OBC <BW1098OBC - USB3 with Onboard Cameras>`, and
:ref:`BW1094 <BW1094 - RaspberryPi Hat>`) has the capability to store the calibration data and field-of-view,
stereo basline (:code:`L-R distance`) and relative location of the color camera to the stereo cameras (:code:`L-RGB distance`)
as well as camera orientation (:code:`L/R swapped`).  To retrieve this information, simply run :code:`python3 depthai_demo.py` and look for
:code:`EEPROM data:`.

Example of information pulled from a :ref:`BW1098OBC <BW1098OBC - USB3 with Onboard Cameras>` is below:

.. code-block::

  EEPROM data: valid (v2)
    Board name     : BW1098OBC
    Board rev      : R0M0E0
    HFOV L/R       : 73.5 deg
    HFOV RGB       : 68.7938 deg
    L-R   distance : 7.5 cm
    L-RGB distance : 3.75 cm
    L/R swapped    : yes
    L/R crop region: top
    Calibration homography:
      1.002324,   -0.004016,   -0.552212,
      0.001249,    0.993829,   -1.710247,
      0.000008,   -0.000010,    1.000000,


Current (those April 2020 and after) DepthAI boards with on-board stereo cameras (:ref:`BW1097 <BW1097 - RaspberryPi Compute Module>`, :ref:`BW1098OBC <BW1098OBC - USB3 with Onboard Cameras>`, and `BW1092 <https://shop.luxonis.com/collections/all/products/bw1092-pre-order>`__) ship calibration and board parameters pre-programmed into DepthAI's onboard eeprom.

Dual-Homography vs. Single-Homography Calibration
#################################################

As a result of some great feedback/insight from the `OpenCV Spatial AI Competition <https://opencv.org/opencv-spatial-ai-competition/>`__ we discovered and implemented many useful features (summary `here <https://github.com/luxonis/depthai/issues/183>`__).

Among those was the discovery that a dual-homography approach, although mathematically equivalent to a single-homography (as you can collapse the two homographies into one) actually outperforms single-homography in real-world practice.  

As a result, we switched our calibration system in September 2020 to use dual-homography instead of single homography.  So any units produced after September 2020 include dual homography.  Any units with single homography can be recalibrated (see :ref:`here <Calibration>`) to use this updated dual-homography calibration.


What is the Field of View of DepthAI and megaAI?
################################################

DepthAI and megaAI use the same 12MP RGB Camera module based on the IMX378.

- 12MP RGB Horizontal Field of View (HFOV): 68.7938 deg
- 1MP Global Shutter Grayscale Cmera Horizontal Field of View (HFOV): 73.5 deg

How Do I Get Different Field of View or Lenses for DepthAI and megaAI?
######################################################################

`ArduCam <https://www.arducam.com/>`__ is in the process of making a variety of camera modules specifically for DepthAI and megaAI, including a variety of M12-mount options (so that the optics/view-angles/etc. are change-able by you the user).

 - M12-Mount IMX378 request `here <https://github.com/luxonis/depthai-hardware/issues/16>`__
 - M12-Mount OV9281 request `here <https://github.com/luxonis/depthai-hardware/issues/17>`__
 - Fish-Eye OV9281 (for better SLAM) request `here <https://github.com/luxonis/depthai-hardware/issues/15>`__
 - Mechanical, Optical, and Electrical equivalent OV9282 module with visible and IR capability `here <https://github.com/luxonis/depthai-hardware/issues/22>`__
 - Global-Shutter Color Camera (OV9782) with same intrinsics as OV9282 grayscale `here <https://github.com/luxonis/depthai-hardware/issues/21>`__
 - Original request for this `here <https://discourse.ros.org/t/opencv-ai-kit-oak/15406/17?u=luxonis-brandon>`__

With these, there will be a variety of options for view angle, focal length, filtering (IR, no IR, NDVI, etc.) and image sensor formats.

.. _maxfps:

What are the Highest Resolutions and Recording FPS Possible with DepthAI and megaAI?
####################################################################################

MegaAI can be used to stream raw/uncompressed video with USB3.  Gen1 USB3 is capable of 5gbps and Gen2 USB3 is capable of 10gbps.
DepthAI and MegaAI are capable of both Gen1 and Gen2 USB3 - but not all USB3 hosts will support Gen2, so check your hosts specifications to see if Gen2 rates are possible.

.. list-table::
  :widths: 33 33 33
  :header-rows: 1
  :align: center

  * - Resolution
    - USB3 Gen1 (5gbps)
    - USB3 Gen2 (10gbps)
  * - 12MP (4056x3040)
    - 21.09fps (390MB/s)
    - 41.2fps (762MB/s)
  * - 4K   (3840x2160)
    - 30.01fps (373MB/s)
    - 60.0fps (746MB/s)

DepthAI and megaAI can do h.264 and h.265 (HEVC) encoding on-device. The max resolution/rate is 4K at 30FPS.
With the default encoding settings in DepthAI/megaAI, this brings the throughput down from 373MB/s (raw/unencoded 4K/30) to
3.125MB/s (h.265/HEVC at 25mbps bit rate).  An example video encoded on DepthAI `BW1097 <https://shop.luxonis.com/collections/all/products/depthai-rpi-compute-module-edition>`__ (Raspberry Pi Compute Module Edition) is below:

.. image:: https://i.imgur.com/uC2sfpj.jpg
  :alt: 4K Video on DepthAI with Raspberry Pi 3B
  :target: https://www.youtube.com/watch?v=ZGERgBTS2T4

It's worth noting that all DepthAI and megaAI products share the same color camera specs and encoding capabilities.  So footage filmed on a DepthAI unit with the color camera will be identical to that taken with a megaAI unit.

Encoded:
  - 12MP (4056x3040) : JPEG Pictures/Stills
  - 4K   (3840x2160) : 30.00fps (3.125MB/s)

How Much Compute Is Available?  How Much Neural Compute is Available?
#####################################################################

DepthAI and megaAI are built around the Intel Movidius Myriad X.  More details/background on this part are `here <https://newsroom.intel.com/wp-content/uploads/sites/11/2017/08/movidius-myriad-xvpu-product-brief.pdf>`__
and also `here <https://www.anandtech.com/show/11771/intel-announces-movidius-myriad-x-vpu>`__.

A brief overview of the capabilities of DepthAI/megaAI hardware/compute capabilities:
  - Overall Compute: 4 Trillion Ops/sec (4 TOPS)
  - Neural Compute Engines (2x total): 1.4 TOPS (neural compute only)
  - 16x SHAVES: 1 TOPS available for additional neural compute or other CV functions (e.g. through `OpenCL <https://docs.openvinotoolkit.org/2020.4/openvino_docs_IE_DG_Extensibility_DG_VPU_Kernel.html>`__)
  - 20+ dedicated hardware-accelerated computer vision blocks including disparity-depth, feature matching/tracking, optical flow, median filtering, harris filtering, WARP/de-warp, h.264/h.265/JPEG/MJPEG encoding, motion estimation, etc.
  - 500+ million pixels/second total processing (see max resolution and framerates over USB :ref:`here <maxfps>`)
  - 450 GB/sec memory bandwidth
  - 512 MB LPDDR4 (contact us for 1GB LPDDR version if of interest)

.. _autofocus:

What Auto-Focus Modes Are Supported? Is it Possible to Control Auto-Focus From the Host?
########################################################################################

DepthAI and megaAI support continuous video autofocus ('2' below, where the system is constantly autonomously
searching for the best focus) and also and :code:`auto` mode ('1' below) which waits to focus until directed by the host.
(PR which adds this functionality is `here <https://github.com/luxonis/depthai/pull/114>`__.)

Example usage is shown in :code:`depthai_demo.py`.  When running :code:`python3 depthai_demo.py` the functionality can be used by keyboard command while the program is running:

 - '1' to change autofocus mode to auto

   - 'f' to trigger autofocus

 - '2' to change autofocus mode to continuous video

 And you can see the reference DepthAI API call `here <https://github.com/luxonis/depthai/blob/3942201d67fe7955370e615aa88045cd8f2211bf/depthai.py#L524>`__

What is the Hyperfocal Distance of the Auto-Focus Color Camera?
###############################################################

The hyperfocal distance is important, as it's the distance beyond which everything is in good focus.  Some refer to this as 'infinity focus' colloquially.

The 'hyperfocal distance' (H) of DepthAI/megaAI's color camera module is quite close because of it's f.no and focal length.

From WIKIPEDIA, `here <https://en.wikipedia.org/wiki/Hyperfocal_distance>`__, the hyperfocal distance is as follows:

.. image:: /_static/images/faq/hyperfocal.png
  :alt: Hyperfocal Distance

Where:

- f = 4.52mm  (the 'effective focal length' of the camera module)
- N = 2.0 (+/- 5%, FWIW)
- c = C=0.00578mm (see `here <https://sites.google.com/site/doftesting/>`__, someone spelling it out for the 1/2.3" format, which is the sensor format of the IMX378)

So H = (4.52mm)^2/(2.0 * 0.00578mm) + 4.52mm ~= 1,772mm, or **1.772 meters** (**5.8 feet**).

We are using the effective focal length, and since we're not optics experts, we're not 100% sure if this is appropriate here,
but the total height of the color module is 6.05mm, so using that as a worst-case focal length, this still puts the hyperfocal distance at **10.4 feet**.

So what does this mean for your application?

Anything further than 10 feet away from DepthAI/megaAI will be in focus when the focus is set to 10 feet or beyond.
In other words, as long as you don't have something closer than 10 feet which the camera is trying to focus on, everything 10 feet or beyond will be in focus.

Is it Possible to Control the Exposure and White Balance and Auto-Focus (3A) Settings of the RGB Camera From the Host?
######################################################################################################################

Auto-Focus (AF)
***************

See :ref:`here <autofocus>` for details on controlling auto-focus/focus.

Exposure (AE)
*************

It is possible to set frame duration (us), exposure time (us), sensitivity (iso) via the API.  And we have a small example for the color camera to show how to do this for the color camera, which is here:
https://github.com/luxonis/depthai/pull/279

We are planning on making these controls more self-documenting (see `here <https://github.com/luxonis/depthai-core/issues/11>`__), but in the meantime, all of the available controls are here:
https://github.com/luxonis/depthai-shared/blob/82435d4/include/depthai-shared/metadata/camera_control.hpp#L107

And for example to set an exposure time of 23.4ms, with the maximum sensitivity of 1600, use:

.. code-block:: python

  self.device.send_camera_control(
     depthai.CameraControl.CamId.RGB,
     depthai.CameraControl.Command.AE_MANUAL,
     "23400 1600 33333")
    
    
White Balance (AWB)
*******************

This will be implemented at the same time as exposure and will be included.  AWB lock, AWB modes.  We will post more information as we dig into this task.

What Are the Specifications of the Global Shutter Grayscale Cameras?
####################################################################

The stereo pair is composed of synchronized global shutter OV9282-based camera modules.

Specifications:
 - Effective Focal Length (EFL): 2.55
 - F-number (F.NO): 2.2 +/- 5%
 - Field of View (FOV):
   - Diagonal (DFOV): 82.6(+/-0.5) deg.
   - Horizontal (HFOV): 73.5(+/-0.5) deg.
   - Vertical (VFOV): 50.0(+/-0.5) deg.
 - Distortion: < 1%
 - Lens Size: 1/4 inch
 - Focusing: Fixed Focus, 0.196 meter (hyperfocal distance) to infinity
 - Resolution: 1280 x 800 pixel
 - Pixel Size: 3x3 micrometer (um)

Am I able to attached alternate lenses to the camera? What sort of mounting system? S mount? C mount?
#####################################################################################################

The color camera on megaAI and DepthAI is a fully-integrated camera module, so the lense, auto-focus, auto-focus
motor etc. are all self-contained and none of it is replaceable or serviceable.  You'll see it's all very small. 
It's the same sort of camera you would find in a high-end smart phone.  

That said, we have seen users attach the same sort of optics that they would to smartphones to widen field of view, zoom, etc. 
The auto-focus seems to work appropriately through these adapters.  For example a team member has tested the
Occipital *Wide Vision Lens* `here <https://store.structure.io/buy/accessories>`__ to work with both megaAI and DepthAI color cameras.
(We have not yet tried on the grayscale cameras.)

Also, see :ref:`below <rpi_hq>` for using DepthAI FFC with the RPi HQ Camera to enable use of C- and CS-mount lenses.

Can I Power DepthAI Completely from USB?
########################################

So USB3 (capable of 900mA) is capable of providing enough power for the DepthAI models.  However, USB2 (capable of 500mA) is not.
So on DepthAI models power is provided by the 5V barrel jack power to prevent situations where DepthAI is plugged into
USB2 and intermittent behavior occurs because of insufficient power (i.e. brownout) of the USB2 supply.

To power your DepthAI completely from USB (assuming you are confident your port can provide enough power), you can use
this USB-A to barrel-jack adapter cable `here <https://www.amazon.com/gp/product/B01MZ0FWSK/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1>`__.
And we often use DepthAI with this USB power bank `here <https://www.amazon.com/gp/product/B0194WDVHI/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1>`__.

.. _virtualbox:

How to use DepthAI under VirtualBox
###################################

If you want to use VirtualBox to run the DepthAI source code, please make sure that you allow the
VM to access the USB devices. Also, be aware that by default, it supports only USB 1.1 devices, and DepthAI
operates in two stages:

#. For showing up when plugged in. We use this endpoint to load the firmware onto the device, which is a usb-boot technique.  This device is USB2.
#. For running the actual code. This shows up after USB booting and is USB3.

In order to support the DepthAI modes, you need to download and install `Oracle VM VirtualBox Extension Pack <https://www.virtualbox.org/wiki/Downloads>`__

.. _parameters_upgrade:

How to increase NCE, SHAVES and CMX parameters?
###############################################

If you want to specify how many Neural Compute Engines (NCE) to use, or how many SHAVE cores, or how many
Connection MatriX blocks, you can do this with the DepthAI.

We have implemented the :code:`-nce`, :code:`-sh` and :code:`-cmx` command line params in our example script. Just clone the
`DepthAI repository <https://github.com/luxonis/depthai>`__ and do

.. code-block:: bash

  ./depthai_demo.py -nce 2 -sh 14 -cmx 14

And it will run the default MobilenetSSD, compiled to use 2 NCEs, 14 SHAVEs and 14 CMXes. Note that
these values **cannot be greater than the ones you can see above**, so you cannot use 15 SHAVEs or 3 NCEs.
14 is the limit for both SHAVE and CMX parameters, and 2 is the limit for NCE.

You can try it out yourself either by following :ref:`local OpenVINO model conversion tutorial <Local OpenVINO Model Conversion>`
or by using our `online MyriadX blob converter <http://69.164.214.171:8083/>`__


.. _rpi_hq:

Can I Use DepthAI with the New RPi HQ Camera?
#############################################

DepthAI FFC Edition (BW1098FFC model `here <https://shop.luxonis.com/products/depthai-usb3-edition>`__) also works via
an adapter board with the Raspberry Pi HQ camera (IMX477 based), which then does work with a ton of C- and CS-mount
lenses (see `here <https://www.raspberrypi.org/blog/new-product-raspberry-pi-high-quality-camera-on-sale-now-at-50/>`__).
And see `here <https://github.com/luxonis/depthai-hardware/tree/master/BW0253_R0M0E0_RPIHQ_ADAPTER>`__ for the adapter board for DepthAI FFC Edition.

.. image:: https://github.com/luxonis/depthai-hardware/raw/master/BW0253_R0M0E0_RPIHQ_ADAPTER/Images/RPI_HQ_CAM_SYSTEM_2020-May-14_08-35-31PM-000_CustomizedView42985702451.png
  :alt: RPi HQ with DepthAI FFC

This is a particularly interesting application of DepthAI, as it allows the RPi HQ camera to be encoded to h.265 4K video (and 12MP stills) even with a Raspberry Pi 1 or :ref:`Raspberry Pi Zero <Can I use DepthAI with Raspberry Pi Zero?>` - because DepthAI does all the encoding onboard - so the Pi only receives a 3.125 MB/s encoded 4K h.265 stream instead of the otherwise 373 MB/s 4K RAW stream coming off the IMX477 directly (which is too much data for the Pi to handle, and is why the Pi when used with the Pi HQ camera directly, can only do 1080p video and not 4K video recording).

Here are some quick images and videos of it in use:

.. image:: https://cdn.hackaday.io/images/9159701591761513514.JPG
  :alt: RPi HQ Camera Support in DepthAI

.. image:: https://cdn.hackaday.io/images/775661591761050468.png
  :alt: RPi HQ Camera Support in DepthAI

.. image:: https://i.imgur.com/AbCHQgW.jpg
  :alt: RPi HQ Camera Support in DepthAI
  :target: https://www.youtube.com/watch?v=KsK-XakrpK8

You can buy this adapter kit for the DepthAI FFC Edition (BW1098FFC) `here <https://shop.luxonis.com/products/rpi-hq-camera-imx477-adapter-kit>`__

.. _rpi_zero:

Can I use DepthAI with Raspberry Pi Zero?
#########################################

Yes, DepthAI is fully functional on it, you can see the example below:


.. image:: /_static/images/faq/pizerosetup.png
  :alt: pizerosetup

.. image:: /_static/images/faq/pizeroruntime.png
  :alt: pizeroruntime

Thanks to `Connor Christie <https://github.com/ConnorChristie>`__ for his help building this setup!

How Much Power Does the DepthAI RPi CME Consume?
################################################

The DepthAI Raspberry Pi Compute Module Edition (RPi CME or BW1097 for short) consumes around 2.5W idle and 5.5W to 6W when DepthAI is running full-out.

- Idle: 2.5W (0.5A @ 5V)
- DepthAI Full-Out: 6W (1.2A @ 5V)

Below is a quick video showing this:

.. image:: https://i.imgur.com/7f6jQ4o.png
  :alt: BW1097 Power Use
  :target: https://www.youtube.com/watch?v=zQtSzhGR6Xg

How Do I Get Shorter or Longer Flexible Flat Cables (FFC)?
##########################################################

 - For the gray scale cameras, we use 0.5mm, 20-pin, same-side contact flex cables.
 - For the RGB camera, we use a 0.5mm 26-pin, same-side contact flex cable.

One can purchase Molex's 15166 series FFCs directly to support shorter or longer lengths.
Make sure you get **same-side** contacts, Molex calls this "**Type A**"

What are CSS MSS UPA and DSS Returned By meta_d2h?
##################################################

- CSS: CPU SubSystem (main cores)
- MSS: Media SubSystem
- UPA: Microprocessor(UP) Array -- Shaves
- DSS: DDR SubSystem

.. _githubs:

Where are the Githubs?  Is DepthAI Open Source?
###############################################

DepthAI is an open-source platform across a variety of stacks, including hardware (electrical and mechanical), software, and machine-learning training using Google Colab.

See below for the pertinent Githubs:

Overall
*******

- https://github.com/luxonis/depthai-hardware - DepthAI hardware designs themselves.
- https://github.com/luxonis/depthai - Python demo and Examples
- https://github.com/luxonis/depthai-python - Python API
- https://github.com/luxonis/depthai-api - C++ Core and C++ API
- https://github.com/luxonis/depthai-ml-training - Online AI/ML training leveraging Google Colab (so it's free)
- https://github.com/luxonis/depthai-experiments - Experiments showing how to use DepthAI.

Embedded Use Case
*****************
- https://github.com/luxonis/depthai-experiments/tree/gen2_common_objdet/gen2-spi - user examples of SPI api and standalone mode.

The above examples include a few submodules of interest. You can read a bit more about them in their respective README files:

- https://github.com/luxonis/depthai-bootloader-shared - Bootloader source code which allows programming NOR flash of DepthAI to boot autonomously
- https://github.com/luxonis/depthai-spi-api - SPI interface library for Embedded (microcontroller) DepthAI application
- https://github.com/luxonis/esp32-spi-message-demo/tree/gen2_common_objdet - ESP32 Example applications for Embedded/ESP32 DepthAI use (e.g. with `BW1092 <https://github.com/luxonis/depthai-hardware/tree/master/BW1092_ESP32_Embedded_WIFI_BT>`__)
 
Can I Use and IMU With DepthAI?
###############################

Yes, our BW1099 (`here <https://shop.luxonis.com/collections/all/products/bw1099>`__) has support to talk to IMUs.  And we are in the process of making a future version of the BW1098OBC (as well as BW1092) which have built-in BNO085.  We do not yet have support for this IMU in the DepthAI API, but we have done proof-of-concepts and will be making this a standard feature through the API.
 
Where are Product Brochures and/or Datasheets?
##############################################

Brochures:
**********

- Editions Summary `here <https://drive.google.com/open?id=1z7QiCn6SF3Yx977oH41Kcq68Ay6e9h3_>`__
- System on Module (BW1099) `here <https://drive.google.com/open?id=1WQMhlh-5Z1YKm4u4i_SVPKxRwgPlfNr8>`__
- USB3 Modular Cameras Edition (BW1098FFC) `here <https://drive.google.com/open?id=1-OWgbJvrdlzRVKtnXDdVKLL9Oba5Nhx3>`__
- USB3 Onboard Cameras Edition (BW1098OBC) `here <https://drive.google.com/open?id=1g0bQDLNnpVC_1-AGaPmC8BaXtGaNNdTR>`__
- Raspberry Pi Compute Edition Module (BW1097) `here <https://drive.google.com/open?id=1QmPQ58NkaxO_Tz1Uzj9LlZcsyZ4Vw7hi>`__
- Raspberry Pi HAT (BW1094) `here <https://drive.google.com/open?id=1QrpV8GXMevqj_ikDJVpaJioXM8axdUEJ>`__
- megaAI (BW1093) `here <https://drive.google.com/open?id=1ji3K_Q3XdExdID94rHVSL7MvAV2bwKC9>`__

Datasheets:
***********

- DepthAI System on Module (BW1099) `here <https://github.com/luxonis/depthai-hardware/blob/master/SoMs/BW1099/BW1099_Datasheet.pdf>`__ 
- PoE Modular Cameras Edition (BW2098FFC) `here <https://drive.google.com/file/d/13gI0mDYRw9-yXKre_AzAAg8L5PIboAa4/view?usp=sharing>`__
 

How can I cite luxonis products in publications ?
##############################################

If DepthAI and OAK-D products has been significantly used in your research and if you would like to acknowledge the DepthAI and OAK-D in your academic publication, we suggest citing them using the following bibtex format.

.. code-block:: latex

  @misc{DepthAI,
  title={ {DepthAI}: Embedded Machine learning and Computer vision api},
  url={https://luxonis.com/},
  note={Software available from luxonis.com},
  author={luxonis},
  year={2020},
  }

  @misc{OAK-D,
  title={ {OAK-D}: Stereo camera with Edge AI},
  url={https://luxonis.com/},
  note={Stereo Camera with Edge AI capabilites from luxonis and OpenCV},
  author={luxonis},
  year={2020},
  }


How Do I Talk to an Engineer?
#############################
 
At Luxonis we firmly believe in the value of customers being able to communicate directly with our engineers.  It helps our engineering efficiency.  And it does so by making us make the things that matter, in the ways that matter (i.e. usability in the right ways) to solve real problems.

As such, we have many mechanisms to allow direct communication:
 - `Luxonis Community Discord <https://discord.gg/EPsZHkg9Nx>`__.  Use this for real-time communication with our engineers.  We can even make dedicated channels for your project/effort public or private in here for discussions as needed.
 - `Luxonis Github <https://github.com/luxonis>`__.  Feel free to make Github issues in any/all of the pertinent repositories with questions, feature requests, or issue reports.  We usually respond within a couple ours (and often w/in a couple minutes).  For a summary of our Githubs, see :ref:`here <Where are the Githubs?  Is DepthAI Open Source?>`.
 - `discuss.luxonis.com <https://discuss.luxonis.com/>`__.  Use this for starting any public discussions, ideas, product requests, support requests etc. or generally to engage with the Luxonis Community.  While you're there, check out this awesome visual-assistance device being made with DepthAI for the visually-impaire, `here <https://discuss.luxonis.com/d/40-questions-re-depthai-usb3-ffc-edition-cameras>`__.



.. include::  /pages/includes/footer-short.rst

