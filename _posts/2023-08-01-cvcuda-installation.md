---
layout: post
title:  CVCuda installation
date:   2023-08-01 20:00:00
description: tutorial
tags: notes
categories: note-posts
---
#### CVCUDA Installation references

<a href="https://cvcuda.github.io">cvcuda documentation</a>

<a href="https://github.com/CVCUDA/CV-CUDA">cvcuda github repo</a>

#### Run samples in docker

<b>1</b>. Download and install cvcuda samples from <a href="https://github.com/CVCUDA/CV-CUDA/releases/tag/v0.3.1-beta">CVCUDA release</a>. The release files are all based on cuda 12.

{% highlight bash %}
# download
mkdir -p ~/tmp
cd ~/tmp
curl -L https://github.com/CVCUDA/CV-CUDA/releases/download/v0.3.1-beta/cvcuda-samples-0.3.1_beta-cuda12-x86_64-linux.deb -o cvcuda-samples-0.3.1_beta-cuda12-x86_64-linux.deb
# install
dpkg -i cvcuda-samples-0.3.1_beta-cuda12-x86_64-linux.deb
{% endhighlight %}

<b>2</b>. Run the docker, this image comes with python 3.8:

{% highlight bash %}
# link the sample folder
docker run -it --gpus=all -v  /opt/nvidia/cvcuda0/samples:/samples nvcr.io/nvidia/tensorrt:23.04-py3
{% endhighlight %}
Tip: Find the desired base docker image with TenserRT and cuda 12 from <a href="https://docs.nvidia.com/deeplearning/tensorrt/container-release-notes/index.html">tensorrt container release notes page</a>.


<b>3</b>. Ensure the scripts are executable: 
(From step 3 to 6, the cmd are run in the docker container.)

