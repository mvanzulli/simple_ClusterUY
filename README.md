
# <span style="color:orange">ClusterUY_Intructions   </span>

![Project Image](project-image-url)

> a simple tutorial for executing in uruguayan compute center.
---

This repository include a brief description for programming beginners who want to use [cluster-UY]([[^first]](https://cluster.uy/)) platform:


## <span style="color:green">Sections
- [Architecture](#Architecture)
- [Create a user](#CreateUser)
- [Create a user](#Execution)
- [Transfer data](#DataTransfer)
- [Refernces](#References)
---
## <span style="color:red">Architecture

 - The compute reosurces are open to free acess, inclung bussnes, researchers, students, and public institutions.  
- The project has no profit propuse and its suteniable by the usears buy of liscenses or computation time. 
- The main componentes are:
  - internet (100MBPs) shared by many users so its higly transitated
  - A conection with FING (1Gbps) (acces by TERO or insitute port)
  - transfer operator: a distributor of data that comunicates each partxs
  - 28 Computations servers: (CPU & GPU) where jobs are executed. Each hardware server offers 2 Intel Xenon GOLD, 128 RAM, GPU Tesla 12GB, Linux CentOS 7, 300 TB. 
  - A conection high performance network with 10GBpS
  - Are specific nodes 9node31 for multiprocess executions (AMD Epic with 96 cores) and multiple parlallel 2-3 GPU nodes for streaming and GPUs shared memoryies tasks. 
   
---
## <span style="color:red">CreateUser

To access clusterUY with ssh just run:

```
ssh usuario@cluster.uy
```

Its not so simple as simple PC, are some hardware requirements to guarantee: 
        - not conflict
        create different type of execution works
        - Work by slots with some specific hardware resources. 
        - Optimize the resources according to available hardware.

SLRUM is the answer to all such problems. This program takes where input script files and create an output to order the executions efficiently. 

## <span style="color:red">Executions
Types of execution tasks:
  -  **interactive** In some development phases is necessary to execuct fast and debug or test the code, for this interactive online task interactive type is the best solution, since the SLRUM gives priority. (Usally runs shorter than 30m ) 
        - To run a itneractive job with 30 min max type:
  ```
  interactivo -g = srun --time+00:30:00 --pty bash 
  
  ```

- **by slots** Off-line execution using BATCH where the output takes 5 days.To run a batch file with 


```
#!/bin/bash
#SBATCH --job-name=charla
#SBATCH --ntasks=1 (number of times that script it is going to be executed,, if three tasks are setted the paralelisim is implemented switching a local enviroment variable)
#SBATCH --cpus-per-task=1(this is modifyed to implement paralelism)
#SBATCH --mem=1G(there is no swap aviable if is exceded you are death)
#SBATCH --time=0:01:00(we should overestimate this time to survive maybe 1,5 factor, this only affect to wait SLRUM )
#SBATCH --partition=normal
#SBATCH --qos=normal(quality of service)
#SBATCH --reservation=charla(this is used if a preview reservation was scheduled)
#SBATCH --mail-type=ALL
#SBATCH --mail-user=mvanzulli@fing.edu.uy
R CMD BATCH code.r
```
If the SLRUM manger finishes the job an email 
Resuming by sbatch paramaters can be moddificated
- Time (max time, initial and ending time)
- I/O (input an output location and name file)
- CPU (number, location, etc)
- Memory (total Mem, location, etc)
- Partition QoS, account (number of partitions, quality of service, account)
## <span style="color:red">Partitions
There are different partitions types depending on the patience that the job require.
A normal exqution guarantee that when de job begien the resources are not being used by other user, so if the job starts then it must be done.
| normal              | Description |
| ------------------- | ----------- |
| number of cores     | 560         |
| maximum times       | 5 days      |
| max number of tasks | 40          |

Or high power computing services, that are reserved by ANTEL and UTE, oportuinse queue. A best effort job could not be finish if any ANTEL or UTE user need them, we could be killed. At this time there are specific nodes that have no traffic. 

| besteffort          | Description |
| ------------------- | ----------- |
| number of cores     | 1200        |
| maximum times       | 5 days      |
| max number of tasks | 40          |
- normal: 560core max time 5 days max wwoks
## <span style="color:red">QualityOfService
There are different partitions types depending on the patience that the job require.
A normal exqution guarantee that when de job begien the resources are not being used by other user, so if the job starts then it must be done.
| normal        | Description              | Partition comptaible | comment                                |
| ------------- | ------------------------ | -------------------- | -------------------------------------- |
| gpu           | 80 cores 384GB RAM, 4GPU | normal               |
| besteffor_gpu | 120 cores 768GB RAM      | besteffort           |
| rapida        | 8 cores 16 RAM           | normal               | mayor priority                         |
| normal        | 80 cores 684 RAM 0 GPU   | normal               |
| manycore      | -                        | normal               |
| bessteffort   | 120 cores, 768 GB RAM    | normal               |
| bigmen        | 40 cores, 512 GB RAM     | bessteffort          | SELECT THIS FOR SHORT TIME THAN normal |


[Back To The Top](#ClusterUY_Intructions)
## <span style="color:red">Commands  
Check the user status:
```bash
[mvanzulli@login ~] $ squeue -u mvanzulli
JOBID   PARTITION  NAME USER  ST  TIME   NODES  NODELIST

493442  norma      mvanzulli  PD  0:00    1  (Waiting reason)
493443  norma      mvanzulli  PD  0:00    2     node18
493444  norma      mvanzulli  R   0:00    2     node 15 - 16
```
Check a job status with the number id 22644, if the job is not currently running then the estimated initial time is shown:

```bash
[mvanzulli@login ~] $ scontrol show job 22644
JOBID   PARTITION  NAME USER  ST  TIME   NODES  NODELIST

493442  norma      mvanzulli  PD  0:00    1  (Waiting reason)
493443  norma      mvanzulli  PD  0:00    2     node18
493444  norma      mvanzulli  R   0:00    2     node 15 - 16
```

The data space it is limited, only 300GB are available but if it is necessary more space it can be assigned by cluster staff support. 

```bash
[mvanzulli@login ~] $ quota -gs show job 22644
Disk quoates fo group clusterusers (grid 10002): none
Disk quoates fo group mvanzulli (grid 10002): 
Filesystem space quota limit  grace files  quota limit grace
filserver: /home
                 218G  300G   301G   8552  0     0     
                 space max    max    numb  limit
                 used  limit  grace  files files
                              limit
```

To copy data `scp` command is used from clsterUY to local machine and in the other way. Transfer from cluster to local:

```bash
scp mvanulli@cluster.uy:heatmap.pdf .
heatmat.pdf                      100% 7699 646 Kb/s 00:04
```
Sent a file to clusterUY with
```bash
scp  heatmap.r mvanulli@cluster.uy
heatmat.pdf                      100% 7699 646 Kb/s 00:04
```
Check the user status:

## <span style="color:red">Programs
The following software are availaible to users:
- [ ] Octave
- [ ] Matlab
- [ ] R
- [ ] Python
- [ ] TensorFlow
- [ ] COMSol
- [ ] Julia

## <span style="color:red">ClusterSuscription
1. To suscribe to cluster platform just fill info at [this link.](cluster.uy/registrio)
2. Create a pair of private keys to access executing:
```bash
$ ssh-keygen -t nameKey -b 4096
```
This by default create a ssh key of 4096 bits in .ssh folder so then must be specified the nomber of the user login. If not is declared by defect use the user name definied in OS. 

To indicate -i specific login user execute:
```bash
$ mvanzulli@cluster.uy -i ssh/nameKey
```

Woalla you are in now!  Then a good practice is to create one bockup private keys to avoid possibles headache missing it. 

## <span style="color:red">References

1. [](https://link.springer.com/chapter/10.1007/978-3-030-38043-4_16Footnote).
