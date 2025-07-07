---
{"publish":true,"title":"Quick Start: Running Moby on Yocto Embedded Device","description":"What is inside docker engine","created":"2025-07-02T11:41:58.749+02:00","modified":"2025-07-07T15:07:06.041+02:00","tags":["docker"],"cssclasses":""}
---


The components and tools in the Moby Project are initially the open source components that Docker and the community have built for the Docker Project. New projects can be added if they fit with the community goals. Docker is committed to using Moby as the upstream for the Docker Product. However, other projects are also encouraged to use Moby as an upstream, and to reuse the components in diverse ways, and all these uses will be treated in the same way. External maintainers and contributors are welcomed.

The Moby project is not intended as a location for support or feature requests for Docker products, but as a place for contributors to work on open source code, fix bugs, and make the code more useful. The releases are supported by the maintainers, community and users, on a best efforts basis only. For customers who want enterprise or commercial support, [Docker Desktop](https://www.docker.com/products/docker-desktop/) and [Mirantis Container Runtime](https://www.mirantis.com/software/mirantis-container-runtime/) are the appropriate products for these use cases.

[Moby](https://mobyproject.org)
[GitHub - moby/moby: The Moby Project - a collaborative project for the container ecosystem to assemble container-based systems](https://github.com/moby/moby

Here’s a practical quick-start guide for running **Moby** (the open-source base for Docker) on a Yocto-based embedded device, running a “Hello World” container both with Moby and with standard Docker, and tips for comparing performance.

---

# Quick Start: Running Moby on Yocto Embedded Device

## Table of Contents

1. [What is Moby?](#what-is-moby)
2. [Prerequisites](#prerequisites)
3. [Building Yocto with Moby Support](#building-yocto-with-moby-support)
4. [Running a Hello World Container with Moby](#running-a-hello-world-container-with-moby)
5. [Running a Hello World Container with Docker](#running-a-hello-world-container-with-docker)
6. [Comparing Performance](#comparing-performance)
7. [References](#references)

---

## What is Moby?

- **Moby** is an open-source project providing the core components for container systems, forming the basis of Docker.
- **Docker** is a product built on top of Moby, adding UX, APIs, and tools.

---

## Prerequisites

- Yocto build environment (Poky or compatible).
- Target device running a Yocto-based Linux image.
- Network access for pulling container images.
- Root access on the device.

---

## Building Yocto with Moby Support

1. **Add meta-virtualization Layer:**
   - Clone the layer:
     ```sh
     git clone https://git.yoctoproject.org/meta-virtualization
     ```
   - Add to your `bblayers.conf`:
     ```
     ${TOPDIR}/../meta-virtualization
     ```

2. **Add Moby to Your Image:**
   - In your image recipe (e.g., `core-image-minimal.bbappend`):
     ```
     IMAGE_INSTALL += "moby-engine"
     ```
   - Or in `local.conf`:
     ```
     IMAGE_INSTALL:append = " moby-engine"
     ```

3. **Build and Deploy:**
   - Build your image:
     ```sh
     bitbake core-image-minimal
     ```
   - Flash the image to your device.

---

## Running a Hello World Container with Moby

1. **Start the Moby Engine (dockerd):**
   ```sh
   systemctl start moby-engine
   # or, if not using systemd:
   dockerd &
   ```

2. **Run Hello World:**
   ```sh
   docker run hello-world
   ```
   - This uses the Docker CLI (provided by `moby-engine`).
   - Output should confirm the container ran successfully.

3. **Alternative: Using Moby Tool Directly**
   - Moby also provides lower-level tools (`containerd`, `ctr`), but for most users, the Docker CLI is the interface.

---

## Running a Hello World Container with Docker

- If you want to compare with “standard Docker”:
  - On a non-Yocto system (e.g., Ubuntu), install Docker:
    ```sh
    sudo apt-get install docker.io
    sudo systemctl start docker
    docker run hello-world
    ```
  - On Yocto, `moby-engine` provides the same CLI and runtime as Docker CE.

---

## Comparing Performance

To compare Moby (on Yocto) and Docker (on another system or Yocto with Docker CE):

1. **Measure Startup Time:**
   ```sh
   time docker run hello-world
   ```

2. **Measure Resource Usage:**
   - Use `top`, `htop`, or `ps` to monitor CPU and memory during container startup and execution.

3. **Repeat for Both Systems:**
   - Run the same commands on both your Yocto/Moby device and a standard Docker system.

4. **Compare Results:**
   - Note differences in startup time, memory footprint, and CPU usage.

**Note:**  
On Yocto, both `moby-engine` and `docker-ce` (if available) use the same core components, so performance differences are usually due to system configuration, not the container engine itself.

---

## References

- [meta-virtualization README](https://git.yoctoproject.org/meta-virtualization/tree/README)
- [Moby Project](https://mobyproject.org/)
- [Docker Hello World Example](https://hub.docker.com/_/hello-world)

---

## Summary Table

| Step                | Moby on Yocto                      | Docker on Standard Linux         |
|---------------------|------------------------------------|----------------------------------|
| Install engine      | Add `moby-engine` to Yocto image   | `apt install docker.io`          |
| Start daemon        | `systemctl start moby-engine`      | `systemctl start