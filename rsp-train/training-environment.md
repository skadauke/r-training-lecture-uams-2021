# Training Environment for UAMS lecture 07-14-2021

This documents the steps to set up an RStudio Server Pro training instance for this session.

The goal is to create an RSP instance reachable at <https://uams.r-training.cloud>. We will use the `rsp-train` Docker image and deploy to AWS, as detailed [here](https://github.com/skadauke/rsp-train). 

## Local testing

```bash
# Replace with valid license
export RSP_LICENSE=XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX

docker run --privileged -it \
    -p 8787:8787 \
    -e USER_PREFIX=train \
    -e N_USERS=100 \
    -e PW_SEED=12345 \
    -e GH_REPO=https://github.com/skadauke/r-training-lecture-uams-2021 \
    -e R_PACKAGES=shiny,flexdashboard,plotly \
    -e R_PACKAGES_GH=rstudio/DT \
    -e RSP_LICENSE=$RSP_LICENSE \
    -v "$PWD/server-pro/conf/":/etc/rstudio \
    rsp-train
```