{% highlight bash %}
# copy the samples into the user folder.
cp -rf /opt/nvidia/cvcuda*/samples ~/
cd ~/samples
chmod a+x scripts/*.sh
chmod a+x scripts/*.py
{% endhighlight %}

<b>4</b>. Install sample dependencies:

Basically, step by step run the script in `./scripts/install_dependencies.sh` with the following modification:

a) pip3 package versions

{% highlight bash %}
# Install torch and torchvision for cuda12:
pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu121
# Install pycuda with auto version (latest version installed: 2022.2.2):
pip3 install av==10.0.0 pycuda nvtx==0.2.5
# Install onnx
pip3 install onnx
{% endhighlight %}

In script, we can do the following (replacing line 55):

{% highlight bash %}
sed -i -e '55s+.*+pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu121\npip3 install av==10.0.0 pycuda nvtx==0.2.5\npip3 install onnx\n+' scripts/install_dependencies.sh
{% endhighlight %}

b) Refine exporting PATH: replace echo "export PATH=$PATH:<some_path>" with "export PATH=\${PATH}:<some_path>" 

{% highlight bash %}
sudo sed -i -e 's+$PATH+\\${PATH}+' scripts/install_dependencies.sh
{% endhighlight %}

c) Edit `setup.py` for `torchnvjpeg` to specify `std:c++17` as the compiler to be compatible with `pytorch`.

{% highlight bash %}
sudo sed -i -e 's/std=c++14/std=c++17/' torchnvjpeg/setup.py
{% endhighlight %}

<b>5</b>. Install the CV-CUDA packages in the docker.

{% highlight bash %}
# download
mkdir -p /tmp
cd /tmp
curl -L https://github.com/CVCUDA/CV-CUDA/releases/download/v0.3.1-beta/nvcv-dev-0.3.1_beta-cuda12-x86_64-linux.deb -o nvcv-dev-0.3.1_beta-cuda12-x86_64-linux.deb

curl -L https://github.com/CVCUDA/CV-CUDA/releases/download/v0.3.1-beta/nvcv-lib-0.3.1_beta-cuda12-x86_64-linux.deb -o nvcv-lib-0.3.1_beta-cuda12-x86_64-linux.deb

curl -L https://github.com/CVCUDA/CV-CUDA/releases/download/v0.3.1-beta/nvcv-python3.8-0.3.1_beta-cuda12-x86_64-linux.deb -o nvcv-python3.8-0.3.1_beta-cuda12-x86_64-linux.deb

# install
dpkg -i nvcv-lib-0.3.1_beta-cuda12-x86_64-linux.deb
dpkg -i nvcv-dev-0.3.1_beta-cuda12-x86_64-linux.deb
dpkg -i nvcv-python3.8-0.3.1_beta-cuda12-x86_64-linux.deb
{% endhighlight %}

<b>6</b>. Build and run the samples:

{% highlight bash %}
cd ~/samples 
./scripts/build_samples.sh
./scripts/run_samples.sh
{% endhighlight %}

<b>7</b>. Check the sample run results in the local terminal outside the docker:

{% highlight bash %}
# Check the running docker containers and get the container id of the sample run:
docker ps
# copy the output file to local machine (example below):
docker cp <container_id>:/tmp/out_Weimaraner.jpg ~/out_Weimaraner.jpg
{% endhighlight %}


#### Run segmentation triton sample

Triton server docker run the <a href="https://github.com/CVCUDA/CV-CUDA/tree/release_v0.3.x/samples/segmentation_triton">segmentation on triton example</a> with corrections.

##### Set up triton server

<b>1</b>. Find the right tritonserver release docker image that matches our devbox setup (<a href="https://docs.nvidia.com/deeplearning/triton-inference-server/release-notes/index.html">release note</a>, <a href="https://catalog.ngc.nvidia.com/orgs/nvidia/containers/tritonserver/tags">tags</a>)

I'll go with <a href="https://docs.nvidia.com/deeplearning/triton-inference-server/release-notes/rel-23-02.html">image 23.02-py3</a>.

```
Install the libs first as in the last section.
```

<b>2</b>. Start the triton server docker with the cvcuda mounted:

{% highlight bash %}
docker run --shm-size=1g --ulimit memlock=-1 -p 8000:8000 -p 8001:8001 -p 8002:8002 --ulimit stack=67108864 -ti --gpus=all -v /opt/nvidia/cvcuda0:/cvcuda -w /cvcuda nvcr.io/nvidia/tritonserver:23.02-py3
{% endhighlight %}

<b>3</b>. Install dependencies on the triton server docker:

{% highlight bash %}
cd /cvcuda/samples/scripts
touch my_install_dependencies.sh
vim my_install_dependencies.sh

# copy the following code into the my_install_dependencies.sh
{% endhighlight %}

The code to be copied:

{% highlight bash %}
mkdir -p ~/samples
cd /cvcuda
cp -r samples/. ~/samples

cd ~/samples
chmod a+x scripts/*.sh
chmod a+x scripts/*.py

sed -i -e '55s+.*+pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu121\npip3 install av==10.0.0 pycuda nvtx==0.2.5\npip3 install onnx\n+' scripts/install_dependencies.sh

sed -i -e 's+$PATH+\\${PATH}+' scripts/install_dependencies.sh

# add new line after torchnvjpeg repo is downloaded
sed -i "$(grep -n 'scripts/install_dependencies.sh' -e 'cd torchnvjpeg' | head -1 | cut -f1 -d:)ised -i -e 's/std=c++14/std=c++17/' torchnvjpeg/setup.py" scripts/install_dependencies.sh

# install dependencies. (Q: How to skip user ENTER while running the script?)
./scripts/install_dependencies.sh
{% endhighlight %}

Install dependencies:

{% highlight bash %}
bash my_install_dependencies.sh
{% endhighlight %}

<b>4</b>. Install cmake that meets the version requirement into `/usr/local/bin` which comes before `/usr/bin` in system `PATH` where the existing cmake is.

{% highlight bash %}
cmake --version
which cmake

cd /opt
curl -L https://github.com/Kitware/CMake/releases/download/v3.26.5/cmake-3.26.5-linux-x86_64.sh --output cmake-3.26.5-linux-x86_64.sh
chmod a+x cmake-3.26.5-linux-x86_64.sh
bash /opt/cmake-3.26.5-linux-x86_64.sh

ln -s /opt/cmake-3.26.5-linux-x86_64/bin/* /usr/local/bin

# export CMAKE_ROOT=/opt/cmake-3.26.5-linux-x86_64/share/cmake-3.26
# Remember to refresh after new cmake is installed
hash -r

cmake --version 
{% endhighlight %}

<b>5</b>. Build the samples

{% highlight bash %}
cd ~/samples
cp ./scripts/build_samples.sh ./scripts/my_build_samples.sh
sed -i -e 's+cmake ..+cmake .. -DCMAKE_PREFIX_PATH=/cvcuda/lib/x86_64-linux-gnu/cmake+' scripts/my_build_samples.sh
./scripts/my_build_samples.sh
{% endhighlight %}

<b>6</b>. Start the triton server:

{% highlight bash %}
# Add the lib path to the libcvcuda.so.0
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/cvcuda/lib/x86_64-linux-gnu/

# To resolve "ModuleNotFoundError: No module named 'cvcuda'"
# option 1: go to the folder where the cvcuda.cpython-38-x86_64-linux-gnu.so is installed
cd /cvcuda/lib/x86_64-linux-gnu/python

# option 2: create symlink in one of the default sys.path
ln -s /cvcuda/lib/x86_64-linux-gnu/python/* /usr/lib/python3/dist-packages

# Start the triton server
tritonserver --model-repository ~/samples/segmentation_triton/python/models 
{% endhighlight %}

##### Set up triton client
<b>1</b>. Start docker to run triton client:

{% highlight bash %}
docker run -ti --net host --gpus=all -v /opt/nvidia/cvcuda0:/cvcuda nvcr.io/nvidia/tritonserver:23.02-py3-sdk -w /cvcuda /bin/bash
{% endhighlight %}

<b>2</b>. Install dependencies into the client docker run:

{% highlight bash %}
cd /cvcuda/samples/scripts
bash my_install_dependencies.sh
{% endhighlight %}

<b>3</b>. Run the segmentation on folder containing images:

{% highlight bash %}
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/cvcuda/lib/x86_64-linux-gnu/

# option 1. hack python sys.path to include the lib folder
sed -i -e "s+import cvcuda+sys.path.append('/cvcuda/lib/x86_64-linux-gnu/python')\nimport cvcuda+" ~/samples/segmentation_triton/python/triton_client.py

# option 2. create symlink (Q: How default sys.path is set?)
ln -s /cvcuda/lib/x86_64-linux-gnu/python/* /usr/lib/python3/dist-packages

python3 ~/samples/segmentation_triton/python/triton_client.py -i ~/samples/assets/images -b 2

# check the results
ls /tmp
{% endhighlight %}

