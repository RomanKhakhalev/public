# RPi5 Google Coral TPU
# credos to Agajan for providing initial manual for setting up the drivers


## Python environment
1. Create Python virtual environment with all default packages 
```bash
python3 -m venv --system-site-packages venvs/robominder
```

2. Activate Python virtual environment:
```bash
source venvs/robominder/bin/activate
```

## Coral TPU installation

### Install libedgetpu
```bash
wget https://github.com/feranick/libedgetpu/releases/download/v16.0TF2.15.1-1/libedgetpu1-max_16.0tf2.15.1-1.bookworm_arm64.deb
sudo dpkg -i libedgetpu1-max_16.0tf2.15.1-1.bookworm_arm64.deb
```

### Install gasket
This step usually causes some problems. Please troubleshoot manually if this step fails by any reason.
```bash
sudo apt install curl devscripts debhelper dh-dkms -y
git clone https://github.com/google/gasket-driver.git
cd gasket-driver; debuild -us -uc -tc -b; cd ..
sudo dpkg -i gasket-dkms_1.0-18_all.deb
sudo depmod -a
sudo modprobe apex
```

### Add 'kernel=kernel8.img' to the boot configuration
```bash
echo "kernel=kernel8.img" | sudo tee -a /boot/firmware/config.txt
```

### Modify the Device Tree Blob (DTB)
```bash
sudo cp /boot/firmware/bcm2712-rpi-5-b.dtb /boot/firmware/bcm2712-rpi-5-b.dtb.bak 
sudo dtc -I dtb -O dts /boot/firmware/bcm2712-rpi-5-b.dtb -o ~/test.dts 
sudo sed -i '/pcie@110000 {/,/};/{/msi-parent = <[^>]*>;/{s/msi-parent = <[^>]*>;/msi-parent = <0x67>;/}}' ~/test.dts
sudo dtc -I dts -O dtb ~/test.dts -o ~/test.dtb 
```

### Apex permissions
```bash
sudo sh -c "echo 'SUBSYSTEM==\"apex\", MODE=\"0660\", GROUP=\"apex\"' >> /etc/udev/rules.d/65-apex.rules"
sudo groupadd apex
sudo adduser $USER apex
```

### Reboot
```bash
sudo reboot now
```

### Verify the accelerator module is detected, output should be like 03:00.0 System peripheral: Device 1ac1:089a
```bash
lspci -nn | grep 089a
```

### Verify PCIe driver is loaded, output the name repeated back
```bash
ls /dev/apex_0
```




## Added by Roman //23.02.25
## Installing pyenv
```bash
curl https://pyenv.run | bash
```

## Installing missing libraries for 3.9.15
```bash
sudo apt-get install libbz2-dev
sudo apt-get install libncurses5-dev libncursesw5-dev
sudo apt-get install libffi-dev
sudo apt-get install libreadline-dev
sudo apt-get install libssl-dev
```
## Installing pyenv 3.9.15
```bash
pyenv install 3.9.15
```

## Installing pycoral
```bash
python3 -m pip install --extra-index-url https://google-coral.github.io/py-repo/ pycoral~=2.0
```

## Downgrade numpy
```bash
python3 examples/classify_image.py \
--model test_data/mobilenet_v2_1.0_224_inat_bird_quant_edgetpu.tflite \
--labels test_data/inat_bird_labels.txt \
--input test_data/parrot.jpg
```

# Installation is complete 
### To check follow the instruction on coral's official webpage - below as was of 23th Feb 25
```bash
mkdir coral && cd coral
git clone https://github.com/google-coral/pycoral.git
cd pycoral
bash examples/install_requirements.sh classify_image.py
python3 examples/classify_image.py \
--model test_data/mobilenet_v2_1.0_224_inat_bird_quant_edgetpu.tflite \
--labels test_data/inat_bird_labels.txt \
--input test_data/parrot.jpg
```
