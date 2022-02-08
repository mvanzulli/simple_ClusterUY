
# <span style="color:orange">Simple_ClusterUY   </span>

![Project Image](project-image-url)

> a simple tutorial for executing in uruguayan compute center.
---

This repository include a brief description for programming beginners who want to use [cluster-UY]([[^first]](https://cluster.uy/)) platform using a ubuntu linux distribution:

---
## <span style="color:green">Sections
- [Architecture](#Architecture)
- [Create a user](#CreateUser)
- [Execution](#Execution)
- [Partitions](#Partitions)
- [Quality of service](#QualityOfService)
- [Commands](#Commands)
- [Programs](#Programs)
- [Cluster subscription](#ClusterSubscription)
- [Execute matlab script](#ExecuteMatlabScript)
- [Transfer data](#TransferData)
- [References](#References)
---
## <span style="color:red">Architecture

 - The compute resources are open to free access, including business, researchers, students, and public institutions.  
- The project has no profit propose and its sustainable by the users buy of licenses or computation time. 
- The main component's are:
  - internet (100MBPs) shared by many users so its highly transitioned
  - A connection with FING (1Gbps) (access by TERO or insitute port)
  - transfer operator: a distributor of data that communicates each parts
  - 28 Computations servers: (CPU & GPU) where jobs are executed. Each hardware server offers 2 Intel Xenon GOLD, 128 RAM, GPU Tesla 12GB, Linux CentOS 7, 300 TB. 
  - A connection high performance network with 10GBpS
  - Are specific nodes 9node31 for multiprocessor executions (AMD Epic with 96 cores) and multiple parallel 2-3 GPU nodes for streaming and GPUs shared memories tasks. 
   
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

---
## <span style="color:red">Executions
Types of execution tasks:
  -  **interactive** In some development phases is necessary to execute fast and debug or test the code, for this interactive online task interactive type is the best solution, since the SLRUM gives priority. (Usually runs shorter than 30m ) 
        - To run a interactive job with 30 min max type:
  ```
  interactivo -g = srun --time+00:30:00 --pty bash 
  
  ```

- **by slots** Off-line execution using BATCH where the output takes 5 days.To run a batch file with 


```
#!/bin/bash
#SBATCH --job-name=charla
#SBATCH --ntasks=1 (number of times that script it is going to be executed,, if three tasks are settee the parallelism is implemented switching a local environment variable)
#SBATCH --cpus-per-task=1(this is modified to implement parallelism)
#SBATCH --mem=1G(there is no swap available if is exceeded you are death)
#SBATCH --time=0:01:00(we should overestimate this time to survive maybe 1,5 factor, this only affect to wait SLRUM )
#SBATCH --partition=normal
#SBATCH --qos=normal(quality of service)
#SBATCH --reservation=charla(this is used if a preview reservation was scheduled)
#SBATCH --mail-type=ALL
#SBATCH --mail-user=mvanzulli@fing.edu.uy
R CMD BATCH code.r
```
If the SLRUM manger finishes the job an email 
Resuming by sbatch parameters can be modified
- Time (max time, initial and ending time)
- I/O (input an output location and name file)
- CPU (number, location, etc)
- Memory (total Mem, location, etc)
- Partition QoS, account (number of partitions, quality of service, account)
## <span style="color:red">Partitions
There are different partitions types depending on the patience that the job require.
A normal execution guarantee that when de job starts the resources are not being used by other user, so if the job starts then it must be done.
| normal              | Description |
| ------------------- | ----------- |
| number of cores     | 560         |
| maximum times       | 5 days      |
| max number of tasks | 40          |

Or high power computing services, that are reserved by ANTEL and UTE, opportunistic queue. A best effort job could not be finish if any ANTEL or UTE user need them, we could be killed. At this time there are specific nodes that have no traffic. 

| besteffort          | Description |
| ------------------- | ----------- |
| number of cores     | 1200        |
| maximum times       | 5 days      |
| max number of tasks | 40          |
- normal: 560core max time 5 days max works
---
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

---
## <span style="color:red">Commands  
- Check the user status:
```bash
[mvanzulli@login ~] $ squeue -u mvanzulli
JOBID   PARTITION  NAME USER  ST  TIME   NODES  NODELIST

493442  norma      mvanzulli  PD  0:00    1  (Waiting reason)
493443  norma      mvanzulli  PD  0:00    2     node18
493444  norma      mvanzulli  R   0:00    2     node 15 - 16
```
- Check a job status with the number id 22644, if the job is not currently running then the estimated initial time is shown:

```bash
[mvanzulli@login ~] $ scontrol show job 22644
JOBID   PARTITION  NAME USER  ST  TIME   NODES  NODELIST

493442  norma      mvanzulli  PD  0:00    1  (Waiting reason)
493443  norma      mvanzulli  PD  0:00    2     node18
493444  norma      mvanzulli  R   0:00    2     node 15 - 16
```

- The data space it is limited, only 300GB are available but if it is necessary more space it can be assigned by cluster staff support. 

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

- To copy data `scp` command is used from clsterUY to local machine and in the other way. Transfer from cluster to local:

```bash
scp mvanulli@cluster.uy:heatmap.pdf .
heatmat.pdf                      100% 7699 646 Kb/s 00:04
```
- Sent a file to clusterUY with
```bash
scp  heatmap.r mvanulli@cluster.uy
heatmat.pdf                      100% 7699 646 Kb/s 00:04
```
- Check the state of the all ClusterUY nodes:
```bash
[mvanzulli@login ~]$ cstate -v
+---------------------------------------------+
|                  ClusterUY                  |
+----------+-------+-----------+------+-------+
| NODENAME | TOTAL | ALLOCATED | IDLE | OTHER |
+----------+-------+-----------+------+-------+
|   node01 |    40 |         0 |   40 |     0 |
|   node02 |    40 |         0 |   40 |     0 |
|   node03 |    40 |         0 |   40 |     0 |
|   node04 |    40 |         0 |   40 |     0 |
|   node05 |    40 |         0 |   40 |     0 |
|   node06 |    40 |         0 |    0 |    40 |
|   node07 |    40 |         8 |   32 |     0 |
```
- Check the state of a node:
```bash
[mvanzulli@login ~]$ squeue -w node07
           JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
         2351957 besteffor    PBE-r  ssastre  R   23:03:15      1 node07
```


---
## <span style="color:red">Programs
The following software are availaible to users:
- Octave
- Matlab
- R
- Python
- TensorFlow
- COMSol
- Julia
---
## <span style="color:red">ClusterSubscription
1. To subscribe to cluster platform just fill info at [this link.](cluster.uy/registrio)
2. Create a rsa pair of private keys to access executing:
```bash
  ssh-keygen -t rsa -b 2048 -C "<your_email>@fing.edu.uy"
```
This by default create a ssh key of 4096 bits in .ssh folder so then must be specified the name of the login user. If not is declared by defect use the user name defined in OS. 
3. Add public key to host. For such task execute 
   ```bash
   cat <mi_ssh.pub>
  ```
  and copy the content into the host page to your user keys, in gitlab fing is 
  https://gitlab.fing.edu.uy/, then config and "SSH keys". Paste the public key content

4. Create a `config` file in  `~/.ssh` and introduce:
   ```bash
    # GitLab.com
    Host gitlab.com
        Preferredauthentications publickey
        User <user_name>
        Port 22
        IdentityFile ~/.ssh/<my_ssh_privatekey>
    ```
5. Before adding a new SSH key to the ssh-agent to manage your keys, you should have checked for existing SSH keys and generated a new SSH key.

```bash
 eval "$(ssh-agent -s)"
```
6. Add your SSH private key to the ssh-agent.
   
```bash
  ssh-add ~/.ssh/nameKey
```

7. To indicate -i specific login user execute:
```bash
  mvanzulli@cluster.uy -i ssh/nameKey
```

Woalla you are in now!  Then a good practice is to create one backup private keys to avoid possibles headache of missing it in the future. 

---
## <span style="color:red">Execute a Matlab script
To run a matlab script it is necessary to create a bash file requesting a partition specifying all the parameters stated in [Partitions](#Partitions) section. A .bash script example to execute `staticVonMisesTruss.m` from the same folder where the file is located is:
```bash
vim onsasExample_staticVonMisesTruss.sh
```
```bash
#!/bin/bash
#SBATCH --job-name=staticVonMisses
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=4096(I the unit is not specifyied the unit will be MB )
#SBATCH --time=00:30:00
#SBATCH --tmp=9G 
#SBATCH --partition=normal
#SBATCH --qos=normal

#PATH TO MATLAB bin: /clusteruy/apps/matlab/R2018b/bin/matlab  
#ALIAS FOR MATLAB bin: alias matlab = "/clusteruy/apps/matlab/R2018b/bin/matlab"
/clusteruy/apps/matlab/R2018b/bin/matlab -nodisplay -nosplash -nodesktop -r "run('./onsasExample_staticVonMisesTruss.m');exit;"
```

 If the output file name is not described in `launch.sh`  the screen will be printed in a file inner the path where the launch is executed named: `slrum-JOBNUM.out`. 
 
---
## <span style="color:red">Trasnfer data 
Here is an example to transfer a .pdf from mi local PC to home clusterUY folder. For such task i should execute

```bash
  scp File.pdf mvanzulli@cluster.uy:~/
```
Ths will paste the file in home  cluster directory through login node. If another folder is the file destination it ~/ it should be changed for the folder:
```bash
scp File.pdf mvanzulli@cluster.uy:~/ONSAS/...
```

From local PC to cluster the upload velocity is near to 2MB/s and form FING server is around 25MB/s, so its highly recommend to copy from local to FING and then to clusterUY. 

To transfer all folder ONSAS files in the other direction then execute:

```bash
scp mvanzulli@cluster.uy:~/ONSAS/* ./home/localFolder
```

Another possible trick is to connect directly with transfer node machine (located in port 10022), **to do this the private keys must be copy yo home folder located in clusteruy**, 

```bash
  scp ~/.ssh/nameKey ~/.ssh/nameKey.pub mvanzulli@cluster.uy:~/
```
then,
```bash
  scp -P 10022 File.pdf mvanzulli@cluster.uy:~/
```
To mount a virtual directory linked to clusterUY in linux or windows use sshfs, this option will be commented in the following section. 

---
## <span style="color:red">Syncornize Folders

First login to your client system and upgrade included packages: 

```bash
sudo apt upgrade && sudo apt update
```
Install sshfs package is available with every Linux package manager. Use the commands specific to your distribution if you are not using Debian or Ubuntu.

```bash
sudo apt-get install sshfs
```
In order to mount file systems using sshfs from a normal user account, you’ll need to add the user to the fuse group first. To do that first

1. Check fuse group exists run:
```bash
cat /etc/group/ | grep 'fuse'
```
2. If the group exists execute
```bash
 sudo usermod -a -G fuse insertUserName
```
3. If the group not exists must be created by
```bash
 sudo groupadd fuse
 sudo usermod -a -G fuse insertUserName
 ```

Then the group is created sshfs format is analogously to scp format, to link a folder from your local PC to the remote just execute:

```bash
sshfs mvanzulli@cluster.uy:/home/folderInCluster /home/localFolder
```

If that command is executed the folders will be synchronized. 

---

[Back To The Top](#ClusterUY_Intructions)

---

## <span style="color:red">References

---

1. [Cluster-UY: Collaborative Scientific High Performance Computing in Uruguay](https://link.springer.com/chapter/10.1007/978-3-030-38043-4_16Footnote).
2. [Cluster Web](https://www.cluster.uy/)
