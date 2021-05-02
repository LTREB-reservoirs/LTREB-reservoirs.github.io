---
title: "Software Architecture"
permalink: /architecture/
header:
  overlay_color: "#5e616c"
  overlay_image: /assets/images/banner.jpg
---

# Introduction

This document describes the software architecture of FLARE, at a high level - not diving deep into the code, but overviewing the major modules and their interactions. The target audience of this document consists of 1) researchers who are interested in learning the underpinnings of FLARE, 2) prospective forecasters who are interested in deploying FLARE for their own lake, and 3) developers interested in contributing to the FLARE code base.

# Background

The key aspects underpinning the architecture and design of FLARE are:

* Open-source: all FLARE modules are based on open-source software
* End-to-end: FLARE has modules that span computing, networking, and storage resources near sensors in the field (at the network's edge) as well as in the cloud
* Container-based: major FLARE software modules are encapsulated and deployed as [Docker containers](https://docker.com), allowing them to be deployed consistently in any cloud platform that supports Docker
* Event-based automation: for automated forecasting workflows, major FLARE software modules are invoked through the [OpenWhisk](https://openwhisk.apache.org/) event-driven framework - essentially deploying FLARE containers on demand, in response to events such as timers and data availability
* Manual execution possible: while focused on automated forecasting use cases, FLARE containers can also execute in a stand-alone, manual mode for development and testing

# Overview and terminology

FLARE integrates several technologies and standards to deliver a scalable, generalizable end-to-end systems. These include Docker containers, OpenWhisk event-driven processing, Git storage services, sensors, IoT gateways, and network virtualization. To get started, the following diagram provides an overview of the major hardware building blocks of FLARE:

![FLARE building blocks](/assets/images/FLARE_arch_overview_1.png)

FLARE includes components that are deployed both "in the cloud" (compute and storage servers) and at the edge (in the field, physically near sensors at the lake/reservoir being used).

## Edge resources

At the edge, a FLARE deployment has the following hardware building blocks:

* Sensor: sensors collect the various types of data (e.g. meteorology, water temperature, chemistry) that are needed to drive forecasts. Different lakes may have different sets of sensors, and these may be deployed at different sites (e.g. at platform/buoy, or at an inflow)
* Data logger: each sensor must be connected to a data logger that records and buffers sensor data at programmable interval, and multiple sensors may be connected to a single data logger. To work with FLARE, the data logger must have the capability to allow data to be programmatically retrieved from the data logger by an external device (gateway, see below) using a well-defined interface (e.g. FTP over Ethernet/USB). One example of a device that is supported is the [Campbell Scientific CR6 logger](https://www.campbellsci.com/cr6).  
* Gateway: the gateway is a small, field-deployable computer. To work with FLARE, the gateway must have the capability of [running FLARE software (in particular, Ubuntu Linux)](https://github.com/FLARE-forecast/FLARE/wiki/Gateway-Setup), and must be connected to the Internet (typically via a cellular modem). The gateway (along with the data logger) may be powered intermittently when connected to a battery and/or solar panel setup, with the help of a [timer switche](https://smile.amazon.com/JVR-Programmable-Digital-Battery-Powered/dp/B00WR0ELCO/ref=sr_1_1?dchild=1&keywords=timer+switch+coop&qid=1599764243&sr=8-1). An example hardware gateway used in FLARE deployments is the [Fitlet2](https://fit-iot.com/web/products/fitlet2/)
* Cellular modem: to satisfy the requirement of Internet connectivity of the gateway, the typical deployment of FLARE uses a cellular modem attached to the gateway. The modem may be attached physically (e.g. a USB modem) or wirelessly (e.g. a Wi-Fi hotspot). It is key that the cellular signal and service are available at the site where the modem is deployed, and that a data plan is setup. Typical sensors in FLARE do not require large data transfers, but it is key to have stable connectivity to allow for daily updates of data/system logs to be transferred over the network. The modem does not need to expose a public IP address - FLARE can work with private IP addresses by using the EdgeVPN.io virtual network. 

## Cloud resources

In the cloud, a FLARE deployment has the following hardware building blocks:

* Storage server: one or more servers are needed to store data (inputs, working data, and outputs) used by the various FLARE modules
* Compute server: one or more servers are needed to run FLARE processing steps (e.g. data preparation, model execution, data assimilation)

# Major software modules

The following diagram expands on the previous figure to provide an overview of the major software modules that make up the end-to-end FLARE system:

![FLARE software modules](/assets/images/FLARE_arch_overview_2.png)

* Logger scripts: these are deployed on the data logger and/or gateways, and are responsible for implementing a method for retrieving data from the logger by the gateway. For instance, with Campbell data loggers, a script is uploaded to the logger to act as an FTP client, and the gateway is setup as an FTP server
* Git client: this is deployed at gateway(s), and use the Git protocol to push sensor data updates to cloud storage server(s)
* Edge processing: optionally, modules can be deployed on the gateway to perform additional processing at the edge, before data is pushed. Currently, the main edge processing done in FLARE is data preparation for uploads, through Git
* EdgeVPN: this runs an endpoint of the [EdgeVPN.io](https://edgevpn.io) virtual network to provide secure, private connectivity to the gateway
* Git servers: [FLARE uses the Git protocol and storage services](https://github.com/FLARE-forecast/FLARE/wiki/Role-of-Git-in-FLARE) as a foundation to support data movement throughout the end-to-end workflow. This allows flexibility in choosing from one or more Git services to store FLARE data. In one method of FLARE deployment, a public Git server (e.g. [Github](https://github.com)) can be used to store input/output data, and a private Git server (e.g. a [Gitlab-based container](https://github.com/FLARE-forecast/FLARE/wiki/gitlab-server-configuration-and-use)) is used to store intermediate data used by FLARE containers
* OpenWhisk controller: This runs the controller for the OpenWhisk event-driven software, which takes events (such as timers, or notifications that new sensor data is available) to trigger the invocation of various FLARE modules
* Openwhisk invoker: this runs the various FLARE containers that are triggered by the events orchestrated by the controller

# Workflow overview

With the major hardware and software modules as described, the following figure overviews a typical workflow of a FLARE forecast:

![FLARE workflow overview](/assets/images/FLARE_arch_overview_3.png)

* First, data is read from the data logger and buffered on local storage at the gateway, for instance with a daily period
* Then, data is prepared for a Git push; optionally, data is also pre-processed at the edge
* Next, data is pushed over the network to one or more Git servers. This is done securely over ssh (if a public server is used) and also over EdgeVPN (if a private server is used)
* Next, events (such as timers, or a notification of a Git push to a repository) are handled by the OpenWhisk controller, and trigger actions
* Finally, a sequence of actions take place and execute on OpenWhisk invoker nodes, and run the various modules used by FLARE to prepare inputs, assimilate data and run ensemble models to generate forecasts, and prepare data for archival and/or visualization, as shown below:

![FLARE triggers and actions](/assets/images/FLARE_arch_overview_4.png)

# FLARE R package and containers

The core of the FLARE workflow is implemented by the [FLARE R package](https://github.com/rqthomas/flare/), which is deployed across the following containers. 

![FLARE containers](/assets/images/FLARE_arch_overview_5.png)

* flare-download-noaa: downloads weather forecasts from NOAA external Web services
* flare-download-data: downloads sensor data from FLARE Git repositories
* flare-process-observations: process observations to produce driver data for model execution
* flare-generate-forecast: runs ensemble models to generate forecasts. Currently, FLARE supports Ensemble Kalman filtering and the [GLM General Lake Model](https://aed.see.uwa.edu.au/research/models/GLM/)
* flare-visualize: create visualization plots of forecast outputs

# Software repositories and additional documentation

If you would like to learn more details about configuring and deploying FLARE, [please visit the FLARE Wiki](https://github.com/FLARE-forecast/FLARE/wiki)

Our software is distributed along the following repositories:

* [FLARE R package](https://github.com/rqthomas/flare/)
* [FLARE containers](https://github.com/FLARE-forecast/FLARE-containers)




