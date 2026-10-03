# Intelligent but Offline Smart-Doorbell

EC-ENG 535/635 Networked Embedded Systems Design, Fall 2026
Team: Ojas Singhvi and Munir Safwan

This repo is for our course project. Right now it has our project proposal. We'll add our code, results, and final report here as the project goes on.

## Motivation

Most smart doorbells like Ring or Nest send everything they record to the cloud, and the person detection and face recognition happen on the company's servers. That means videos of your house, your family, and anyone who comes to your door are stored on someone else's server. It also means the "smart" features stop working if the internet goes down, and a lot of them need a monthly subscription.

We think a doorbell shouldn't need the cloud for this. Small boards like the Raspberry Pi can now run lightweight neural networks (like MobileNet) on their own, so we want to build a doorbell that does everything locally. It should detect when someone is at the door, figure out if it's someone in the household or a stranger, and send an alert to our phones over the home Wi-Fi without sending anything to the internet.

We picked this project because we both use smart home devices and wanted to build something we'd actually use. We also liked that it covers the whole embedded system and not just the ML part. We have to set up the hardware, get models running on a small device, and make the device talk to other devices on the network. We're also curious how much accuracy we lose when the models have to be small and fast enough for a Raspberry Pi.

## Design Goals

1. **Works offline.** All the processing happens on the Raspberry Pi. Nothing gets sent to the cloud.
2. **Detects people.** The doorbell should notice when a person shows up at the door. We're aiming for around 90% precision and recall on test clips we record ourselves.
3. **Recognizes known people.** It should tell the difference between people in our "household" and unknown visitors. We're aiming for around 85% accuracy.
4. **Fast enough to be useful.** The alert should show up within about 2 seconds of someone appearing at the door.
5. **Small models.** The models need to fit and run on the Pi, so we're keeping the total model size under about 20 MB.
6. **Local alerts.** Alerts get sent to a phone or laptop on the same Wi-Fi network.
7. **Easy to add new people.** Adding a new household member should only take a few photos and no retraining.
8. **Stretch goal:** detect if the visitor is a delivery person.

These numbers are our starting targets. We'll adjust them once we see how the Pi actually performs.

## Deliverables

- A working doorbell prototype on a Raspberry Pi with a camera that runs inference on the device itself
- Code for the full pipeline: motion trigger, person detection, face detection, and known/unknown face matching
- A script to enroll new household members by taking a few photos of them
- Alerts sent over the local network (using MQTT) and a simple local web page that shows recent visitors with a snapshot and time
- An evaluation of the system covering accuracy, latency, and memory use, plus a comparison of at least two model options (for example a regular model vs. a quantized one)
- Stretch goal: a delivery person classifier trained on Google Colab and run on the Pi
- A final demo and report with graphs of our results

## System Blocks

```
  [Camera] ----+
               |
  [Doorbell    +--> [Motion / Button Trigger]
   Button]                  |
                            v
                  [Person Detection]
                            |
                            v
                   [Face Detection]
                            |
                            v
                   [Face Embedding] <---- [Saved embeddings of
                            |               household members]
                            v
                 [Known or Unknown?]
                            |
                            v
                    [Event Logger]
                   (snapshot + time)
                       /        \
                      v          v
             [MQTT Alert]    [Local Web Page]
                      \          /
                       v        v
               [Phone / Laptop on home Wi-Fi]

  Everything above the phone runs on the Raspberry Pi.
  Google Colab is only used ahead of time for training/converting models.
```

How it works:

- **Camera and trigger:** The camera is always on at low resolution. When it notices motion, or someone presses the doorbell button, it starts the rest of the pipeline. That way the heavier models don't have to run on every single frame.
- **Person detection:** A small pretrained object detector (SSD MobileNet or EfficientDet-Lite) checks if there's actually a person in the frame.
- **Face detection and embedding:** If there's a person, we find their face and turn it into an embedding, which is basically a list of numbers that describes the face.
- **Known or unknown:** We compare that embedding to the saved embeddings of the people we've enrolled. If it's close enough to one of them, it's a known person. If not, it's an unknown visitor. This is the approach suggested in the project description, and it means we don't need to retrain anything to add a new person.
- **Event logger:** Saves a snapshot, the result, and the time to a log file on the Pi.
- **Networking:** The Pi runs an MQTT broker (Mosquitto) and publishes alerts that our phones or laptops can subscribe to. It also hosts a small Flask web page on the local network. None of this needs internet, only the home Wi-Fi.

## Hardware and Software Requirements

**Hardware** (we plan to borrow these from the department):

- Raspberry Pi 4 (4 GB or more) or Raspberry Pi 5
- Raspberry Pi Camera Module 3 or v2 (a USB webcam would also work)
- microSD card, 32 GB or more
- Raspberry Pi power supply
- A push button and jumper wires for the doorbell button
- Optional: PIR motion sensor, buzzer or LED, and a case with a heatsink or fan
- Our laptops and phones for development and receiving alerts

**Software:**

