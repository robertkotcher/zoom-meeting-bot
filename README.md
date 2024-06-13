# Motivation

I wanted to attend fewer meetings, so I decided to create a bot to inconspicuously join zoom meetings, record the audio, and upload to a google cloud bucket on my behalf. I could then listen later, or summarize with an AI tool.

Secondly, I wanted to break away from Zoom's requirement of asking permission to record, as this could draw too much attention to my absense (as this would be for personal use only).

# Contents

This sample demonstrates how you can run the Zoom Meeting SDK for Linux as a kubernetes job, and
upload the meeting recordings to a google cloud bucket.

It includes the following features:

- Break out of the permissions model for recording the meeting (i.e. the audio will be recorded via a virtual linux device)
- Repackaged the docker contents & run via k8s job, with config.toml read as a secret
- Upload resulting wav file to gcloud bucket via signed upload URL

I have built off of Zoom's [meetingsdk-headless-linux-sample](https://github.com/zoom/meetingsdk-headless-linux-sample)

# Setup

## 0. Prerequisites

1. [Docker](https://www.docker.com/)
2. [Zoom Account](https://support.zoom.us/hc/en-us/articles/207278726-Plan-Types-)
3. [Zoom Meeting SDK Credentials](#config:-sdk-credentials) (Instructions below)
    1. Client ID
    2. Client Secret
4. Kubernetes cluster set up with kubectl
5. Google cloud project and `gcloud` CLI

## 1. Clone the Repository

```bash
# Clone down this repository
git clone git@github.com:zoom/meetingsdk-headless-linux-sample.git
```

## 2. Download the Zoom Linux SDK

Download the latest version of the Zoom SDK for Linux from the Zoom Marketplace and place it in
the [lib/zoomsdk](lib/zoomsdk) folder of this repository.

## 3. Configure the Bot

If you don't already have them, follow the section on how
to [Get your Zoom Meeting SDK Credentials](#get-your-zoom-meeting-sdk-credentials).


### Copy the sample config file

```bash
cp sample.config.toml config.toml
```

### Fill out the config.toml

Here, you can set any of the CLI options so that the bot has them available when it runs. Start by adding your Client ID and Client Secret in the relevant fields.

**At a minimum, you need to provide an Client ID and Client Secret along with information about the meeting you would like to join.**

You can either provide a Join URL, or a Meeting ID and Password.

### Base64 encode the config.toml

`cat config.toml | base64`

Then copy this output into `config-toml-secret.template.yaml` and remove "template" from the name.

### Apply the secret

`kubectl apply -f config-toml-secret.yaml`

## 4. Upload image to registry

First, build the image with `docker build -f Dockerfile.k8s -t gcr.<REGISTRY_PATH> .`.

Then, upload to google cloud registry: `docker push gcr.<REGISTRY_PATH>`

Finally, update `zoom-bot-job.template.yaml` with image name `gcr.<REGISTRY_PATH>`.

## 5. Get signed bucket URL

`gcloud storage signed-url generate gs://your-bucket-name/your-object-name --duration=1h`

# Get meeting credentials and join meeting

The invite link and audio upload path can be provided via environment variables:

```
JOIN_URL=<zoom invite link>
SIGNED_URL=<signed url from step 5>
```