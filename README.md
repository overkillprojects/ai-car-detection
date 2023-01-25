# Renesas RZ/V2L Object Detection Project - Detecting Cars

## Hardware Requirements

- Renesas [RZ/V2L Evaluation Board Kit](https://www.renesas.com/eu/en/products/microcontrollers-microprocessors/rz-mpus/rzv2l-evkit-rzv2l-evaluation-board-kit)
- If you plan on remote access to the board, as I did here, you will
need a router that allows port-forwarding.
- Possibly an enclosure, depending on where you are placing your board.

## Software Requirements

- Edge Impulse account
- Yocto build for the board (via the instructions from Renesas and Edge Impulse).
- A terminal with `ssh`.


## First Steps

Before starting this project, I wanted to familiarize myself with the Edge
Impulse platform. I had very good luck trying out an image classification example
as a warm-up. If you want something smaller to try out before you get into
object detection on this board, I recommend that you follow
[this guide](https://docs.edgeimpulse.com/docs/tutorials/image-classification)
for a smooth experience to get you some practice and insight.

## 1. Set up and connected the Renesas RZ/V2L board to the Edge Impulse Project

<!-- I don't know how you got the port for the ssh command and I think it should be included here, along with the creation of a user -->

The [setup instructions](https://docs.edgeimpulse.com/docs/development-platforms/officially-supported-cpu-gpu-targets/renesas-rz-v2l)
for the board went relatively smoothly. The only
issues I ran into were that I didn't notice that I had to apply the
`v3.0.0-update2` patch to the Renesas VLP/V package at first, and I didn't
notice until I had already done a few `bitbake`s that I needed to add a few
things to the configuration for Edge Impulse, specifically the `nodejs`
and `npm` packages and updating `glibc`. But they are all in the documentation,
so it shouldn't be a problem for you if you read more carefully than I do.

After the board was prepared, I forwarded it from port 2461 through my router
and connected to it with `ssh` using the correct port and the user that I created:

```sh
ssh -p2461 myuser@68.81.34.204
```

To connect to the device, you may need to enter a password for you user depending
on if you have one or not. If you are using the `root` user like they do in the
Edge Impulse guide for the RZ/V2L board, then you won't need to bother with
`sudo`ing the commands below.

First I needed to set up `npm` to run as the `root` user and to install
the [`edge-inpulse-linux`](https://github.com/edgeimpulse/edge-impulse-linux-cli) package.

```sh
sudo npm config set user root
sudo npm install edge-impulse-linux -g --unsafe-perm
```

Now that everything is setup and ready to go, we run the Edge Impulse runner.

```ssh
sudo edge-impulse-linux-runner
```

You'll have to sign into your EI account to access your models.

If that all worked out for you, you're set! You connected to the board and
you're ready to move onto the other steps.

## 2. Reformatted the CSV for a new dataset into JSON for Edge Impulse to accept it

Because I have limited access to the physical board, which is shared with other
students and this project is only a proof of concept, I decided to train my
model using a public dataset, in this case a car detection dataset
from [kaggle](https://www.kaggle.com/). However, the bounding box data for my
dataset came as a `CSV`, where Edge Impulse expects a
[specifically formatted](https://docs.edgeimpulse.com/docs/edge-impulse-cli/cli-uploader#bounding-boxes)
`JSON` file. Not to be deterred, I decided to create
[a script](https://github.com/ajaxromik/boundingBoxesCSVToJSON/blob/main/converter.py)
to convert the data to the correct format using
[Python](https://www.python.org/downloads/)

Now as long as your CSV file is formatted the same way as mine, the script
will create a `bounding_boxes.labels` file for you.

## 3. Prepared the model and trained it

All that is left is to test the model and load it onto the board.

```sh
sudo edge-impulse-linux-runner --download downloaded-model.eim
```

### Worth mentioning:

Normally the settings to run the training would have me do 60 epochs, but it goes over the allotted usage for a developer's project.
I was able to get it to run with only 38 epochs even though I would have done at least 80 if it was available.
