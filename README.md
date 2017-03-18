QEMU-free KVM Connector/Client/Controller

===============================================================================
[What am I]
KVMC is ...	isolation & simplification

[Why are we]
1. Google Cloud Platform ...
2. IOMMU, SR-IOV, VT-d, VFIO, ...
3. Qemu code base, ease of customization ...
4. Existing implementation (novmm+ limits itself to GCP env, that is closed source,
   and requires special hw commonly found at Google)

[Where are we]
...


[Reference]
[https://cloudplatform.googleblog.com/2017/01/7-ways-we-harden-our-KVM-hypervisor-at-Google-Cloud-security-in-plaintext.html]
[https://news.ycombinator.com/item?id=13484926]
[https://systems.cs.columbia.edu/projects/kvm-arm/]
[https://github.com/google/novm]
===============================================================================

Non-QEMU implementation: Google does not use QEMU, the user-space virtual machine monitor and hardware emulation.
Instead, we wrote our own user-space virtual machine monitor that has the following security advantages over QEMU:

  Simple host and guest architecture support matrix. QEMU supports a large matrix of host and guest architectures,
  along with different modes and devices that significantly increase complexity. Because we support a single
  architecture and a relatively small number of devices, our emulator is much simpler. We don't currently support
  cross-architecture host/guest combinations, which helps avoid additional complexity and potential exploits.
  Google's virtual machine monitor is composed of individual components with a strong emphasis on simplicity and
  testability. Unit testing leads to fewer bugs in complex system. QEMU code lacks unit tests and has many
  interdependencies that would make unit testing extremely difficult.

No history of security problems. QEMU has a long track record of security bugs, such as VENOM, and it's unclear
what vulnerabilities may still be lurking in the code.

##############################################################################
From the original announcement email:
-------------------------------------------------------
The goal of this tool is to provide a clean, from-scratch, lightweight KVM host tool implementation that can boot
Linux guest images (just a hobby, won't be big and professional like QEMU) with no BIOS dependencies and with only
the minimal amount of legacy device emulation.

It's great as a learning tool if you want to get your feet wet in virtualization land: it's only 5 KLOC of clean
C code that can already boot a guest Linux image.

Right now it can boot a Linux image and provide you output via a serial console, over the host terminal, i.e. you
can use it to boot a guest Linux image in a terminal or over ssh and log into the guest without much guest or host
side setup work needed.
--------------------------

The guest kernel has to be built with the following configuration:
 - For the default console output:	CONFIG_SERIAL_8250=y,CONFIG_SERIAL_8250_CONSOLE=y
 - For running 32bit images on 64bit hosts:	CONFIG_IA32_EMULATION=y
 - Proper FS options according to image FS (e.g. CONFIG_EXT2_FS, CONFIG_EXT4_FS).
 - For all virtio devices listed below:	CONFIG_VIRTIO=y, CONFIG_VIRTIO_RING=y, CONFIG_VIRTIO_PCI=y
 - For virtio-blk devices (--disk, -d):		CONFIG_VIRTIO_BLK=y, CONFIG_SCSI_VIRTIO=y
 - For virtio-net devices ([--network, -n] virtio):	CONFIG_VIRTIO_NET=y
 - For virtio-9p devices (--virtio-9p):	CONFIG_NET_9P=y, CONFIG_NET_9P_VIRTIO=y, CONFIG_9P_FS=y
 - For virtio-balloon device (--balloon):	CONFIG_VIRTIO_BALLOON=y
 - For virtio-console device (--console virtio):	CONFIG_VIRTIO_CONSOLE=y
 - For virtio-rng device (--rng):	CONFIG_HW_RANDOM_VIRTIO=y
 - For vesa device (--sdl or --vnc):	CONFIG_FB_VESA=y

And finally, launch the hypervisor as root:
  ./kvmc run --disk example/linux.img --kernel example/bzImage --network virtio -sdl

See the following thread for original discussion for motivation of this project:
http://thread.gmane.org/gmane.linux.kernel/962051/focus=962620

Another detailed example can be found in the lwn.net article:
http://lwn.net/Articles/658511/