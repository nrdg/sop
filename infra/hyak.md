# Using Hyak

## Logging in

To remotely access Hyak:

`ssh [UW Net ID]@klone.hyak.uw.edu`

You'll be prompted to authenticate.

## Starting a session

In order to load miniconda and do other intensive tasks, you will need to start a session. For example:

`srun -p gpu-a40 -A escience --nodes=1 --ntasks-per-node=4 --time=2:00:00 --mem=120G --pty /bin/bash`

More information on flags can be found [here](https://wiki.cac.washington.edu/display/hyakusers/Mox_scheduler)

## Loading modules

Once resources have been allocated and the session begins, you can perform computational tasks. For example:

`module load ssmc/miniconda`
`conda activate my_environment`
`python my_script.py`

You may want to load other modules, such as
`module load apptainer`

More information on modules can be found [here](https://hyak.uw.edu/docs/tools/modules/)

## Submitting jobs to slurm

You can also submit a task to be queued and executed through slurm. 

The status of these jobs, as well as interactive sessions, can be seen with:
`squeue -u [UW Net ID]`

## Managing your disk space 

The home directory you begin in after logging in has limited space. More (but not persistent) space can be accessed by moving large files, caches, and working directories to /gscratch. For example:

`cd /gscratch/escience/`
`mkdir [UW Net ID]`

Any command with a cache or working directory flag should probably be pointed to a location in /gscratch, or a memory error may result. 
