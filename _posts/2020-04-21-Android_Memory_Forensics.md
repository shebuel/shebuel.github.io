---
layout: post
date: 2020-04-21
title: 'Android Memory Forensics'
tags: [Android, Memory Forensics]
featured_image_thumbnail:
featured_image: assets/images/posts/2020/android.jpg
featured: true
---

***Disclaimer**:
This post is either a very good or a horrible representation of my writing skills depending on how much you like it*


## Memory Forensics:

Memory forensics is essentially just a subset in the computer forensic field where the goal is to focus on the contents of snapshot of memory (RAM) rather than the contents of the hard disk. Memory forensics has its own set of difficulties the worst of which is that the content in memory is volatile meaning once the machine loses power all the contents of memory are pretty much gone for good (Unless you manage to instantly freeze but I am not sure about the science behind that). Memory forensics has been gaining a lot of traction recently mostly due to the fact that there are a lot more memory resident malware and memory resided data often provide a fresh real-time perspective to an investigation case.


## Memory forensics in android devices:

Android which is currently the most popular mobile OS is a framework which runs on top of the Linux kernel. So many things that apply to UNIX systems also apply to Android except the way that applications are run (used to be Dalvik VMs, now Android Run Time (ART) environment). The important thing in regards to this project is that Android similar to Linux allows for loadable kernel modules (LKMs). (*My future self: "Oh my sweet summer child"*)

Ways to extract memory
- /dev/(k)mem
- Fmem
- Crash
- Wont get into the hardware ways (Jtag maybe be a possible way but hard to get to w/o removing the battery)

