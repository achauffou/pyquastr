# PyQuaStR

This container provides a reproducible development environment that 
features R, Python, Stan, LaTeX, and Quarto. It ships the JupyterLab
and RStudio Server Integrated Development Environments and facilitates
package management.

## Requirements
Apptainer 1.3.1 or any compatible version.

## Quick start
1. Build the container .sif image from the definition file:

   ```
   apptainer build pyquastr.sif pyquastr.def
   ```

2. Start a container instance with the password of your choice:

    ```
   apptainer instance start --writable-tmpfs --env PASSWORD=password pyquastr.sif pyquastr
    ```

3. Open JupyterLab or RStudio Server in your browser:

    <http://127.0.0.1:8888/lab>

    <http://127.0.0.1:8888/rstudio>

4. Log in from your browser with the password you set previously.

## Description
### Base container image
This Apptainer container image is built upon the r-ver Docker image 
from the Rocker Project. The Rocker Project provides Docker container
images for the R environment. Additionally, most features in this
Apptainer image are installed at build time using scripts from the 
Rocker Project.

### List of container features
- R
- RStudio Server
- Pandoc
- Quarto
- TeXlive
- Rust toolchain
- Typst
- Commonly used R publishing packages
- Commonly used R geospatial packages
- Python 3
- JupyterLab
- A virtual environment for Python 3
- CmdStan

### Graphic User Interfaces
PyQuaStR ships two web-based Integrated Development Interfaces: RStudio
Server and JupyterLab. By default, starting an instances exposes them
on the localhost (127.0.0.1) over port 8888.

### Build resource usage
Building the container can take 30+ minutes, require 4+ Go RAM, and 
take 2+ Go of disk space.

### Password restricted access
To enhance security of PyQuaStR instances, access to the web-based IDEs
is restricted by a password. By default, PyQuaStR generates a random 
password when an instance is started, and prints it to the instance 
standard output log (`apptainer instance list --logs <instance name>`).
Alternatively, a custom password can be set through the `PASSWORD` 
environment variable when starting the instance.

### Additional security features
By default, Jupyter Lab runs on the localhost (127.0.0.1) over port 
8888 and restricts remote access. Thus, it can only be accessed from
the localhost, or through an SSH tunnel. It is highly recommended to
keep this security feature enabled as the connection is not encrypted 
with SSL/TLS.

### Bind directories
When running a container, Apptainer can bind locations on the host to
the container. By default, several system-defined bind paths are
defined, such as the home directory of the user, current working
directory, and tmp directory. Users can set up additional bind paths
with the `--bind [host path]:[container path]` option.

### Sandbox mode
During development, it can be useful to build images with the --sandbox
option to create a writable directory instead of an immutable container
image. Sandbox directories behave similarly as images in the 
singularity image format, but can be modified with the --writable 
option. It is recommended to use this option for development purposes 
only, since it decreases reproducibility.

### Overlays
Apptainer container images are read-only by default. However, PyQuaStR
containers require read-write access. Overlaying a filesystem provides
the illusion of read-write access to the immutable container. There are
two different types of overlays:
-   Temporary overlays store all changes in an in-memory temporary 
    filesystem which is discarded as soon as the container finishes 
    executing. They are useful in production contexts as they ensure 
    full reproducibility.
-   Persistent overlays store all changes in a an ext3 filesystem 
    image. Persistent overlays are useful in development contexts as 
    they provide a simple way to install additional dependencies at 
    runtime.

### Persistent overlays
To create a persistent overlay that can store additional dependencies, 
use the command `apptainer overlay create --size <size in Mb> 
--create-dir / pyquastr.img`. Then, attach the persistent overlay to 
the container at run time with the `--overlay pyquastr.img` option.

### Dependencies management
PyQuaStR facilitates the installation and use of apt, TeXlive, python,
and R dependencies. Dependencies respecively stated in apt-deps.txt, 
tex-deps.txt, python-deps.txt, and r-deps are installed at build time. 
When running sandbox directories or images with a persistent overlay, 
it is possible to install additional dependencies at run time with the 
pyquastr-install-deps command or by setting the environment variable 
`PYQUASTR_INSTALL_DEPS=true`. Note that installing apt dependencies 
requires the --fakeroot option.

