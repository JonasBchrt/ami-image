# ami-image

*Author: Jonas Beuchert*

*Date: October 2024*

This Github action uses [pi-gen](https://github.com/RPi-Distro/pi-gen) to build a Raspberry Pi OS image in several stages.
In addition to the default Raspberry Pi OS stages and configuration, it does the following:
* Keeps `pi` as the default user account with password `raspberry`.
* Enables SSH access.
* Names the OS image `ami-image` and the release `AMI OS`.
* Sets up the AMI-specific software by adding an additional stage `ami-stage` at the end of the workflow. (See [below](#ami-stage).)
* Uploads the OS image as a downloadable artifact to Github.

The action automatically uses the third-party Github actions [pi-gen-action](https://github.com/usimd/pi-gen-action) and [upload-artifact](https://github.com/actions/upload-artifact).

*Note:* As of now, you still need to copy any models for on-board classification manually. See the last step in the readme in the `pi_inferences` repository for details.

## ami-stage

The custom AMI stage does the following (cf. [blank.yml](.github/workflows/blank.yml)):

* Create an additional directory `ami-stage` with sub-directory `ami`: `mkdir -p ami-stage/ami`.
* Create a shell script `00-run-chroot.sh` in the `ami` directory, which is to be executed by root and write to it: `cat > ami-stage/ami/00-run-chroot.sh`.
* The following lines go into the script:
1) Install the version control system git: `apt install -y git`.
2) Use git to clone the desired AMI-System repository from Github: `git clone -b jonas-dev https://github.com/AMI-system/pi_inferences.git`. *Modify this if you want to build an OS image based on a different branch or a different repository.*
3) Switch into the cloned directory: `cd pi_inferences`. *Modify this if your repository has a different name.*
4) Make the cloned installation script executable: `chmod +x install.sh`. *Modify this if the installation script in your branch/repository has a different name.*
5) Run the installation script: `./install.sh`. *Modify this if the installation script in your branch/repository has a different name.*
* Make the created shell script executable: `chmod +x ami-stage/ami/00-run-chroot.sh`.
* Indicate that the stage exports and OS image: `touch ami-stage/EXPORT_IMAGE`.

## Runners

Run the action either on a Github hosted runner by choosing `runs-on: ubuntu-latest` or on a self-hosted runner with a Debian-based OS, e.g., Ubuntu, by choosing `runs-on: self-hosted`.
However, the memory limit of the free Github hosted runner does not allow to build an OS image with a graphical user interface and, hence, the stage list must be set to `stage-list: 'stage0 stage1 stage2 ami-stage'`.
If using a self-hosted runner with sufficient memory, you can build an operating system with graphical user interface by setting the stage list to `stage-list: 'stage0 stage1 stage2 stage3 stage4 ami-stage'`.

When using a self-hosted runner your probably need to install docker, set the environment variable `RUNNER_ALLOW_RUNASROOT` to `"1"` (`export RUNNER_ALLOW_RUNASROOT="1"`), and (after configuration) run the runner with `sudo -E bash -c './run.sh'` (to pass the environment variable on).

## Proposed AMI OS development workflow

1. Clone Github repository

    ↓

2. Make change locally

    ↓

3. Push to Github

    ↓

4. Build OS image in the cloud

    ↓

5. Download OS image

    ↓

6. Decompress OS image

    ↓

7. Flash OS image on SD card
