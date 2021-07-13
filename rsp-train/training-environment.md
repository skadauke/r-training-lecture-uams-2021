# Training Environment for UAMS lecture 07-14-2021

This documents the steps to set up an RStudio Server Pro training instance for this session.

The goal is to create an RSP instance reachable at <https://uams.r-training.cloud>. We will use the `rsp-train` Docker image and deploy to AWS, as detailed [here](https://github.com/skadauke/rsp-train).

## Local testing

```bash
# Replace with valid license
export RSP_LICENSE=XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX

docker run --privileged -it \
    -p 8787:8787 \
    -e USER_PREFIX=uams \
    -e N_USERS=30 \
    -e PW_SEED=12345 \
    -e GH_REPO=https://github.com/skadauke/r-training-lecture-uams-2021 \
    -e R_PACKAGES=rmarkdown \
    -e RSP_LICENSE=$RSP_LICENSE \
    -v "$PWD/rsp-train/custom-login":/etc/rstudio \
    skadauke/rsp-train
```

### Deploy to AWS

- Registered `r-training.cloud` using AWS Route 53 (Personal AWS)
- Created public `t2.micro` instance for testing running Ubuntu 20.04 LTS
  - For the real thing, will create public `c5a.2xlarge` instance (CHOP AWS), $0.31/hr
- Configured A record sending `uams.r-training.cloud` to the public IP of the instance (Personal AWS)

```bash
ssh -i ~/.ssh/cgt.pem ubuntu@uams.r-training.cloud
```

```bash
sudo apt update &&

sudo ufw allow OpenSSH &&
sudo ufw enable
```

```bash
sudo apt install nginx &&
sudo ufw allow 'Nginx Full' &&
sudo systemctl start nginx &&
sudo systemctl enable nginx
```

```
MY_DOMAIN=uams.r-training.cloud &&

sudo snap install --classic certbot &&
sudo ln -s /snap/bin/certbot /usr/bin/certbot &&

sudo certbot --nginx -d $MY_DOMAIN
```

```
sudo apt update &&
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release &&

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg &&

echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null &&

sudo apt update &&
sudo apt install docker-ce docker-ce-cli containerd.io
```

```
# Replace with valid license
export RSP_LICENSE=XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX &&

GH_REPO=https://github.com/skadauke/r-training-lecture-uams-2021 &&

git clone $GH_REPO &&

cd r-training-lecture-uams-2021 &&

sudo docker run --privileged -it \
    --detach-keys "ctrl-a" \
    --restart unless-stopped \
    -p 8787:8787 -p 5559:5559 \
    -e USER_PREFIX=uams \
    -e GH_REPO=$GH_REPO \
    -e R_PACKAGES=rmarkdown \
    -e N_USERS=30 \
    -e PW_SEED=12345 \
    -e RSP_LICENSE=$RSP_LICENSE \
    -v "$PWD/rsp-train/custom-login/":/etc/rstudio \
    skadauke/rsp-train
```