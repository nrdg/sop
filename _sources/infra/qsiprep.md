# Running qsiprep on Hyak

Running qsiprep with Docker is not available on Hyak, but the Docker container can be converted to Apptainer (formerly known as singularity) with the 'apptainer build' command. 

The Apptainer container can then be run on Hyak with:
`apptainer run --mount type=bind,src=/gscratch/escience/[UW Net ID]/[BIDS dir],dst=/data,ro --mount type=bind,src=/gscratch/escience/[UW Net ID]/[BIDS dir]/derivatives,dst=/out /gscratch/escience/[UW Net ID]/qsiprep.sif /data /out participant --fs-license-file /data/fs_license.txt [flags] -w /scr`
`apptainer exec --mount type=bind,src=/gscratch/escience/[UW Net ID]/[BIDS dir],dst=/data,ro --mount type=bind,src=/gscratch/escience/[UW Net ID]/[BIDS dir]/derivatives,dst=/out /gscratch/escience/[UW Net ID]/qsiprep.sif qsiprep /data /out participant --fs-license-file /data/fs_license.txt [flags] -w /scr`
`singularity run --mount type=bind,src=/gscratch/escience/[UW Net ID]/[BIDS dir],dst=/data,ro --mount type=bind,src=/gscratch/escience/[UW Net ID]/[BIDS dir]/derivatives,dst=/out /gscratch/escience/[UW Net ID]/qsiprep.sif /data /out participant --fs-license-file /data/fs_license.txt [flags] -w /scr`
`singularity exec --mount type=bind,src=/gscratch/escience/[UW Net ID]/[BIDS dir],dst=/data,ro --mount type=bind,src=/gscratch/escience/[UW Net ID]/[BIDS dir]/derivatives,dst=/out /gscratch/escience/[UW Net ID]/qsiprep.sif qsiprep /data /out participant --fs-license-file /data/fs_license.txt [flags] -w /scr`

In this example, the BIDS directory and qsiprep.sif container have been placed in /gscratch/escience/[UW Net ID] and fs_license.txt has been placed in the BIDS directory. 

## Notes
-w /scr has been used to specify the working directory as /scr
`exec` requires `qsiprep` to be specified after the container, but `run` does not
The derivatives directory must exist before running the command

## Output
To capture and separate the output, one option is to run the command within a Python script:
```
import subprocess
import os.path as op

log_path = "/gscratch/escience/[UW Net ID]/qsiprep_log.txt"
data_path = "/gscratch/escience/[UW Net ID]/[BIDS dir]"
out_path = op.join(data_path, "derivatives")

with open(log_path, "wb") as f:
    p = subprocess.Popen(
        f"apptainer run --mount type=bind,src={data_path},dst=/data,ro --mount type=bind,src={out_path},dst=/out {sif_path} /data /out participant --fs-license-file /data/fs_license.txt [flags] -w /scr",
        shell=True,
        stdout=f,
    )

 p.wait()
 ```

