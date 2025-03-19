---
tags:
  - Tools/Docker
type: page
---
Dockerfile
```
FROM python:3.10

# Update the package lists and install required dependencies
# RUN apt-get update && \
#     apt-get install -y python3 python3-pip

# Install requirements for fiftyone
# RUN apt-get update && apt-get install ffmpeg libsm6 libxext6 libcurl4 -y

# Get the current user's name
ARG USER_UID
ARG USER_GID
ARG USERNAME=defaultuser

# Create the user with the same name as the current local user
RUN groupadd -g $USER_GID $USERNAME
RUN useradd -u $USER_UID -g $USER_GID -m $USERNAME

# Switch to the new user
USER $USERNAME

# Install the Python dependencies from the requirements.txt file
# COPY requirements.txt .
# RUN pip3 install --no-cache-dir -r requirements.txt

# Setup sertificates
# RUN export CURL_CA_BUNDLE=/home/${USERNAME}/myCA.crt 
# RUN export REQUESTS_CA_BUNDLE=/app/${USERNAME}/myCA.crt 

# Set the entrypoint to the default Python command
ENTRYPOINT ["/bin/bash"]
```
Build
```
docker build --build-arg USER_UID=$(id -u) --build-arg USER_GID=$(id -g) -t my-image .
```---

