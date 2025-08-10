# Tetris Game

## Overview
This is a classic Tetris implementation using the `avian19/tetrisv1` Docker image, deployed on our Kubernetes cluster.

## Quick Links
- [Live Game](http://tetris.skylarhoughtongithub.local)
- [Docker Image](https://hub.docker.com/r/avian19/tetrisv1)

## Architecture
The application runs as a containerized React app with the following components:
- Frontend: React-based Tetris game
- Deployment: Kubernetes via ArgoCD
- Ingress: Available at tetris.skylarhoughtongithub.local