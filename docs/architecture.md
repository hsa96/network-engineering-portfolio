# Architecture

This document describes the architecture of the project, including an overview of the components and how they interact

## Design Goals

- Deploy a management network, using ZTP to bootstrap network devices with an initial configuration (Day 0)

- Use a linux based automation controller with ansible to connect to network devices and provide central, automated configuration of the simulated enterprise network (Day 1)

- To have ongoing control over the network via the central auotmation controller

- Introduce a single source of truth for the network

- Eventually to integrate an AI agent with read access to the network to assist with troubleshooting

- To document the process on this github repo, showing what I have learned and acheiving during this project

## Components

- EVE-NG VM (7.0.1-21-PRO) running on Google Cloud Platform (GCP)

- EVE network devices are currently Cisco vIOS images: vios l2 15.2 (switches), vios 15.9 (routers)


- 'AUTO1' is an Ubuntu 24.04 machine that sits inside the EVE-NG lab