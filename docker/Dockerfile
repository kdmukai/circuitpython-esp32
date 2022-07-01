# Pull base image: Ubuntu v20.x (aka "focal")
FROM arm64v8/ubuntu:focal

# Setup steps from https://learn.adafruit.com/building-circuitpython/linux
RUN apt update
RUN apt install -y software-properties-common
RUN add-apt-repository ppa:pybricks/ppa
RUN apt install -y git gettext uncrustify

# Setup steps from Espressif for ESP32: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html
RUN apt install -y git wget flex bison gperf python3 python3-venv python3-dev cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0

# Get ESP-IDF
WORKDIR /root/home
RUN mkdir -p esp
WORKDIR /root/home/esp
RUN git clone --recursive https://github.com/espressif/esp-idf.git
WORKDIR /root/home/esp/esp-idf
RUN git checkout v4.4.1

WORKDIR /root
RUN curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
RUN python3 get-pip.py
RUN rm get-pip.py

# Set up the tools
WORKDIR /root/home/esp/esp-idf
RUN ./install.sh esp32,esp32s2,esp32s3

WORKDIR /root
RUN git clone https://github.com/adafruit/circuitpython.git circuitpython
WORKDIR /root/circuitpython
RUN git checkout 7.3.1
RUN make fetch-submodules

# Install CircuitPython python dependencies
RUN pip3 install --upgrade -r requirements-dev.txt
RUN pip3 install --upgrade -r requirements-doc.txt

# Build mpy-cross
RUN make -j6 -C mpy-cross

WORKDIR /root/circuitpython/ports/espressif
RUN /bin/bash -c "./esp-idf/install.sh"
RUN /bin/bash -c "source esp-idf/export.sh"

# # Set up the interactive bash shell to our liking
COPY docker/bashrc /root/.bashrc

WORKDIR /root/circuitpython