There is a very good presentation from the creators of the tool that I am about to use which you can find [here](https://www.youtube.com/watch?v=oWkOyphlmM8)

## My test environment

- Nexus 5 
	- Android 6.0.1 (Latest update, EOL)
	- kernel version is 3.4.0-gcf10b7e
	- Rooted
- SIFT Workstation
	- Volatility
	- Ubuntu 14.04
	- Kernel 4.4.0-177-generic
- USB to Micro USB cable

Okay. I know what you are thinking. Hey Sheb, that's kind of convenient that the phone that you are trying to analyze is just randomly rooted and I agree. What are the odds right. I guess I am just that lucky. \*wink\* \*wink\*

The reason I chose the SIFT workstation is only because it had Volatility pre-installed and I wouldn't need to setup anything. That being said the steps should be similar in any other Linux machine I think.

## LiME

LiME extractor is what I am going to be using for extracting the memory for analysis.

Just from reading the documentation and having a quick look at the video this is what I learned...

* LiME uses loadable kernel modules to load a kernel module directly into a running system (Duh!)
* Once the kernel is loaded the kernel checks the /proc/iomem to figure out the location of the "System RAM" and then uses that physical memory address to carve it out.
* It then sends the data back to the TCP port that was setup when making the kernel module or stores it in the certain location in the SD card.

I think one of the best things I learned in life is from [Ippsec](https://ippsec.rocks/#). It is to have a good understanding of the tools that you use so that If you are in a pinch and do not have access to the tools you can create a rudimentary version yourself.
So all thats left to do is look at the source code for the lime extractor and see if I have guessed everything right... This should be fun ( I meant that in the most sarcastic way possible... End my life already)


The following code found in main.c of the lime iterated through all the entries in /proc/iomem and if the name is "System RAM" then see whether the user selected Lime format or the padded format and creates the respective header and then copies the pages represented by that memory range using the `write_range()` function.

```c
for (p = iomem_resource.child; p ; p = p->sibling) {

        if (strcmp(p->name, LIME_RAMSTR))
            continue;

        if (mode == LIME_MODE_LIME && write_lime_header(p) < 0) {
            DBG("Error writing header 0x%lx - 0x%lx", (long) p->start, (long) p->end);
            break;
        } else if (mode == LIME_MODE_PADDED && write_padding((size_t) ((p->start - 1) - p_last)) < 0) {
            DBG("Error writing padding 0x%lx - 0x%lx", (long) p_last, (long) p->start - 1);
            break;
        }

        write_range(p);
```

The structure for `iomem_resource` can be found in resource.h 

```c
struct resource iomem_resource = {
   36     .name   = "PCI mem",
   37     .start  = 0,
   38     .end    = -1,
   39     .flags  = IORESOURCE_MEM,
   40 };
```
```c
struct resource {
        resource_size_t start;
        resource_size_t end;
        const char *name;
        unsigned long flags;
        struct resource *parent, *sibling, *child;
    };
```

The `write_range()` function gets a resource structure and checks if the page is the same as the page size or it needs to be padded. It then calls the `write_vaddr()` function which in turn calls the `write_vaddr_tcp()` or the `write_vaddr_disk()` function based on how the memory is to be stored.

```c
 if (no_overlap) {
                copy_page(vpage, v);
                s = write_vaddr(vpage, is);
            } else {
                s = write_vaddr(v, is);
            }

```
^ Code from `write_range()` function

```c
/* Run deflate() on input until output buffer is not full. */
        do {
            ret = try_write(deflate_page_buf, deflate(v, is));
            if (ret < 0)
                return ret;
        } while (ret == PAGE_SIZE);
```
^ Code from `write_vaddr()` function

```c
 ret = RETRY_IF_INTERRUPTED(
        (method == LIME_METHOD_TCP) ? write_vaddr_tcp(v, is) : write_vaddr_disk(v, is)
    );
```
^ Code from `try_write()`

I spent like 5 hours analyzing all the code ( I have never made a Kernel module before) and then realized that "The Art of Memory Forensics" has already explained the code in its notes section. Currently reconsidering life decisions. Send help!

## Memory Extraction:

### Obtaining the kernel config:


So getting the kernel config sounded super easy when looking at the lime documentation but it actually turned out much harder than I expected.

So this is what the lime documentation has to say about obtaining the kernel config

Installed the prerequisites, downloaded the NDK from [here](https://github.com/android/ndk/wiki/Unsupported-Download) and downloaded the SDK and kernel from official android source. I just created a file with all of the paths just so the I can source it from there every time. My source file looked something like this...

![exports](/assets/images/posts/2020/exports.png )

We must then retrieve and copy the kernel config from our device. As per the LiME docs, 

```bash
cd $SDK_PATH/platform-tools
./adb pull /proc/config.gz
gunzip ./config.gz
cp config $KSRC_PATH/.config
```
Simple enough... So lets try that..

Hmm... Thats strange... That file does not exist... \**Spends 30 mins spell checking and finds nothing*\*

So time to Google...

I found out that only kernels that are configured with the `CONFIG_IKCONFIG` or the `CONFIG_IKCONFIG_PROC` ( Which are usually not enabled by default ) are enabled will have the the config available in /proc/config.gz.

Some phones also have the config.gz stored in /vendor/kernel so be sure to check that out too.

I also find that devices which have CWM (clockworkmod) have that configuration enabled by default. It's kind of hard to assume that the suspect who I want to analyze is the same amount of a nerd I am and has modded his phone. Apparently some Xperia phones and a lot of Huawei phones also have it enabled by default. So if that's what you have to analyze you are in luck. Must. Be. Nice.

Okay, this is going to be a little more complicated than I initially thought it would be...

At this point I should have just built the config file myself from the kernel source but I wanted to take the easy way out and just find a way to get it from the device. My argument was that "Hey, that will be the most accurate config file". The problem with that argument was that I am using a Nexus device and that is almost always going to be built on the default config.


So I tried using the method that is mentioned in [this article](https://jiafei427.wordpress.com/2016/11/01/how-to-extract-kernel-configuration-from-android-kernel-or-boot-img-and-build-related-kernel-for-the-device/) to extract the kernel image from the device and then use the extract-ikconfig.sh found in scripts/ folder.


// Fun fact... U can just get the zImage of the kernel directly from android source.

After years of struggling I managed to obtain the boot.img file and extract the kernel image from that. Its finally time to use the script and get the kernel config.

![extract-ikconfig terminal](/assets/images/posts/2020/extract_ikconfig.png)

Well that did not go as expected... *Spends some more years making sure that he did everything right*

Well there is no way there is another config setting for this right? RIGHT?

![Extract IKconf](/assets/images/posts/2020/extract_ikconf.png)

![gif](https://media.giphy.com/media/5saWnCIJL7nmU/source.gif)

*Finally accepts his fate and compiles the config from the source code.*


// The source that I am using is the one I obtained that I got by just Google searching. I think a better way to do this would have been to get the one for a Qualcomm msm chipset from the Google source. I honestly don't know if this is going to bite me in the ass later on.

These are the commands I used from the source directory:

```bash
export ARCH=arm
export SUBARCH=armexport
CROSS_COMPILE=$CC_PATH/arm-linux-androideabi-

make hammerhead_defconfig
```

Now that I finally have the .config file I can follow the LiME docs again to finish my work.

Okay, I just have to make changed to the Makefile(as per the LiME docs) and run make. Seems simple enough.

I did not take a screen shot of the error but this is the error message I got:

"The present kernel configuration has modules disabled.Type 'make config' and enable loadable module support.Then build a kernel with module support enabled
make:  [modules] Error 1"

I found the solution from a similar blog post to the one I am making. Wait someone already did all this work??? AHHHHHHH.

You can find the post [here](https://sgros-students.blogspot.com/2014/04/lime.html)

I just had make changes to the config file and add the following lines.

```
CONFIG_MODULES=y
CONFIG_MODULE_UNLOAD=y
```

\**At this point I was honestly just happy to find a solution and didnt really think about what the solution did but I later realized a very big problem that I had*\*

Well that worked and I obtained a lime.ko file. 

Again following the LiME docs I did the following commands to push my .ko file to the device and the load the kernel module.

```bash
adb push lime.ko /sdcard/lime.ko
adb forward tcp:4444 tcp:4444
adb shell
su
# insmod /sdcard/lime.ko “path=tcp:4444 format=lime”
```

![insmod terminal image here](/assets/images/posts/2020/insmod.png)

Wait that's not right... There is no way it's a config problem again right? RIGHT?

![Android kernel config page](/assets/images/posts/2020/kernel_config.png)

Obviously if I had noticed what changes I am adding to the config file when making lime.ko I would have realized that for android to accept LKM it has to have those config options enabled which wasnt in the default config and obviously wouldn't in the actual device.

The only method I can see working now is to flash a custom kernel with those configs enabled but that would require a reboot which kind of makes a large part of why I am doing this go down the drain.

So I shall end this post here. I may make another blog on playing around with volatility with an android memory capture just to see all the information that I could gain.

(link for ndk)[https://github.com/android/ndk/wiki/Unsupported-Download]
~

















