# Updating

## Keynote: After successfully installing the iBEAT Docker, if you encounter any errors or bugs, please send us the log file along with the testing subject for efficient troubleshooting.

## Release notes for version 2.1.0
* We add the subcortical semgmentation module to this release. By default, when you using this version, it will output the subcortical segmentation file, whose filename is ended with "-subcortical-segmentation.nii.gz". If you want to skip the subcortical segmentation module, you can add an additional option with "--subcortical_type 0".
* The latest version can be acquired by `docker pull ibeatgroup/ibeat_v2:release210`.

## Release notes for version 2.0.8
* We upgraded the cerebellum removal performance.
* The latest version can be acquired by: `docker pull ibeatgroup/ibeat_v2:release208`.

## Release notes for version 2.0.5
* We have fixed bugs, including:
    * The input masks cannot be correctly recognized when input brain mask or cerebrum mask.
    * When the brain is close to the boundary, the pipeline may occasionally exit abnormally.
    * The `--skip_surface` option may not work properly in some cases.
* The option `--skull_prob_thresh` has been removed. Instead, we provided two more options, i.e., `--t1_skull_prob_thresh` and `--t2_skull_prob_thresh` to make it more flexible for different modalities especially when two modalities are input together.
* We upgraded the skull stripping performance.
* The latest version can be acquired by: `docker pull ibeatgroup/ibeat_v2:release205`.

## Release notes for version 2.0.0
* We have improved the skull stripping and cerebellum removal model to make the toolkit more reliable.
* We have fixed the boundary issues. Previously, when the brains are too close to the boundary, the segmentation may generate inaccurate results. Now, the issue has been addressed.
* We have made the system work more robust. When the potential error occurs, the pipeline will stop and output more meaningful logs for easy problem location.
* We updated the command line parameters to make them more consistent with human instinct. Please refer to section **Running the pipeline container** for more details.
* The latest version can be acquired by: `docker pull ibeatgroup/ibeat_v2:release200`.

## Release notes for version 1.2.0
* We have fixed the issue that the pipeline fails to run on newer GPUs. The latest version can be acquired by: `docker pull ibeatgroup/ibeat_v2:release120`.
* Using Cuda 11+cudnn 8 will increase the GPU memory needs. On our local testing, we noticed that the needed minimal GPU memory is around 5GB.

