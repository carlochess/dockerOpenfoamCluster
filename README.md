# OpenFoam
OpenFOAM is a free, open source CFD software developed primarily by OpenCFD Ltd since 2004, distributed by OpenCFD Ltd and the OpenFOAM Foundation. It has a large user base across most areas of engineering and science, from both commercial and academic organisations. OpenFOAM has an extensive range of features to solve anything from complex fluid flows involving chemical reactions, turbulence and heat transfer, to acoustics, solid mechanics and electromagnetics.

# Single Instance
Use (http://www.openfoam.com/download/install-binary.php) or you can play with:

```
$ user="$(id -u)"
$ home="${1:-$HOME}"

$ docker run  -it -d --name ofoam \
    --user=${user} \
    -e USER=${USER} -e QT_X11_NO_MITSHM=1 \
    -e DISPLAY=${DISPLAY} --workdir="${home}" \
    --volume="${home}:${home}" \
    --volume="/etc/group:/etc/group:ro" \
    --volume="/etc/passwd:/etc/passwd:ro" \
    --volume="/etc/shadow:/etc/shadow:ro" \
    --volume="/etc/sudoers.d:/etc/sudoers.d:ro" \
    -v=/tmp/.X11-unix:/tmp/.X11-unix \
    carlochess/of_v30plus_rhel66 \
    /bin/bash --rcfile /opt/OpenFOAM/OpenFOAM-v3.0+/etc/bashrc 
```

You can attach whenever you want using 

```
$ xhost +local:ofoam
$ docker start  ofoam
$ docker attach ofoam
```

## Test

```
$ mkdir -p $FOAM_RUN
$ run
$ cp -r $FOAM_TUTORIALS/incompressible/icoFoam/cavity .
$ cd cavity
$ blockMesh
$ icoFoam
$ paraFoam
```

# Local cluster (Docker compose)

Install Docker Compose (https://docs.docker.com/compose/install/)

You can use this docker-compose.yml

```
version: "2"

services:
  mpi_head:
    image: carlochess/of_v30plus_rhel66
    command: "/bin/bash --rcfile /opt/OpenFOAM/OpenFOAM-v3.0+/etc/bashrc"
    user: "mpirun"
    working_dir: /app
    stdin_open: true
    tty: true
    environment:
        - USER="mpirun"
        - QT_X11_NO_MITSHM=1
        - DISPLAY=:0
    volumes:
        - .:/app
```
An then apply this command. Notice that the user in the container is mpirun with
the password mpirun. Also the root user has password mpirun.

```
# Maybe you should run "docker-compose up -d" , first
$ docker-compose scale mpi_head=2
```
Assuming that you run the above command on the folder called openfoam, this is the result

```
Creating and starting openfoam_mpi_head_1 ... done
Creating and starting openfoam_mpi_head_2 ... done
```
You can attach to an instance by running

```
$ docker attach openfoam_mpi_head_1
```

Check connectivity with other containers by

```
$ ping openfoam_mpi_head_2
or
$ ssh openfoam_mpi_head_2
```
If wverything goes well, 
```
$ cp -r /opt/OpenFOAM/OpenFOAM-v3.0+/tutorials/multiphase/interFoam/laminar/damBreak/ .
$ cd damBreak
$ cp -r 0/alpha.water.org 0/alpha.water
$ blockMesh
$ setFields
$ decomposePar
$ mpirun --host openfoam_mpi_head_1 openfoam_mpi_head_2 -np 4 interFoam
```

# Docker compose + Docker Swarm
- You need a docker swarm cluster, use a discovery backend (consul), a docker registry,
- Maybe you need distribuited file system like nfs.
