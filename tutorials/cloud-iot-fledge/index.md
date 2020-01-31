---
title: Using Fledge with IoT Core on Google Cloud
description: Learn how to process sensor data with Fledge.
author: gguuss
tags: Cloud IoT Core, Gateways, Raspberry Pi, Python, MQTT, internet of things, Fledge, FogLAMP, Dianomic
date_published: 2020-01-31
---

Gus Class | Developer Programs Engineer | Google Cloud IoT Core

Fledge (previously known as *FogLAMP*) is an open-source framework contributed by Dianomic
for getting data from sensors into data stores. Fledge provides a bridge between sensors
that communicate using the *South* service to Google Cloud as a data store using the
*North* service.

To see an architecture diagram that illustrates the use of Fledge in the context
of Google Cloud IoT Core, see the
[Fledge documentation](https://fledge-iot.readthedocs.io/en/latest/fledge_architecture.html).
That architecture diagram shows HTTPS traffic to a cloud platform; for IoT Core, message
traffic can also use the MQTT protocol.

For this tutorial, we use a Raspberry Pi to host the Fledge service.

## Set up your Raspberry Pi with the Raspbian Buster image

This tutorial requires that you use the Debian Buster version of the Raspberry
Pi software because it uses packages built for that version of Linux.

1.  Download a Raspbian Buster image from the
    [Raspbian downloads page](https://www.raspberrypi.org/downloads/raspbian/).

1.  Extract the image that you downloaded, which creates a file with a name that ends
    in `.img`.
    
    For example, if you download the full version of Raspbian, the file is
    named `2019-09-26-raspbian-buster-full.img`.

1.  Create the SD card image for your Raspberry Pi using a disk image utility.

    Raspberry Pi recommends that you use [balenaEtcher](https://www.balena.io/etcher/).

1.  Flash your Raspberry Pi using the disk image that you created.

    For information about flashing your Raspberry Pi, see the detailed instructions in the
    [Raspberry Pi installation documentation](https://www.raspberrypi.org/documentation/installation/installing-images/README.md).

1.  Set up access to the Raspberry Pi. 

    *   If you intend to access the Raspberry Pi in headless mode, follow the instructions for
        [setting up headless](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md)
        on the Raspberry Pi site.
    *   If you intend to access your Raspberry Pi locally, connect it to a keyboard,
        mouse, and display.

1.  Do any final configuration of the Raspberry Pi—such as expanding the filesystem and
    setting the user password—with the
    [raspi-config](https://www.raspberrypi.org/documentation/configuration/raspi-config.md)
    utility.

You perform subsequent steps on the Raspberry Pi itself either over SSH if you're working in
headless mode or on the Raspberry Pi hardware if you're using the desktop UI.

## Download the Fledge packages

1.  Visit the
    [download page on the Dianomic site](https://dianomic.com/download-packages/)
    and fill out the form to get an email message with a link to the latest version
    of the Fledge packages.
    
1.  When you receive the email message containing the link to the Fledge packages,
    copy the link referencing v1.7 for ARM-based devices (Buster), the package
    for Raspberyy Pi devices.

1.  From your Raspberry Pi, use the following commands to download the Fledge
    packages to the file system and place them in a folder named `foglamp` in your
    user's home directory. Replace `[LINK_FROM_EMAIL]` with the link that you copied
    in the preivous step.

        mkdir $HOME/foglamp
        cd $HOME/foglamp
        wget [LINK_FROM_EMAIL]
        tar -xzvf foglamp-1.7.0_armv7l_buster.tgz

    The `foglamp` folder in your user's home directory contains all the resources
    required for getting your Raspberry Pi up and running.

## Install the Fledge service and Fledge GUI on your Raspberry Pi

The Fledge graphical user interface (GUI) makes it easier to configure and
control Fledge from your Raspberry Pi.

1.  Browse to the folder that you downloaded the Fledge packages to, and install
    the Fledge and Fledge GUI packages:

        cd $HOME/foglamp/foglamp/1.7.0/buster/armv7l/
        sudo apt -y install ./foglamp-1.7.0-armv7l.deb
        sudo apt -y install ./foglamp-gui-1.7.0.deb

1.  Start the Fledge service and check its status:

        export FOGLAMP_ROOT=/usr/local/foglamp
        $FOGLAMP_ROOT/bin/foglamp start
        $FOGLAMP_ROOT/bin/foglamp status

## Open the Fledge GUI

To open the Fledge GUI, you use your web browser to go to the web server that
started running on the Raspberry Pi when you installed the Fledge GUI.

1.  To determine the IP address of your server, run the `ifconfig` command:

        ifconfig

    The output looks similar to the following:

        wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
                inet 192.168.1.217  netmask 255.255.255.0  broadcast 192.168.1.255
                inet6 fe80::9f73:3b56:93c2:4ed4  prefixlen 64  scopeid 0x20<link>
                ether dc:a6:32:03:b7:cb  txqueuelen 1000  (Ethernet)
                RX packets 426133  bytes 305815327 (291.6 MiB)
                RX errors 0  dropped 0  overruns 0  frame 0
                TX packets 274458  bytes 172317386 (164.3 MiB)
                TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

    In this example, the IP address is `192.168.1.217`.
    
1.  Use your web browser to go to the URL for the IP address from the previous step,
    such as this URL using the value from the preivous example:
    
        http://192.168.1.217

    When you navigate to the web server, you see a dashboard similar to the following:

    ![Fledge GUI dashboard](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/foglamp-gui.png)

## Install a Fledge South plugin for generating data

In this tutorial, you install a *South* plugin for generating and getting data and
a *North* plugin for publishing data to Google Cloud.

One quick way to get some data is with the "randomwalk" plugin.

1.  Install the `foglamp-south-randomwalk` plugin:

        cd $HOME/foglamp/foglamp/1.7.0/buster/armv7l
        sudo apt install ./foglamp-south-randomwalk-1.7.0-armv7l.deb

1.  In the left pane of the Fledge GUI, click **South**, and then click the **Add +** button
    in the upper-right corner of the **South Services** window:

    ![Add button](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/foglamp-south-add.png)

1.  Select the `randomwalk` plugin, give your plugin a name (for example, `random`), and
    click **Next**.

    ![Add random plugin](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/foglamp-south-add-random.png)

1.  Click **Next** again on the next screen, and then click **Done**.

1.  In the left pane, click **Assets & Readings**.

    You should see random data generated by the South plugin that you just configured.

    ![Fledge Assets & Readings graph](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/foglamp-south-readings.png)

## Install the Fledge North plugin for IoT Core

Now that you have data being generated by the South plugin, you can
publish that data to Google Cloud through the IoT Core device bridge. 

1.  Install the Fledge North plugin for Google Cloud IoT Core (GCP North plugin):

        cd $HOME/foglamp/foglamp/1.7.0/buster/armv7l
        sudo apt install ./foglamp-north-gcp-1.7.0-armv7l.deb

1.  In the left pane of the Fledge GUI, click **North**, and then click the
    **Create North Instance +** button in the upper-right corner of the
    **North Instances** window:

    ![Create North instance](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/foglamp-north-add.png)

1.  Select the `GCP` plugin, give your instance a name (for example, `GCP`), and
    click **Next**.

## Create a registry

1.  Go to the [IoT Core page in the Cloud Console](https://console.cloud.google.com/iot).

1.  At the top of the page, click **Create registry**.

1.  Enter a registry ID (for example, `foglamp`).

1.  Choose a region (for example, `us-central1`).

1.  Select an existing telemetry topic or create a new topic (for example,
    `projects/[YOUR_PROJECT_ID]/topics/foglamp`).
    
1.  Click **Create**.

    ![Create registry form](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/foglamp-north-create-registry.png)

    After the registry is created, the registry overview page opens.
    
## Create and configure a device for communicating with IoT Core
    
In this section, you create a device that will be used to transmit data to IoT
Core. Before you can register a device, you need to create a key pair used
to authenticate the device.

1.  Create an RSA public/private key pair as described in the
    [IoT Core documentation](https://cloud.google.com/iot/docs/how-tos/devices):

        cd $HOME/foglamp
        openssl genpkey -algorithm RSA -out rsa_private.pem -pkeyopt rsa_keygen_bits:2048
        openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem

1.  Show the contents of the public key:

        cat rsa_public.pem

1.  In the left pane of the IoT Core page in the Cloud Console, click **Devices**.

1.  At the top of the **Devices** page, click **Create a device**.

    ![Create device screen](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/foglamp-north-create-device.png)

1.  Enter a device ID (for example, `foglamp`).

1.  Make sure the public key format matches the type of key that you created in the
    first step of this section (for example, `RS256`).
    
1.  Paste the contents of your public key in the **Public key value** field.

1.  Click **Create**.

    ![Add the device screen](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/foglamp-north-create-device-contents.png)

## Configure the North plugin to communicate using your device

First, you will need to copy the
device private key to the Fledge certificate store as well as the root
certificate for Google Cloud IoT core.

```
wget https://pki.goog/roots.pem
cp roots.pem /usr/local/foglamp/data/etc/certs/
cp rsa_private.pem /usr/local/foglamp/data/etc/certs/
```

Now that the device has been created in the registry, you will need to
return to the Fledge GUI screen to configure the North plugin. In the
Review Configuration screen, input your Project ID from the Google Cloud IoT
Core console (<your-project-id>, the registry name you used (foglamp), the
device ID (foglamp), the key name that was created (rsa_private), the JWT
algorithm (RS256), and the data source which is typically readings.

![Fledge North GCP plugin configuration](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/foglamp-north-configure-gcp.png)

After you click Next, ensure the plugin is enabled on the following screen and
click Done.

Now, if you return to the dashboard, the count of readings Sent should be
incrementing while the Received readings increase. You can also navigate to the
Cloud IoT Core Console page for your device and see that telemetry events were
being received from the device.

![Cloud IoT Core Device page](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/cloud-iot-console-device-telemetry.png)

Congratulations, you've now setup Fledge to publish data generated on the
Raspberry Pi! To see the telemetry messages, create a subscription to the
PubSub subscription configured in your registry by first clicking on the link
in the Registry Details page.

![Cloud IoT Registry details](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/cloud-iot-registry-page.png)

Clicking Create Subscription.

![Create Subscription page](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/cloud-iot-pubsub-create-subscription.png)

And finally, entering the subscription details such as the name for your
subscription (e.g. foglamp) before clicking the Create button.

![Input Subscription details page](https://storage.googleapis.com/gcp-community/tutorials/cloud-iot-fledge/cloud-iot-pubsub-create-subscription-details.png)

With the subscription created, you can see the messages published by Fledge
using the [Google Cloud SDK](https://cloud.google.com/sdk). The following
command lists the messages published to the foglamp subscription.

```
gcloud pubsub subscriptions pull --limit 500 foglamp
```

The output will have JSON data corresponding to the generated data.

```
┌───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┬─────────────────┬────────────────────────────────────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    DATA                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   │    MESSAGE_ID   │             ATTRIBUTES             │                                                                                           ACK_ID                                                                                           │
├───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼─────────────────┼────────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ {"randomwalk" : [ {"ts":"2020-01-08 22:22:23.126610","randomwalk" : 100},{"ts":"2020-01-08 22:22:24.126691","randomwalk" : 100},{"ts":"2020-01-08 22:22:25.126676","randomwalk" : 100},{"ts":"2020-01-08 22:22:26.126688","randomwalk" : 99},{"ts":"2020-01-08 22:22:27.126671","randomwalk" : 98}]} │ 928125343880036 │ deviceId=foglamp                   │ IT4wPkVTRFAGFixdRkhRNxkIaFEOT14jPzUgKEUXAQgUBXx9d0FPdV1ecGhRDRlyfWBzPFIRUgEUAnpYURgEYlxORAdzMhBwdWB3b1kXAgpNU35cXTPyztSQrLSyPANORcajjpomIZzZldlsZiQ9XxJLLD5-NSxFQV5AEkw-GURJUytDCypYEU4EIQ │
│                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           │                 │ deviceNumId=2550637435869530       │                                                                                                                                                                                            │
│                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           │                 │ deviceRegistryId=foglamp           │                                                                                                                                                                                            │
│                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           │                 │ deviceRegistryLocation=us-central1 │                                                                                                                                                                                            │
│                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           │                 │ projectId=glassy-nectar-370        │                                                                                                                                                                                            │
│                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           │                 │ subFolder=                         │                                                                                                                                                                                            │
└───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┴─────────────────┴────────────────────────────────────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

Now that you've seen how the plugin works end-to-end, you can delete the North
instance to prevent data from continuing to be published by navigating to the
North menu on the Fledge GUI, selecting your instance of the GCP plugin, and
then clicking the Delete instance button.

## Next steps

Fledge provides a robust solution for getting data into Google Cloud Platform
using Cloud IoT Core. You can readily migrate the data from PubSub to persistent
data stores such as:

* [Google Cloud BigQuery](https://cloud.google.com/bigquery/docs)
* [Google Cloud SQL](https://cloud.google.com/sql/docs)
* [Google Cloud Spanner](https://cloud.google.com/spanner/docs)

and can analyze the data using Google Cloud Analytics products.

You can also evaluate other South plugins such as the [SenseHat](https://github.com/foglamp/foglamp-south-sensehat)
plugin which transmits gyroscope, accelerometer, magnetometer, temperature,
humidity, and barometric pressure.

You can also look into the hardware partners for more robust and secure hardware
solutions. The following reference hardware solutions are available from Nexcom:

* [NISE50](http://www.nexcom.com/Products/industrial-computing-solutions/industrial-fanless-computer/atom-compact/fanless-nise-50-iot-gateway)
* [NISE105](http://www.nexcom.com/Products/industrial-computing-solutions/industrial-fanless-computer/atom-compact/fanless-computer-nise-105)
* [NISE3800](http://www.nexcom.com/Products/industrial-computing-solutions/industrial-fanless-computer/core-i-performance/fanless-pc-fanless-computer-nise-3800e)

Finally, you can look into advanced usage of the Fledge platform through
features such as
[filters](https://foglamp.readthedocs.io/en/master/foglamp_architecture.html#filters),
[events](https://foglamp.readthedocs.io/en/master/foglamp_architecture.html#event-engine),
and API access to device data with
[applications](https://foglamp.readthedocs.io/en/master/foglamp_architecture.html#rest-api).