## Improvement in Version 1.1.0
* The cerebellum removal modal was improved.
* Enable user to provide customized brain mask and/or cerebrum mask for the pipeline. See more details in [run the pipeline section](#run-the-pipeline).
* The latest version can be acquired by: `docker pull ibeatgroup/ibeat_v2:release110`.

# Introduction
This ```README``` illustrates how to install and run the Docker version of iBEAT V2.0 pipeline, which is an infant-dedicated structural imaging processing pipeline for infant brain MR images. More details of the pipeline can be referred to [iBEAT V2.0 Cloud](http://www.ibeat.cloud). 

# System requirement
Since this is a *Linux* based container, please install the container on a Linux system. The supported systems include but not limited to `Ubuntu`, `Debian`, `CentOS` and `Red Hat`. 

The pipeline is developed based on the deep convolutional neural network techniques. A GPU is required to support the processing. During the running, around 3 GB GPU memory is required. 

# Installation
### Install Docker
Please refer to the official installation guideline [Install Docker on Linux](https://docs.docker.com/desktop/install/linux-install/). You can follow the below commands to install the Docker on the popular Ubuntu Linux system. If you have previously installed Docker on your computer, please skip this step.
```
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release
sudo apt mkdir -p /etc/share/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/share/keyrings/docker.gpg
sudo echo "deb [arch=$(dpkg --print-architechture) signed-by=/etc/share/keyrings/docker.png] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
Since the docker needs the `sudo` to run the general commands, to run without `sudo`, you need to add your user name `${USER}` to the docker group, which can be done
```
sudo group add docker
sudo usermod -aG docker ${USER}
```
After running these commands, you may need to restart your computer to make the configuration take effect. One easy way to check whether you have successfully installed the Docker or not is to run the Docker using the command ```docker info```, which dose not need ```sudo``` requirement if you have added your username to the `docker` group. Please refer to [more details](https://docs.docker.com/get-started/) for potential issues on Docker.


### Install Nvidia-docker
The ```Nvidia-docker``` is required since our pipeline needs GPU support. Please refer to the official installation guideline [Install Nvidia-docker on Linux](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker). If you have previously installed Nvidia-docker, please skip this step. The following commands can be used to install the Nvidia-docker on the popular Ubuntu Linux system.
```
sudo apt update
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-docker.gpg
distribution=$(. /etc/os-release; echo ${ID}${VERSION_ID})
curl -sL https://nvidia.github.io/nvidia-docker/libnvidia-container/${distribution}/libnvidia-container.list | sed 's#deb https://# deb [signed-by=/usr/share/keyrings/nvidia-docker.gpg] https://#g' | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt update
sudo apt install -y nvidia-docker2
```
After the installation, please restart the Docker daemon to complete the installation after setting the default funtime:
```
sudo systemctl restart docker
```
Finally, you can check whether you have successfully installed the ```Nvidia-docker``` using the following command:
```
docker run --rm --gpus=all nvidia/cuda:9.0-base nvidia-smi
```
If succeeded, the output should be the GPU card information on your PC. 

### Download the pipeline
Run ```docker pull ibeatgroup/ibeat_v2:release_version_tag```, where ```release_version_tag``` is the container tag. Currently, the latest container tag is ```release205```, thus run ```docker pull ibeatgroup/ibeat_v2:release205``` to download the pipeline.

After downloading, you can use ```docker images``` to see the container images you have downloaded.

# Running the pipeline container
## Get the free license
The container is totally free. Please first register on the [iBEAT V2.0 Cloud](https://ibeat.wildapricot.org/Sys/Login?ReturnUrl=%2fdocker) to get a free license.

## Run the pipeline
Before running the pipeline, please:
* Create a directory (let's say **`your_data_folder`**), which will be mounted to the container. The container will load the data for processing from this directory and also save the processed results to this directory. 
* Put the "License" file (please do not rename/modify the license file, no extension) into the created folder.

After that, you can run the pipeline with the following example code:
```
docker run --gpus=all --rm -it -v /your_data_folder:/InfantData --user $(id -u):$(id -g) ibeatgroup/ibeat_v2:release100 --t1 <t1_file_path> --t2 <t2_file_path> --age <age_in_month> --out_dir <result_dir> --sub_name <subject_id>
```
In the above example, you can regard `docker run --gpus --rm -it -v /your_data_folder:/InfantData --user $(id -u):$(id -g) ibeatgroup/ibeat_v2:release100` as a simple linux command (despite it is long)
- `docker run --gpus=all` is the command to run a docker container with all gpus, you can change `--gpu=all` to a specific gpu id if you have multiple gpus installed, for example, `--gpu=0` will use the first gpu.
- `--rm` indicates that the container will be removed from the memory once it is finished.
- `-v` indicates the input data folder. **`/your_data_folder`** is the directory where you put the processing data and the license. **`/InfantData`** is the internal path inside the container to locate the data and license.
- `--user $(id -u):$(id -g)` indicates the container will be runned in the provided user (current user of the linux) and provided group (current user group).
- `ibeatgroup/ibeat_v2:release100` is the container name of the pipeline.

The **important parameters** are: 
* `--t1` or `--t2`: the path of the T1w or T2w infant brain image (relative to **`your_data_folder`**), which is the data folder you plan to mount into the container. They should be in NIfTI format (.nii). If you only have one modality, just input the available modaility (T1w or T2w).
* `--age[a]`: the infant age at scan (in MONTH).
* `--out_dir[d]`: the folder where to save the results. If empty, it will be created. Of note, this folder is rooted based on the inputed data folder (*your_data_folder* in the above example).
* `--sub_name[n]`: the subject name for the current processing subject. If you don’t assign, the pipeline will refer based on your T1 and T2 image names.
You can get detailed parameter information and simple example command by just running the pipeline without any parameters.

The **user interventioned parameters** are:
* `--skull_type`: how to do the skullstripping. `0`: skip. `1`: use our model. `2`: provide the mask. The mask file is provided by the following 2 parameters. (optional, default: `1`)
* `--t1_skull_mask`: skull mask for T1. (optional, only meaningful when skull_type is set to `2`)
* `--t2_skull_mask`: skull mask for T2. (optional, only meaningful when skull_type is set to `2`)
* `--cere_type`: how to do the cerebellum removal. `0`: skip. `1`: use our model. `2`: provide the mask. The mask file is provided by the cere_mask parameter. (optional, default: `1`)
* `--cere_mask`: cerebrum mask file. (optional, only meaningful when cere_type is set to `2`)
* `--tissue_type`: how to do the tissue segmentation. `0`: skip. `1`: use our model.  If `0`, the pipeline will only do the surface reconstruction with the input tissue specified by tissue_in parameter. (optional, default: `1`)
* `--tissue_in`: the provided tissue map. (optional, only meaningful when cere_type is set to `0`)
* `--skip_surface`: whether need to do the surface reconstruction. `0`: skip. `1`: do the reconstruction. (optional, default: `1`)
* `--t1_skull_prob_thresh`: The threshold for binarizing the skull stripping probability map for the T1w modality. You can set this value between 0 and 1 to adjust the skull stripping mask. (optional, default: 0.5).
* `--t2_skull_prob_thresh`: The threshold for binarizing the skull stripping probability map for the T2w modality. You can set this value between 0 and 1 to adjust the skull stripping mask. (optional, default: 0.9).
* `--cerebrum_prob_thresh`: The threshold for binarizing the cerebrum probability map. You can set this value between 0 and 1 to adjust cerebrum mask. (optional, default: 0.5)
* `--gpu_id`: determine which gpu will be used. `0`, `1`, `2`, … (optional, default: `0`)

Note, these parameters are modified since version release 200.
* `--skull_mask`(removed from version release200): the path of the user provided brain mask. If there is only one modality (either `--t1` or `--t2` is used), this would be the brain mask for that modality. If there are two modalities input (both `--t1` and `--t2` are used), this would be the brain mask for the t1w modality. In this case, if you also want to provide a brain mask for t2 modality, please use an additional parameter `--exmod_skull_mask` (see below).
* `--exmod_skull_mask`(removed from version release200): the path of the user provided brain mask for t2w image when both t1w and t2w images are input.
* `--cere_mask`(removed from version release200): the path of the user provided cerebrum mask. If there is only one modality, this would be the cerebrum mask for that modality. If there are two modalities input, this would be the cerebrum mask for the t1w modality. Since at the cerebellum removal stage, the t2w image has been aligned with the t1w image. So, the t2w cerebrum mask is no longer needed.

```
nvidia-docker run --rm -it -v /your_data_folder:/InfantData --user $(id -u):$(id -g) ibeatgroup/ibeat_v2:release100 --t1 <t1_file> --age <t1_image_age> --out_dir <result_dir> --sub_name <subject_id>
```
The above command is a typical example of processing one subject with both T1w and T2w images. You can also only input a single T1w (or T2w) image if you only have one modality.

## Batch processing
Since `docker run --gpus=all --rm -it -v /your_data_folder:/InfantData --user $(id -u):$(id -g) ibeatgroup/ibeat_v2:release100` can be regarded as single command, you can also write a script to process the data in a batch by treating the pipeline command as a simple command in a ```for``` or ```while``` loop. The following is a simple example if you are using the bash script,
```
for t1_file in t1_file_pattern; do
    nvidia-docker run --rm -it -v /your_data_folder:/InfantData --user $(id -u):$(id -g) ibeatgroup/ibeat_v2:release100 --t1 <t1_file> --age <t1_image_age> --out_dir <result_dir> --sub_name <subject_id>
done
```

## Output illustration
After the processing is finished, in the "mounted" folder **`your_data_folder`**, all the processing results will be generated. The following explains what the results are: 
* T1(or T2).nii.gz:	The raw image file.
* T1(or T2)-brainmask.nii.gz: The brain mask of T1w (or T2w).
* T1(or T2)-ceremask.nii.gz: The cerebrum mask of T1w (or T2w).
* T1(or T2)-skullstripped.nii.gz: The skull stripped brain image of T1w (or T2w).
* T1(or T2)-skullstripped-rmcere.nii.gz: The skull stripped and cerebellum removed T1w (or T2w) image.
* T1(or T2)-skullstripped-rmcere-tissue.nii.gz: The cerebrum tissue segmentation in T1w (or T2w) space.
* T2-ToT1.nii.gz: The aligned T2w in T1w space.
* T1(or T2)-skullstripped-rmcere.lh.nii.gz: Left hemisphere tissue image.
* T1(or T2)-skullstripped-rmcere.rh.nii.gz: Right hemisphere tissue image.
* T1(or T2)-skullstripped-rmcere.lh.InnerSurf.PhysicalSpace.vtk: Reconstructed left hemisphere inner cortical surface.
* T1(or T2)-skullstripped-rmcere.lh.MiddleSurf.PhysicalSpace.vtk: Reconstructed left hemisphere middle cortical surface.
* T1(or T2)-skullstripped-rmcere.lh.OuterSurf.PhysicalSpace.vtk: Reconstructed left hemisphere outer cortical surface.

# Frequently asked questions
### Do I must have a GPU to run the pipeline?
Yes. In the current version, we do need GPU support to run the pipeline. In the future, we will also release the pipeline that only needs a CPU for computation.
### Is the pipeline robust to the imaging parameters?
Yes. We have successfully processed 16,000+ infant brain images with various imaging protocols and scanners from 100+ institutions. Please see https://ibeat.wildapricot.org/Feedbacks. 
### Are there any differences between iBEAT V2.0 Docker and iBEAT V2.0 Cloud (http://www.ibeat.cloud)?
Yes. The iBEAT V2.0 Cloud (http://www.ibeat.cloud) is timely updated with our latest developments, while the Docker version could be slightly delayed in updating. For optimal performance, iBEAT V2.0 Cloud is highly recommended.


# How to Cite?
Please cite the following papers if you use the results provided by the iBEAT V2.0 pipeline:
* L. Wang, Z. Wu, L. Chen, Y. Sun, W. Lin, G. Li, "[iBEAT V2. 0: a multisite-applicable, deep learning-based pipeline for infant cerebral cortical surface reconstruction](10.1038/s41596-023-00806-x)," Nature Protocol, vol. 18(5), 2023, pp. 1488-1509. 
* L. Wang, G. Li, F. Shi, X. Cao, C. Lian, D. Nie, et al., "[Volume-based analysis of 6-month-old infant brain MRI for autism biomarker identification and early diagnosis](https://pubmed.ncbi.nlm.nih.gov/30430147/)," in International Conference on Medical Image Computing and Computer-Assisted Intervention, 2018, pp. 411-419.
* G. Li, J. Nie, L. Wang, F. Shi, J. H. Gilmore, W. Lin, et al., "[Measuring the dynamic longitudinal cortex development in infants by reconstruction of temporally consistent cortical surfaces](https://pubmed.ncbi.nlm.nih.gov/24374075/)," Neuroimage, vol. 90, pp. 266-279, 2014.
* G. Li, L. Wang, P.-T. Yap, F. Wang, Z. Wu, Y. Meng, et al., "[Computational neuroanatomy of baby brains: A review](https://pubmed.ncbi.nlm.nih.gov/29574033/)," NeuroImage, vol. 185, pp. 906-925, 2018.
* G. Li, L. Wang, F. Shi, J. Gilmore, W. Lin, D. Shen, "[Construction of 4D high-definition cortical surface atlases of infants: Methods and applications](https://pubmed.ncbi.nlm.nih.gov/25980388/)," Medical Image Analysis, 25: 22-36, 2015.

# Contacts
The iBEAT V2.0 software is developed by the University of North Carolina at Chapel Hill:
* Volume-based analysis was designed in the Developing Brain Computing Lab, led by Dr. Li Wang (li_wang@med.unc.edu);
* Surface-based analysis was designed in the Brain Research through Analysis and Informatics of Neuroimaging (BRAIN) Lab, led by Dr. Gang Li (gang_li@med.unc.edu).

For questions/bugs/feedback, please contact:

Zhengwang Wu, Ph.D.,  zhwwu@med.unc.edu\
Li Wang, Ph.D.,  li_wang@med.unc.edu\
Gang Li, Ph.D.,  gang_li@med.unc.edu\
Department of Radiology and Biomedical Research Imaging Center\
The University of North Carolina at Chapel Hill








