---
permalink: /
title: "FLARE"
excerpt: "Forecasting Lake And Reservoir Ecosystems"
header:
  overlay_color: "#5e616c"
  overlay_image: /assets/images/banner.jpg
  caption: "Falling Creek Reservoir, Virginia, USA"
---
The FLARE project creates open-source software for flexible, scalable, robust, and near-real time iterative ecological forecasts in lakes and reservoirs. FLARE is composed of water temperature and meteorology sensors that wirelessly stream data, a data assimilation algorithm that uses sensor observations to update predictions from a hydrodynamic model and calibrate model parameters, and an ensemble-based forecasting algorithm to generate forecasts that include uncertainty.

![FLARE workflow overview](/assets/images/flare-workflow.jpeg)

The FLARE open-source platform integrates edge and cloud computing open-source software frameworks: Docker containers for microservice deployment, Apache OpenWhisk for orchestration of event-driven actions, Git and EdgeVPN.io for edge-to-cloud transfers and remote management, and S3 for cloud staging and storage. The numerical forecasting core uses the General Lake Model and data assimilation based on the ensemble Kalman filter (EnKF).
