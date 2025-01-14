# Kokkos Training 2025 at Barcelona

## SSH in the Ruche local cluster

```sh
ssh login@ruche.mesocentre.universite-paris-saclay.fr
cd $WORKDIR
```

For Windows users without SSH, use KiTTY: https://github.com/cyd01/KiTTY/releases

## Clone the repo of the training

```sh
git clone --recurse-submodules \
          https://github.com/CExA-project/cexa-kokkos-tutorials.git
```

## Load modules and set environment variables

```sh
module load gcc/11.2.0/gcc-4.8.5 \
            cmake/3.28.3/gcc-11.2.0 \
            cuda/12.2.1/gcc-11.2.0
export OMP_PROC_BIND=spread \
       OMP_PLACES=threads
```

## Ruche documentation

Ruche documentation: https://mesocentre.pages.centralesupelec.fr/user_doc/
Ruche CPU model: Intel Xeon Gold 6230 (Cascade Lake)
Ruche GPU model: NVIDIA V100

## Submit jobs on Ruche

Ruche is a cluster using Slurm as job submission system.
You should not run executables on the login node, beyond compilers and usual tasks.

### Using one-liner commands

These commands allow to run one executable in a CPU or a GPU job, and redirect its standard output/error to the screen, just like you were running it live.

Run on CPU:

```sh
srun --partition cpu_short --cpus-per-task 20 --pty path/to/exe
```

Run on GPU:

```sh
srun --partition gpu --gres gpu:1 --pty path/to/exe
```

### Using a script (alternative)

Alternatively to the one-liner commands provided above, you can also use submission scripts.

Script for CPU:

```sh
!/bin/bash

#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=20
#SBATCH --mem=10G
#SBATCH --time=00:10:00
#SBATCH --partition=cpu_short
#SBATCH --job-name=kt25-exercise-cpu
#SBATCH --output=%x.%J.out
#SBATCH --error=%x.%J.err
#SBATCH --export=NONE
#SBATCH --propagate=NONE

module load gcc/11.2.0/gcc-4.8.5

export OMP_PROC_BIND=spread \
       OMP_PLACES=threads

path/to/exe
```

Script for GPU:

```sh
!/bin/bash

#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --mem=10G
#SBATCH --time=00:10:00
#SBATCH --partition=gpu
#SBATCH --gres gpu:1
#SBATCH --job-name=kt25-exercise-gpu
#SBATCH --output=%x.%J.out
#SBATCH --error=%x.%J.err
#SBATCH --export=NONE
#SBATCH --propagate=NONE

module load gcc/11.2.0/gcc-4.8.5 \
            cuda/12.2.1/gcc-11.2.0

path/to/exe
```

Submit your job:

```sh
sbatch my_script.sh
```

See [the documentation](https://mesocentre.pages.centralesupelec.fr/user_doc/ruche/06_slurm_jobs_management/) for how to manage your Slurm jobs.

## CMake commands for Ruche

Build with the OpenMP backend:

```sh
cmake -B build_openmp \
      -DCMAKE_BUILD_TYPE=Release \
      -DKokkos_ENABLE_OPENMP=ON
cmake --build build_openmp \
      --parallel 5
```

Build with the Cuda backend:

```sh
cmake -B build_cuda \
      -DCMAKE_BUILD_TYPE=Release \
      -DKokkos_ENABLE_CUDA=ON \
      -DKokkos_ARCH_VOLTA70=ON
cmake --build build_cuda \
      --parallel 5
```
