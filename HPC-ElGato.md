# About the UAMLDA Computing Resources 

This tutorial aims at clarifying how to use [Elgato](http://elgato.arizona.edu/) at the [High Performance Computing Center (HPC)](https://it.arizona.edu/service/high-performance-computing) at The University of Arizona. Elgato has 128 Tesla K20X GPU, which makes it very useful in large-scale machine learning problems. Students can upload code to Elgato and run TensorFlow applications using Elgato's GPU. This can save a lot of time.


# Contact Information

If you have any questions about the content of this tutorial, here are some people you may find helpful:

* HPC consultants (hpc-consult@list.arizona.edu). This list contains the experts who maintain the Elgato service at UA's HPC center. Replies are usually immediate. So definitely they are very helpful.
* Prof. Ditzler(ditzler@email.arizona.edu). Prof. Ditzler can sponsor you to have access to the Elgato system.
* Zhengzhong Liang (zhengzhongliang@email.arizona.edu). Author of this tutorial. If there are any unclear expressions, please feel free to contact.

# Login to the Elgato System

## Get Sponsored
Before you can login to the HPC, you need to get sponsored by a faculty. You can talk to Prof. Ditzler to discuss the sponsorship.

## Login via SSH
After you are sponsored, you can login to the HPC server using terminal. For example in Linux, you can press `alt+ctrl+t` to open a terminal. Then in terminal, type the following commands to login:

```
ssh username@hpc.arizona.edu
```

Then you are required to enter the password. You should enter the password corresponding to your NetID. Then the system requires an additional security verification. For me I use Duo Push for IOS. Thus after I select it using terminal, HPC will send a request to my phone. And I should confirm this request on my phone. After this two-factor verfication (password + Duo), I can login to the system.

The final step is to choose which platform we want to use. There are 3 in total: elgato, ocelote and ICE. Since we want to use elgato, we enter elgato.

## Run TensorFlow Application on Elgato
To run any resource on Elgato, you need to firstly apply for computation resources. Please do not run any application without firstly applying for resources. To achieve this, you need to do 2 things. Firstly, you need to write a bash file which contains your request. And secondly, you need to submit this request to Elgato system.
### Prepare Submit File
The following script is `submit.sh`. It shows the simplest way of requesting resources on Elgato.

```
#!/bin/bash
 
#BSUB -n 2
#BSUB -R "span[ptile=16]"
#BSUB -R gpu
#BSUB -q "windfall"
#BSUB -o GradNorm_clean.out
#BSUB -e GradNorm_clean.err
#BSUB -J GradNorm_clean
 
#---------------------------------------------------------------------
# Load "singularity 2.3.1, which is crucial to run tensorflow application on Tensorflow"
module load singularity

# Use this cd command to enter the directory where you want to run the application (usually where you store the .py script. This directory is the directory where the output .out file and .err file will be stored. The following directory is only an example. Please modify it according to your project.)
cd /home/u15/zhengzhongliang/Projects

# Use singulartiy as the python interpreter on elgato and run `My_Python_Script.py`
singularity run --nv /unsupported/singularity/keras/keras+tensorflow-1.4.1_gpu-cp35-cuda8-cudnn6.img My_Python_Script.py 

# Here `--nv` indicates that the application will require the usage of GPUs.
# tf_gpu-1.2.0-cp35-cuda8-cudnn51.img is a TensorFlow image which is prepared by the HPC staff and immediately available on Elgato. You can access that using the directory above, or you can copy it to your own directory.
```

The header of this `submit.sh` file is queue request. The usage of each line is listed as following:
```
#BSUB -n 2
```
We will apply for 2 GPUs for our job. Elgato has 128 NVIDIA K20X GPUs, and it has 64 cores, which means each core has 2 GPUs. And these 2 GPUs in a core have shared memory. So communication between the 2 GPUs in the same core does not require Message Passing Interface (MPI).

```
#BSUB -R "span[ptile=16]"
```
The 128 GPUs on Elgato are divided into 8 piles, with each pile consisting 16 GPUs. The exact meaning of this sentence is not very clear. But according to experience, setting ptile to 16 usually makes the tasks go well.

```
#BSUB -R gpu
```
Specify we want to use GPU on Elgato.

```
#BSUB -q "windfall"
```
This is the priority of our job. `windfall` has the lowest priority. And one drawback of windfall is that, if you submit multiple jobs to Elgato consecutively under windfall, the tasks may be terminated half way. So if possible, please require the HPC staff to add you to a `standard` queue. This way you are able to use more resource on Elgato.

```
#BSUB -o GradNorm_clean.out
```
After finishing execution, Elgato will generate the output file of the task. And the output file's name will be `GradNorm_clean.out`. 

```
#BSUB -e GradNorm_clean.err
```
The error message will be included in the file named `GradNorm_clean.err`.

```
#BSUB -J GradNorm_clean
```
Your task will appear in Elgato system with the name of `GradNorm_clean`

The basic logic behind this is that you need both `singularity` and `tf_gpu-1.2.0-cp35-cuda8-cudnn51.img` to run TensorFlow application on Elgato. These 2 modules combined serve as a python interpreter with TensorFlow library installed.

### Submit Request to Elgato
After finishing writing the `submit.sh` file, you need to submit this request to Elgato through terminal. You need to firstly login to Elgato through ssh (as stated in the sections above). Then you need to access the directory where your `submit.sh`
file is stored. For example, if the submit file is stored at `/home/u15/zhengzhongliang/Projects`, then you need to type the following commands to submit the request:
```
$ cd /home/u15/zhengzhongliang/Projects
$ bsub < submit.sh
```

# Advanced Techniques of Elgato

The following techniques may maker it easier if you want to run multiple experiments on Elgato. 
## Use Linux GUI to Access Files on HPC
If you are using Ubuntu, you will be able the access the files on Elgato through Ubuntu's GUI. The following show the steps:
(1) Open the file manager GUI (usually on the left side of desktop)
(2) In the left menu, find the "Other Locations" option.
(3) In the bottom blank, enter "ssh://zhengzhongliang@filexfer.hpc.arizona.edu". Here "zhengzhongliang" is your HPC username. Then enter the password of your HPC account. Then you should be able to access the fils through GUI.


## Use the GPUs of a Core Exclusively
If you want your job to be run un-interupted by other jobs you submit, you need to request to use the GPUs on 1 core excusively. That is, a different job that you submit to Elgato should be run on another core (another 2 GPUs). To do this, you need to revise the submit file accordingly. You need to change priority to "standard", and you need to add a declaration `#BSUB -x`, which asserts to use the core exclusively.
```
#!/bin/bash
 
#BSUB -n 2
#BSUB -R "span[ptile=16]"
#BSUB -R gpu
#BSUB -q "standard"
#BSUB -o GradNorm_clean.out
#BSUB -e GradNorm_clean.err
#BSUB -J GradNorm_clean
#BSUB -x
```

## Write Loop to Submit Multiple Jobs Automatically
The scripts above is not the only way of writting a submission request. The following two pieces of files show another way of writing submission files. 

The first file is `submit.sh`.
```
#!/bin/bash


module load singularity
cd /home/u15/zhengzhongliang/TensorflowGPU/RNN_Poison_Project/Exp19_FormalResultGeneration_4_GradNorm_MoreRatio

seed=42

rep_times="4 8 16 32 64 128 256"
ratios="002 005 010 020"

# medium network, nested loop: loop over ratio and loop over rep times
n_step="35"
keep_prob="0.5"
grad_norm=$1
for ratio in ${ratios[*]}
do
  for rep_time in ${rep_times[*]}
  do
    singularity run --nv /unsupported/singularity/keras/keras+tensorflow-1.4.1_gpu-cp35-cuda8-cudnn6.img rnn_lm.py \
      --model medium \
      --seed ${seed} \
      --n_step ${n_step} \
      --keep_prob $keep_prob \
      --grad_norm ${grad_norm} \
      --train="data/ptb.train.mix"${ratio}".rep"${rep_time}".txt" \
      --train2=data/shakes.train.010.txt \
      --test=data/ptb.test.txt \
      --valid=data/ptb.valid.txt \
      --output="outputs/ptb_exp_medium_mix"${ratio}"_rep"${rep_time}"_grad_"${grad_norm}".csv"
  done
done



# small network, nested loop: loop over ratio and loop over rep times
n_step="20"
keep_prob="1.0"
grad_norm=$1
for ratio in ${ratios[*]}
do
  for rep_time in ${rep_times[*]}
  do
    singularity run --nv /unsupported/singularity/keras/keras+tensorflow-1.4.1_gpu-cp35-cuda8-cudnn6.img rnn_lm.py \
      --model small \
      --seed ${seed} \
      --n_step ${n_step} \
      --keep_prob $keep_prob \
      --grad_norm ${grad_norm} \
      --train="data/ptb.train.mix"${ratio}".rep"${rep_time}".txt" \
      --train2=data/shakes.train.010.txt \
      --test=data/ptb.test.txt \
      --valid=data/ptb.valid.txt \
      --output="outputs/ptb_exp_small_mix"${ratio}"_rep"${rep_time}"_grad_"${grad_norm}".csv"
  done
done
```

The second file is `submitAll.sh`
```
#!/bin/bash

interval="5"
grad_norms="1 2 10 20 40"

for grad_norm in ${grad_norms}
do
  bsub -n 2 -R "span[ptile=16]" -R gpu -q "standard" -o "GradNorm_"${grad_norm}".out" -e "GradNorm_"${grad_norm}".err" -J "GradNorm_"${grad_norm} -x "sh submit.sh ${grad_norm}"
  sleep ${interval}
done
```

The logic behind is: you need to enter `./submitAll.sh` in Elgato's terminal, and this `submitAll.sh` file will automatically submit `submit.sh` file for several times. Each time, a different `grad_norm` will be passed to `submit.sh` file. In the `submit.sh` file, the received `grad_norm` value can be retrieved by `$1`. This is how parameters can be passed through different bash files. This way you can submit the requests for multiple jobs at a time, and that these jobs can have different parameters.
