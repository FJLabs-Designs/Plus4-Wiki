# TMC2240 Chopper Resonance Tuning for X and Y Axes

The QIDI Plus 4 uses [Trinamic TMC2240](https://www.analog.com/en/products/tmc2240.html) stepper motor drivers for the X and Y axes in the default Klipper configuration. However, in order to achieve peak performance, reduce motor noise, and reduce motor vibration, we need to tune our TMC2240 stepper drivers to behave together with the characteristics of the associated motor installed. The TMC2240 comes with a large number of tunables that can be fully in the [Klipper reference](https://www.klipper3d.org/Config_Reference.html#tmc2240). The goal for us is to tune the few most important tunables with the help of the semi-automated [Chopper Resonance Tuner script](https://github.com/MRX8024/chopper-resonance-tuner/tree/main).

This guide is based heavily on the MRX8024 Chopper Resonance Tuning guide, but makes some needed modifications for the purposes of installation into the QIDI Plus 4. The focus on this guide will be tuning the X and Y TMC2240 stepper drivers with the help of the toolhead accelerometer. We will **not** be tuning the Z axis in this guide, as we do not have an accelerometer measuring vibrations of the Z axis motors, nor the extruder. Manual tuning is possible, but not covered under this guide. 

> [!TIP]
> A full Chopper calibration suite across X and Y axes will take upwards of 3+ hours on the Plus4. Most of the process is hands-off, but the printer will not be usable during that window.

## Script Installation

> [!TIP]
> Do not follow the automated installation found in the Chopper Resonance Tuner repository. Certain packages are out of date on the Plus 4's mainboard, and therefore the automated installation will fail. Follow all steps here carefully as doing them improperly may jeopardize your mainboard's OS or stability. 

1. `ssh` into the QIDI Plus 4, following the [typical steps](../ssh-access/readme.md).

```
ssh mks@<ip-address>
```

2. Create a folder for the output files, by default - `mkdir ~/printer_data/config/adxl_results/chopper_magnitude`

3. Clone the Chopper Resonance Tuner repository to your home directory.

```
cd ~

git clone https://github.com/MRX8024/chopper-resonance-tuner
```

4. Create a symbolic link from the repository to your printer config

```
ln -sf ~/chopper-resonance-tuner/chopper_tune.cfg ~/printer_data/config/
```

5. Debian Buster has recently moved its repositories. Run the following to fix up the printer's repo references.

```
sudo sed -i -e '/security\.debian\.org/ s/^deb/#deb/g' -e 's!deb\ http\:\/\/deb\.!deb\ http\:\/\/archive\.!' /etc/apt/sources.list
```

6. Install certain dependencies for the Chopper Resonance Tuner script.

```
sudo apt-get install libatlas-base-dev libopenblas-dev

python3.7 -m pip install --upgrade --user plotly numpy tqdm matplotlib
```

7. Include the following line in your `printer.cfg` file, towards the top. This includes the newly downloaded Chopper Resonance Tuner macros, making them ready for use by Klipper.

```
[include chopper_tune.cfg]
```

8. (Optional) Add an update section to moonraker for subsequent updates via Fluidd update managers. Include the following at the end of your `moonraker.conf` file.

```
[update_manager chopper-resonance-tuner]
type: git_repo
path: ~/chopper-resonance-tuner/
origin: https://github.com/MRX8024/chopper-resonance-tuner.git
primary_branch: main
managed_services: klipper
```

9. Run a Firmware Restart through Fluidd. Once the restart is complete, Chopper Resonance Tuner should now be fully installed and ready for use.