### Additional fonts
Similarly to dependencies management, PyQuaStR facilitates getting 
additional fonts. It downloads fonts from Debian or Google with the
[fnt](https://github.com/alexmyczko/fnt) package and attempts to 
provide a static version of the font alongside its variable 
counterpart. Similarly to dependencies, fonts stated in fnt-deps.txt 
are installed at build time or at run time if the --fakeroot option 
and the environment variable `PYQUASTR_INSTALL_DEPS=true` are set. You 
can manually search for fonts with the `fnt search <font-name>` 
command. Do not use the font or google prefix in fnt-deps.txt (e.g. 
notosans).

### User configuration files
By default user-specific configuration files for Jupyter Lab and 
RStudio Server are stored in the user home directory. Thus, those
options persist on the host and can be shared across containers. To
isolate those options in the container, it is possible to set the 
`USERCONFIG` environment variable to another location that is not mounted
on the host (e.g. /opt).

## Usage
Build the container .sif image from the definition file:
    
  ```
  apptainer build [build options...] <image path> pyquastr.def
  ```

Start a PyQuaStR instance:

  ```
  apptainer instance start [options...] <image path> <instance name>
  ```

Stop a PyQuaStR instance:
    
  ```
  apptainer instance stop <instance name>
  ```

Run a shell within a PyQuaStR container:
  
  ```
  apptainer shell [options...] <container>
  ```

Execute a command within the container:

  ```
  apptainer exec [options...] <container> <command>
  ```

Run a script within the container:
  
  ```
  apptainer run [options...] <container> <script path>
  ```

Create a persistent image overlay:
  
  ```
  apptainer overlay create --size <overlay size> --create-dir <overlay directory> <overlay path>
  ```

### Arguments
 name | description
---|---
 \<image path\> | a path to an Apptainer image in the singularity image format (.sif) or a sandbox directory (see the sandbox build option) 
 \<instance name\> | a user-defined name for an Apptainer instance (e.g. pyquastr-1) 
 \<container\> | the image path or instance name of the target container 
 \<command\> | a command to run in the container shell 
 \<script path\> | a script to execute in the container 
 \<overlay size\> | size of the image overlay in Mb 
 \<overlay directory\> | a path to the overlay directory in the container 
 \<overlay path\> |  a path to the overlay image on the host 

### Build options
 option | description 
---|---
 -B, --build-arg \<variable=value\> | defines variable=value to replace entries in the build definition file 
 -h, --help | display help
 -F, --force | overwrite an image file if it exists 
 --passphrase | prompt for an encryption passphrase 
 --pem-path \<path\> | enter a path to a PEM formatted RSA key for an encrypted container 
 -s, --sandbox | build image as sandbox format (chroot directory structure) 
 ... | other build options are described in the Apptainer user manual 

### Options
 option | description 
---|---
 -e, --cleanenv | clean the environment before running container 
 -c, --contain | use minimal /dev and empty other directories (e.g. /tmp and $HOME) instead of sharing filesystems from your host 
 -C, --containall | contain not only file systems, but also PID, IPC, and environment 
 --env \<KEY=value\> | pass an environment variable to the contained process 
 --env-file \<file\> | pass environment variables from a file to the contained process 
 -f, --fakeroot | run the container with the appearance of running as root 
 -h, --help | display help 
 -H, --home \<path\> | a home directory specification; it can either be a src path or src:dest pair; src is the source path of the home directory outside the container and dest overrides the home directory within the container 
 --no-eval | do not shell evaluate environment variables or OCI container CMD/ENTRYPOINT/ARGS 
 --no-home | do NOT mount users home directory if /home is not the current working directory 
 -o, --overlay \<overlay path\> | use an overlayFS image for persistent data storage or as read-only layer of container 
 --passphrase | prompt for an encryption passphrase 
 --pem-path \<path\> | enter an path to a PEM formatted RSA key for an encrypted container 
 --unsquash | convert SIF file to temporary sandbox before running 
 -W, --workdir \<path\> | working directory to be used for /tmp, /var/tmp and $HOME (if -c/--contain was also used) 
 -w, --writable | by default all Apptainer containers are available as read only; this option makes the file system accessible as read/write 
 --writable-tmpfs | makes the file system accessible as read-write with non persistent data (with overlay support only) 
 ... | other options are described in the Apptainer user manual 

## Troubleshooting
### No space left on device during the build process
On many linux distributions, the temporary folder is an in-memory `tmpfs` 
filesystem. To prevent the build process to fail because there is no more
space left in the RAM buffer, you can set the temporary folder for Apptainer
to any other system location, such as `/var/tmp` (recommended) with the 
`APPTAINER_TMPDIR` environment variable:

- Using the `-E` option in Apptainer commands:

```
-E APPTAINER_TMPDIR=/var/tmp
```

- By exporting it to your shell variables:

```
export APPTAINER_TMPDIR=/var/tmp
```

## Additional help
The Apptainer user manual provides documentation on Apptainer usage.

<https://apptainer.org/docs/user/main>
    
## Acknowledgement
This container definition file uses a Docker image and shell scripts 
from the Rocker Project. It is implemented using the Apptainer 
container platform (formely Singularity).

## Copyright notice
&copy; Alain Chauffoureaux 2024

This program is free software: you can redistribute it and/or modify it
under the terms of the GNU General Public License as published by the 
Free Software Foundation, either version 3 of the License, or (at your
option) any later version.

This program is distributed in the hope that it will be useful, but 
WITHOUT ANY WARRANTY; without even the implied warranty of 
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU 
General Public License for more details.

You should have received a copy of the GNU General Public License along
with this program. If not, see <https://www.gnu.org/licenses/>.
