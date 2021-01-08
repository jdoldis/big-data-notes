---
title: Rasberry Pi Cluster
---

{% include toc.html %}

## Why
To do distributed computing you need multiple computers. A cheap option for building a cluster is to use Raspberry Pis.

## Equipment
<table>
    <tr>
        <th>Quantity</th>
        <th>Item</th>
        <th>Amazon Link</th>
    </tr>
    <tr>
        <td>3-4</td>
        <td>Raspberry Pi</td>
        <td><a href="https://www.amazon.com.au/gp/product/B07TC2BK1X/ref=ppx_yo_dt_b_asin_title_o07_s00?ie=UTF8&psc=1">Raspberry Pi</a></td>
    </tr>
    <tr>
        <td>1 per pi</td>
        <td>Micro SD Card</td>
        <td><a href="https://www.amazon.com.au/gp/product/B06XWMQ81P/ref=ppx_yo_dt_b_asin_title_o06_s01?ie=UTF8&psc=1">SD Card</a></td>
    </tr>
    <tr>
        <td>1 per pi</td>
        <td>USB A to USB C Cables</td>
        <td><a href="https://www.amazon.com.au/gp/product/B07D78C55Y/ref=ppx_yo_dt_b_asin_title_o06_s01?ie=UTF8&psc=1">Cable Pack</a></td>
    </tr>
    <tr>
        <td>1 per pi</td>
        <td>Ethernet Cable</td>
        <td><a href="https://www.amazon.com.au/gp/product/B004C4U3MU/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1">Ethernet Cable</a></td>
    </tr>
    <tr>
        <td>1</td>
        <td>USB Wall Charger</td>
        <td><a href="https://www.amazon.com.au/gp/product/B00P936188/ref=ppx_yo_dt_b_asin_title_o06_s00?ie=UTF8&psc=1">Wall Charger</a></td>
    </tr>
    <tr>
        <td>1</td>
        <td>Cluster Case</td>
        <td><a href="https://www.amazon.com.au/gp/product/B07MW24S61/ref=ppx_yo_dt_b_asin_title_o05_s00?ie=UTF8&psc=1">Cluster Case</a></td>
    </tr>
    <tr>
        <td>1</td>
        <td>Micro SD Card Reader</td>
        <td><a href="https://www.amazon.com.au/Sandisk-SDDR-B531-GN6NN-MobileMate-microSD-Reader/dp/B07G5JV2B5/ref=sr_1_2?dchild=1&keywords=micro+sd+card+reader&qid=1594035429&sr=8-2">Reader</a></td>
    </tr>
</table>

The ethernet cables aren't required if you just want to use wifi, but wired connections are more reliable. If you do order them  be sure to check that your router has sufficient ethernet ports. If not, look into getting an ethernet switch.

## Photos
[![Cluster Pic 1]({{site.baseurl}}/assets/images/cluster1.jpg)]({{site.baseurl}}/assets/images/cluster1.jpg)
[![Cluster Pic 2]({{site.baseurl}}/assets/images/cluster2.jpg)]({{site.baseurl}}/assets/images/cluster2.jpg)
[![Cluster Pic 3]({{site.baseurl}}/assets/images/cluster3.jpg)]({{site.baseurl}}/assets/images/cluster3.jpg)

## Setup
There's already a ton of guides about how to set up Raspberry Pis so I'll keep this very high level:
1. Format the SD Card: This involves putting the operating system onto the SD card. Essentially you just download the raspbian OS and write it to the card. You'll need a micro SD card reader to do that. See [the docs](https://www.raspberrypi.org/documentation/installation/) for more details.
1. Configure the pi: It's easiest to do this using the GUI. This requires a keyboard, mouse and monitor (or TV) to be plugged into the pi. If you can't do this look into a [headless setup](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md).
