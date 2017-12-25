---
layout: post
title:  "Ubuntu, TensorFlow, and Nvidia"
author: Joe Meyer
date:   2016-01-01 21:37:38 -0600
categories: nvidia tensorflow ubuntu
---
The last week I've been migrating my main desktop machine from Windows 10 to Ubuntu. The main reason for the transfer was to have a unified development environment for my brother and I to work on our TensorFlow application. TensorFlow does not run on Windows (easily) and the only reason I was using Windows was for gaming, which will be an uphill battle in Ubuntu.

TensorFlow has a nice feature to allow computation using Nvidia CUDA technology. Luckily, I have a 980 Ti for gaming which does extremely well at CUDA computation. This card was the main headache during this whole process.

If anyone has a new Nvidia card and has had a ton of issues with [low graphics mode][low-graphics-mode], getting CUDA setup, or overall Ubuntu configuration, hopefully this information will help you.

First of all do NOT use:
{% highlight bash %}
sudo apt-get install nvidia-current
{% endhighlight %}

The driver this will install is around the 302 version at the time of writing this post. The newest version according to the [Nvidia site][nvidia-site] is 352 for long-lived and 358 for short-lived.

Until you get a working driver up don't think about trying to have a GUI. I would disconnect my 980 Ti, install Ubuntu 14.04 LTS, and everything would work fine. When shutdown occurs, I plugged in the GPU and booted up, only to be greeted by 'low graphics mode' which would not respond if I told it to use low graphics mode and would only allow the command line interface with Ubuntu. Even if I tried to install nvidia-current beforehand, it would default to this behavior.

From what I have seen, that driver supports up to around the 600 series, which is extremely outdated for 'current'. For some reason, even the ppa for the nvidia drivers had the same behavior of low graphics mode, and no nvidia-xconfig being present.

I thought I finally figured it out when I installed the latest driver straight from Nvidia. The version was 352.63 and I installed with the runfile and did all proper pre and post steps. This allowed me to finally boot into Ubuntu (YAY!) and I thought all was right with the world. Now onto TensorFlow...

For GPU enabled computation you will need the [CUDA Toolkit 7.0][cuda-toolkit] and [CUDNN Toolkit 6.5][cudnn-toolkit]. These are both older versions of their respective toolkits, and the links will go to the archived versions. Upon downloading the CUDA toolkit (install this first), you will execute the run file and follow the prompts. This is where the second main snag came in - the toolkit needed at least version 345 (I had 352) of the Nvidia drivers. It didn't see mine as new enough.

Easy enough, I'll simply uninstall the existing driver and use the one bundled with the toolkit (it will prompt to ask if you want this driver). It goes through but again the computer doesn't fully recognize the 980 Ti and just sees a generic GPU. When trying to run a TensorFlow neural net it will spit out
{% highlight bash %}
>no CUDA-capable device is detected
{% endhighlight %}

This was from Ubuntu not fully seeing the GPU. The last step was to sneak in the newer driver installed earlier that we had to removed to install the CUDA toolkit. It blocked the installation even though it was new enough, but the one it installed wasn't quite up-to-date to support the new Maxwell chip in the 980 Ti. You can easily install the newest driver by going to [Nvidia's site][nvidia-site] or simply execute
{% highlight bash %}
sudo apt-get install nvidia-352
{% endhighlight %}

Don't forget to reconfigure your xconfig file using
{% highlight bash %}
nvidia-xconfig
{% endhighlight %}

After all this, setup (follow the CUDA and CUDNN instructions on their sites) I was finally ready to harness the 980 Ti for TensorFlow. Follow the instruction for 
{% highlight bash %}
./configure
{% endhighlight %}
within the TensorFlow repo you'll need to download. That passing is the first step to success. After this, fire up one of the tutorial files in the TensorFlow website, or visit [my repo][tf-tutorials] to get those bundled. Upon running these, you'll likely get a similar error as earlier with the no CUDA-enabled device detected. Go to where the CUDA Toolkit installed the samples (if you didn't download them re-run the CUDA run file and hit 'yes'. Default installation is at 
{% highlight bash %}
/home/user/NVIDIA_CUDA-7.0_Samples
{% endhighlight %}

Build all the samples and run 
{% highlight bash %}
cd /home/username/NVIDIA_CUDA-7.0_Samples/
make
cd bin/x86_64/linux/release/
sudo ./deviceQuery
sudo ./bandwidthTest
{% endhighlight %}

If the deviceQuery command says it can't find a GPU, the drivers are not up to date and you should reinstall your 352 drivers. If that passes, run the bandwidthTest to make sure it can actually send information to and from the GPU. After all of these pass, try and run a TensorFlow script again and VIOLA! You should be running through epochs much faster than before. We were running batches of 200 images and each epoch took around a second on the CPU, and when I finally was able to run on the 980 Ti it was about 15 a second.

Every time I reboot, there seems to be an 'unlinking' of the CUDA Toolkit. You can tell when this happens by trying to run a TensorFlow script and it defaults to CPU mode. Simply rerun the deviceQuery script again, it will give you another 'pass' message, and then tensorflow should see your GPU.

I hope if you ran across this post and have had similar issues you were able to resolve them. It was a painful transition and I almost admitted defeat multiple times and was about to switch back to Windows. I'm glad I didn't.

[nvidia-site]: http://www.nvidia.com/object/unix.html
[cuda-toolkit]: https://developer.nvidia.com/cuda-toolkit-70
[cudnn-toolkit]: https://developer.nvidia.com/rdp/cudnn-archive
[tf-tutorials]: https://github.com/JEMeyer/tensorflow-tutorials
[low-graphics-mode]: http://askubuntu.com/questions/141606/how-to-fix-the-system-is-running-in-low-graphics-mode-error
