We use Datalad to do provenance tracking for large-scale data processing. 

Penn LINC has their workflow "The Way" documented out [here](https://pennlinc.github.io/docs/TheWay/RunningDataLadPipelines/), and similarly [here] (http://handbook.datalad.org/en/latest/beyond_basics/101-170-dataladrun.html) in the Datalad Handbook. This documentation is meant to provide an extension to modifying this general workflow for your BIDSAPP, data, and cluster, as well as smooth over some problems you might run into. This is adapted from a pyAFQ workflow on Stanford's Sherlock cluster, but can easily be translated to most BIDSAPPS.

Within "The Way", you set up your workflow by running a "bootstrap script" that creates your code and sets up your datalad infrastructure. Typically, starting from one of the Penn LINC scripts is ideal. It might be worth asking Matt and/or Sydney to see if they have any suggestions for which script would be easiest to adapt to your use case, but it depends on how close your container workflow is to another, and idiosyncrasies of your compute cluster. 

### Setup.
Datalad itself can be installed with `pip install datalad` or `conda install -c conda-forge datalad`. 

To get a boostrap script, you can clone The Way github repo (link). When you run the boostrap script, it will set up the boostrap directory in at the same level as the script, which can be annoying if you're in a Github repo, so you can avoid this by hard link the specific bootstrap script that you want (`ln path/to/target path/to/source`) to a different directory where you want your Datalad work directory to live. As results are created, the directory created by this script can get very large, so OAK or SCRATCH might be a good place.

If you're getting the data from another Datalad workflow, it'll likely be saved as a RIA store and already be a datalad dataset. If not, you'll need to set up your BIDS directory as a [datalad dataset] (https://handbook.datalad.org/en/latest/basics/101-101-create.html). 

If you are using one of the bootstrap scripts right out of the box and have the data stored in a RIA (more on this later), you run it with the following command: 

```bash
bash bootstrap-script.sh ria+file://[/absolute/path/to/output_ria]\#~data [/path/to/container]
```
*Note the leading slashes and lack of trailing slashes. 

So, for example, here is the NKI pyAFQ run with our specific Sherlock pathways. 

```bash
bash bootstrap-pyAFQ-NKI.sh ria+file:///oak/stanford/groups/jyeatman/NKI_Rockland/QSIPrep/output_ria#~data /scratch/groups/jyeatman/pyafq-container 
```

This sets up the bootstrap directory. If you’re working with a larger dataset, it can take awhile to run. When complete, you’ll have a directory with `analysis/` `input_ria/` `output_ria/`. For now, you can ignore the `input_ria/` and `output_ria/`. 

RIA stores are a type of compressed git-annex storage that are efficient in terms of inodes and size. 

This analysis directory makes up a “subdataset” of our main dataset located at `/oak/stanford/groups/jyeatman/NKI_Rockland/QSIPrep/output_ria#~data` 

Inside `analysis/code` we have several scripts that are chained together when they are ran. `sbatch_calls.sh` →  `participant_job.sh` →`pyAFQ_zip.sh` . 

**Script details:** 

`sbatch_calls.sh` : contains the direct calls to launch the batch jobs. These batch jobs run `participant_job.sh`, with inputs designating which subjects are being run. 

`participant_job.sh` : gets the relevant subject from the RIA store, and executes a `datalad run` command on `pyAFQ_zip.sh`. 

`datalad run` gets the relevant inputs from the RIA store (`datalad get`), runs the command, and `datalad save`s the outputs as specified in the run command flags. Then, it pushes the outputs to datalad, and cleans up unnecessary directories that would mess with datalad. 

`pyAFQ_zip.sh` : creates each subject’s pyAFQ TOML file, and runs the pyAFQ singularity image.  

`merge_outputs.sh`: merges the job branches into the master branch, allowing you to see the outputs in each branch. If you want to run this iteratively, you’ll have to `rm -rf merge_ds` . 

**Suggested workflow:**

1. setup bootstrap directory `bash bootstrap.sh ria+file://[/path/to/output_ria]#~data [/path/to/container]`
    a. before you do this, you might need to update `session_list` parsing in the bootstrap to work with your input filenames (the lines after SLURM SETUP START). 
    b. you want this to be in a part of your filesystem that’s able to hold *all* of the outputs of your data. On Sherlock this is `$SCRATCH` and `$OAK`. 
2. make any tweaks you know you might need 
    a.  in `participant_job.sh`
        i. input/output filenames and paths
        ii. sbatch param adjustments
    b. changing `pyafq_zip.sh` to your desired preprocessing software
        i. changing toml file to a bids-filter file 
        ii. changing singularity command and flags
    c. adjusting `sbatch_calls.sh` for your filesystem 
        i. sbatch param adjustments
3. save these changes 
    a. `datalad save`  
    b. `datalad push --to input`
    c. `datalad push --to output` 
4. run some test subjects
    a. from the `analysis` directory: `$(head -n 5 code/sbatch_calls.sh | tail -n 1)` runs the 5th line in `sbatch_calls.sh` 
5. check these subjects out 
    a. `bash /path/to/merge_outputs.sh` 
    b. look in the merge_ds directory to see/unzip/check on your subjects 
6. If you’re satisfied, run all jobs, removing the test subjects from the sbatch_calls.sh
    a. `bash sbatch_calls.sh` 
7. If you’re not satisfied, copy/paste your changes from step 2 into the bootstrap script, re/move the original bootstrap directory, and rerun bootstrap command.
8. When all is done - *move your outputs somewhere stable*. 

**Problems that you might run into**
Changing the location of the bootstrap directory. 
This might happen if you don't set up the bootstrap directory somewhere logical that has enough storage space. It's a pain to try to change all of the git-annex guts, so try to avoid it. 

If you get a git-config error and you're working together with someone else, it's likely a permissions issue. Change your umask to something like 002. 




