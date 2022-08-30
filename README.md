# minion_docker

---

**Warning!**: Run a sequencing simulation to test your setup before committing to
a true sequencing experiment. See [Testing](#Testing) for some instructions.

**Note**: The `ont-minknow` container runs currently as a *privileged* container.
Moreover the containers were not configured and tested under user namespace isolation.

---

## Quick setup

1. Install the Kingfisher UI electron application on the host system.

   Depending on the distribution,you may download and install the DEB package directly
   from Oxford Nanopore repository, preferably using your system package manager.
   For Arch Linux, use the provided PKGBUILD in the
   [`ont-kingfisher-ui-minion` directory](./ont-kingfisher-ui-minion/). Install only
   the package version compatible with the MinION Software release version. To check for
   the appropriate version consult the
   [Nanopore Community release notes](https://community.nanoporetech.com/downloads/minion_release/release_notes)
   or the provided [PKGBUILD](./ont-kingfisher-ui-minion/PKGBUILD) in this repository.
   The direct link to the package can be extracted from the above-mentioned
   [PKGBUILD](./ont-kingfisher-ui-minion/PKGBUILD).

2. (*Optional for GPU accelerated basecalling with Guppy Docker container*)
Install the recommended
[NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-docker).

   For instructions consult the
   [official NVIDIA documentation](https://docs.nvidia.com/ai-enterprise/deployment-guide/dg-docker.html)
   or the Arch Linux specific [ArchWiki entry](https://wiki.archlinux.org/title/Docker#Run_GPU_accelerated_Docker_containers_with_NVIDIA_GPUs).

3. Install `docker-compose`.

4. Configure the Guppy basecalling parameters in
the `ont-guppy-gpu` [Dockerfile](./ont-guppy-gpu/Dockerfile) by changing the appropriate
environmental variables.

5. Run `docker-compose build`.

6. Run `docker-compose up`.

7. Run the Kingfisher application (MinKNOW) on the host system.

8. Access the run data.

   By default the MinKNOW data directory (`/var/minknow/data/`) will be mounted to
   the local `./minknow_data` folder. If you store your run results there, they will be
   persistent and accessible outside of the running container.

   Alternatively specify another volume in the `docker-compose.yml` and point
   the software there. **NOTE**: If you decide to do this, the directory structure
   of the host must currently mirror the container directory structure
   as the Kingfisher UI operates outside of the `ont-minknow` container without the
   knowledge of it's directory tree.

## Component description

What follows in this section is the short description of the running components
and their configuration.

### 1. Kingfisher UI

Kingfisher is the electron application providing the graphical user interface for the
MinKNOW sequencing software. It is being run on the host system. The application
authenticates with the MinKNOW software running in a Docker container with a token in
a `/tmp/minknow-auth-token.json` file on a shared `/tmp` volume. It communicates through
the exposed Docker ports.

**NOTE**: The user interface does not have access to the file-system inside the docker
container. That means, when you would like to change the default file paths through
the interface, it is easier if the container volume mount points mirror directly
the host director structure.

By default the MinKNOW data directory: `/var/lib/minknow/data/` is mounted on the host
to `./minknow_data/` local directory. Change the mount point at your convenience,
without changing the default file path in the GUI.

### 2. Guppy basecaller

The Guppy basecaller is run inside a separate NVIDIA CUDA Docker container. To run the
container configured NVIDIA graphics driver is required, but CUDA packages should
not be.

The basecaller server communicates with the MinKNOW software through a Unix
socket on the shared Docker volume named `guppy-socket`.

The basecaller is configured through the `ENV` variables defined in the Dockerfile.
Customize the variables to match your graphics card capabilities.

The provided Dockerized basecaller might quite simply be replaced either by a local
Guppy installation (replace the shared `guppy-socket` volume with
`/tmp/.guppy/:/tmp/.guppy/` volume mount) or a container running CPU version of the
basecaller.

Basecaller logs are by default stored in the local `./minknow_log/guppy/` directory.

### 4. MinKNOW Software suite

MinKNOW software runs in a privileged container with `/dev/bus/usb` mounted to provide
access to the host USB interface for communication with MinION device.

The software communicates with Guppy basecaller through Unix socket on the shared volume
named `guppy-socket` mounted on the `/tmp/.guppy/` directory.

The container to host `/tmp` directory volume mapping is needed for
the `minknow-auth-token.json` to be shared between the container and
the Kingfisher UI application. Additional ports are exposed for inter-process
communication.

By default the MinKNOW data directory: `/var/lib/minknow/data/` is mounted on the host
to `./minknow_data/` local directory.

The logs are by default stored in the local `./minknow_log/minknow` directory.

## Testing

1. Download an open access
[bulk FAST5 file](http://s3.amazonaws.com/nanopore-human-wgs/bulkfile/PLSP57501_20170308_FNFAF14035_MN16458_sequencing_run_NOTT_Hum_wh1rs2_60428.fast5).

2. In the `docker-compose.yml` file add an entry in the `volumes` section
of the `ont-minknow` service to mount the directory containing your downloaded
bulk fast5 file in the container.

   Example:

   ```yml
   services:
       .
       .
       .
       ont-minknow:
       .
       .
       .
           volumes:
               .
               .
               .
               - /home/user/Downloads:/mnt/data
   ```

3. To configure the software to use the bulk fast5 for the simulation,
insert before the `EXPOSE` directive in `ont-minknow/Dockerfile`:

   ```docker
   RUN sed -E -i \
       -e '/\[custom_settings\]/a simulation = "/container/path/to/your/bulk.fast5"' /opt/ont/minknow/conf/package/sequencing/sequencing_MIN106_DNA.toml
   ```

   Remember to change the `"/docker/path/to/your/bulk.fast5"` to the full path where
   you have mounted the file in the container (from the example above it would be
   `/mnt/data/bulk.fast5`). Adjust the TOML configuration file to your needs.

4. Run `docker-compose build` and `docker-compose up`.

Proceed to plug in your MinION device. Run the simulated sequencing experiment
and inspect the results.

If the results are correct, remove the changes and rerun the `docker-compose build`
process.