- Raspberry Pi OS (64-bit)
- Python 3
- Picamera2 for the camera
- OpenCV for image processing and face detection
- TensorFlow Lite (now called LiteRT) for running the models on the Pi
- NumPy for comparing embeddings
- Mosquitto and paho-mqtt for the alerts
- Flask for the local web page
- gpiozero for the button and sensors
- Google Colab with TensorFlow for any training and for converting models to TFLite
- GitHub for version control

Models we're looking at: SSD MobileNet or EfficientDet-Lite for person detection, BlazeFace or OpenCV's YuNet for face detection, and MobileFaceNet for face embeddings.

## Team Members and Responsibilities

We're both going to work on every part of the project, but each of us is leading certain areas.

**Munir Safwan** is leading:
- **Setup:** getting the hardware, setting up the Pi and Raspberry Pi OS, wiring the camera and button, and installing everything
- **Software:** the main program on the Pi that connects all the pieces, plus logging and the enrollment script
- **Networking:** the MQTT alerts, the local web page, and making sure everything works with no internet

**Ojas Singhvi** is leading:
- **Research:** reading papers on small CNNs and face recognition, picking which models to try, and deciding how we'll evaluate the system
- **Algorithm design:** the detection and recognition pipeline, tuning the matching threshold, comparing models, and the stretch delivery person classifier
- **Writing:** the proposal, the documentation in this repo, check-in updates, and the final report

Munir has more experience with Linux, so it made sense for him to take the Pi and networking side. Ojas is taking the ML and writing side. We'll test the full system and do the final demo together.

## Project Timeline

We have about 10 weeks from the proposal to the final demo, which is in week 13 of the course. We kept the weeks around the midterm presentations (course weeks 7 and 8) lighter on project work. We also want the system working by Nov 30 so we have a buffer week before the demo.

- **Week 1 (Oct 5 - 11):** Get the hardware, set up the Pi, get the camera taking pictures, and start reading about models
- **Week 2 (Oct 12 - 18):** Get a pretrained person detector running on the Pi and measure how fast it runs
- **Week 3 (Oct 19 - 25):** Add face detection and embeddings, write the enrollment script, and take our own test photos
- **Week 4 (Oct 26 - Nov 1):** Get known/unknown matching working and add the motion and button trigger. The goal is to have the basic pipeline working from start to finish.
- **Week 5 (Nov 2 - 8):** Set up MQTT alerts and event logging with snapshots
- **Week 6 (Nov 9 - 15):** Build the local web page and test everything with the internet turned off. This should be ready to show at a check-in.
- **Week 7 (Nov 16 - 22):** Evaluate accuracy, latency, and memory, and compare different model options
- **Week 8 (Nov 23 - 29):** Thanksgiving week, so a lighter week. We'll work on the delivery person stretch goal if we're on track.
- **Week 9 (Nov 30 - Dec 6):** Fix bugs, test in different lighting and distances, and practice the demo
- **Week 10 (Dec 7 - 11):** Final demo, results graphs, final report, and clean up the repo

If the final dates change once they're posted, we'll update this timeline.

**Things that could go wrong and our backup plans:**
- If getting the hardware takes a while, we'll start writing and testing the code on our laptops with a webcam and move it to the Pi later.
- If the models run too slowly on the Pi, we'll try smaller or quantized models, lower the image resolution, or only run detection when there's motion.
- If face recognition isn't accurate enough, we'll use more photos per person, tune the threshold, and average results over a few frames.

## References

1. A. G. Howard et al., "MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications," arXiv:1704.04861, 2017. https://arxiv.org/abs/1704.04861
2. M. Sandler et al., "MobileNetV2: Inverted Residuals and Linear Bottlenecks," CVPR, 2018. https://arxiv.org/abs/1801.04381
3. W. Liu et al., "SSD: Single Shot MultiBox Detector," ECCV, 2016. https://arxiv.org/abs/1512.02325
4. F. Schroff, D. Kalenichenko, and J. Philbin, "FaceNet: A Unified Embedding for Face Recognition and Clustering," CVPR, 2015. https://arxiv.org/abs/1503.03832
5. S. Chen et al., "MobileFaceNets: Efficient CNNs for Accurate Real-Time Face Verification on Mobile Devices," arXiv:1804.07573, 2018. https://arxiv.org/abs/1804.07573
6. V. Bazarevsky et al., "BlazeFace: Sub-millisecond Neural Face Detection on Mobile GPUs," arXiv:1907.05047, 2019. https://arxiv.org/abs/1907.05047
7. Google AI Edge, LiteRT (TensorFlow Lite) documentation. https://ai.google.dev/edge/litert
8. Raspberry Pi Ltd., The Picamera2 Library manual. https://pip-assets.raspberrypi.com/categories/1044-raspberry-pi-ai-camera/documents/RP-008156-DS-3-picamera2-manual.pdf
9. OASIS, MQTT Version 5.0 specification, 2019. https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html
10. Eclipse Mosquitto MQTT broker. https://mosquitto.org/
