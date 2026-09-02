---
title: "Google"
description: "Internship at Google"
publishDate: "September 2026"
tags: ["c", "work"]
---

I interned in the Platform and Devices product area, specifically working on
the Pixel Core Platform - Chassis team. The Chassis team provides low level
system software for key Pixel infrastructure. Chassis specifically works on
Linux device drivers for IOMMUs, interrupt controllers, timers, watch dogs,
system coherency, and other system level components. I worked on the IOMMU, a
hardware device that controls device DMA and allows isolation between device
access to system memory.

My code is on Pixel 11 phones (at the time of this writing) and will be on
future Pixel phones when forward ported. My key contributions were increasing
system visibility with end to end Telemetry and system level improvements in
terms of reduced TLB invalidation latencies and reduced memory footprint.

Source code should eventually be available in the Google Android source, linked
in the Droid picture.
<a
    href="https://android.googlesource.com/kernel/common/+/refs/heads/android16-6.12/drivers/iommu/arm/arm-smmu-v3/"
    target="_blank" rel="noopener noreferrer">
  <img
    src="/droid.png"
    alt="Android robot"
    class="h-100 w-100 rounded-full object-cover"
  />
</a>

## ARM SMMU v3
ARM SMMU v3 is ARM's IOMMU specification. The key difference with this
specification is in memory queue based command insertion rather than register
based control. It also includes various translation configurations for
different needs. MMU 700 is a specific implementation of the ARM SMMU v3
specification. Throughout the internship, I spent a lot of time reading and
understanding the architecture spec as well as the MMU 700 Technical Reference
Manual. This was a new but exciting experience for me. I really enjoyed working
closely with the hardware. Specifically, I wrote code that directly influenced
hardware behavior by writing commands into memory (typical shared memory queue
and consumer produced indices).

As you may have noticed, the IOMMU is very similar to the MMU on a CPU. The MMU
protects memory from CPU accesses while the IOMMU protects memory from device
accesses. Similar concepts such as page tables, TLB management, memory mappings
apply to the IOMMU. However, there are some distinct differences. When we think
about the CPUs memory management subsystem and its clients, we typically think
about it in the context of user space processes. Thus concepts such as forking
and copy on write apply. However, the clients of the IOMMU subsystem are
typically DMA capable devices on the system such as NIC, modem, camera, GPU,
TPU, etc. The device drivers of these peripherals will set up domains and
translation mappings and the device will initiate memory accesses.

## Android and pKVM in the context of IOMMU
Android maintains a fork of upstream Linux called Android Common Kernel. One
key difference between upstream Linux and ACK is the KVM infrastructure. The
KVM subsystem allows kernel based virtualization where guest OSes can run on
top of the host OS. pKVM (also referred to as a hypervisor) is Android's
extension of KVM for confidtential protected VMs. The key difference is that
the hypervisor portion (referred to as pKVM) runs in a higher privilege level
(EL2) while the host kernel remains at EL1. This allows guest OSes
confidentiality and integrity to be protected even if the host kernel is
compromised.

pKVM is relevant in the context of IOMMU because the IOMMU subsystem also needs
to ensure host kernel isolation from pKVM and pVMs to prevent host kernel DMA
into protected memory regions. To do this, our version of the ARM SMMU v3
driver had a host kernel portion and a pKVM portion as well as various
translation configurations. To ensure host kernel isolation, the host kernel's
memory mappings (referred to as stage 2 translation config or IDMAP) is managed
by pKVM. pKVM will unmap memory regions donated to protected from the host
kernel's memory mapping, making that region inaccessible by the host kernel.
SMMU integration is shared with the EL1 host and EL2 pKVM with pKVM handling
the memory isolation.

Peripheral devices have the option to use different translation configuration.
Stage 1 only translation allows a driver to program mappings using I/O Virtual
Addresses (IOVA) to physical addresses (PA), similar to a how a user space
program uses VA instead of raw physical addresses. Similar to the abstraction
that virtual memory provides for user space, stage 1 translation and IOVAs
provide the abstraction of contiguous memory while the underlying mapped pages
are non-contiguous and a large address space that exceeds the capacity of DDR
size.

Devices can also choose a nested translation configuration where two
translations are applied, stage 1 then stage 2 for further virtualization. If a
guest OS wants to maintain mappings for multiple virtualized devices, it can do
so by maintaing multiple stage 1 mappings per device / streams. pKVM isolates
this guest OS by controlling the underlying stage 2 mappings. Nested
translation in IOMMU works similarly to how nested translation works in the
CPUs MMU when there is hardware supported virtualization. The guest OS manages
its own stage 1 translation mappings that map IOVA to Intermediate Physical
Addresses (IPA). The hypervisor maintains a mapping from IPA to PA.

## IOMMU Telemetry
The first project I worked on was extending the existing telemetry framework
for IOMMU. I instrumented various parts of the codebase, integrated with sysfs
to export telemetry data to userspace, and wrote C++ code that collected this
sysfs data and exported it to a central database.

This involved understanding the aforementioned translation configurations and
abstractions provided by the IOMMU, the intricacies of pKVM, and how boot time
memory donations between the host and pKVM work. Exporting to userspace was
relatively easy as the infrastructure for sysfs is very robus and simple to
implement.

One interesting implementation detail that I knew of but did not work with
first hand is how *"object oriented programming"* can be done in C. In the
device driver infrastructure, OOP is done like in the following
```C
struct arm_smmu_device {
	struct device			*dev;
...
```
In the above example, `dev` is a pointer to a generic device. This is what the
device driver infrastructure uses to refer to the specific arm smmu device
which is transparent to the infrastructure. The ARM SMMU v3 driver assosciates
this in its own struct and also registers itself to `*dev` using
`platform_set_drvdata`. When you want to go from the `*dev` to the parent
wrapping `arm_smmu_device`, you can use `dev_get_drvdata`. This way, you can
maintain a one to one relationship between the two structs while not leaking
abstractions to the device driver infrastructure.

Sysfs infrastructure also makes it very easy to go from a generic `kobject` to
a specific device driver specific object. You can register with the sysfs
infrastruture by doing the following:
```C
struct my_sysfs {
    struct kobject kobj;
    ...
};
```
The macros such as `__ATTR_RO` and `ATTRIBUTE_GROUPS` do the heavy lifting in
terms of writing the boiler plate to register with the sysfs framework. The
`kobj` embedded in `my_sysfs` can be used to go from a generic `*kobj` to the
struct that wraps that `*kobj` using `container_of`. `container_of` essentially
takes the offset of the `kobj` within the struct and subtracts the pointer by
that offset to get the outer struct.

Overall, I found the sysfs framework very convenient to work with, especially
for doing things like microbenchmarking. It was very simple to write some code
that takes the start and end times of a routine and export them via sysfs.

I also did a major refactoring of the C++ program that collected statistics
from sysfs. I leveraged structs that were autogenerated based on protobuf
definitions that someone else on the team had setup for a different component
to safely read and store data instead of using brittle vector indexing math. I
also spent some time using more modern C++ 17 features such as `std::optional`
and `std::filesystem::directory_iterator`. While this was relatively simple
refactoring that LLMs were very good at, I found it a little frustrating how
prone to exceptions the STL is. The model that my environment followed was
avoiding exceptions and the `std::filesystem::directory_iterator` STL methods
all threw exceptions by default unless overloaded with `std::error_code`
arguments.

## TLB Invalidation Optimizations
I also spent a good chunk of time improving our TLB invalidation routines,
working on algorithmic and architectural improvements to reduce the time we
spend doing invalidations. For a bit of background TLB invalidation happens
whenever there is an unmapping. It is important that we do invalidations
because if a translation from IOVA/IPA to PA remains in the TLB, future
accesses may use this cached translation resulting and thus accesses to memory
that should not be allowed. TLB invalidations can happen in essentially two
different ways, range based by sepcifying a start and end or by invalidating
all entries associated with a certain ID (VMID or ASID). The majority of time
spent doing invalidations is waiting on the SMMU to process some commands. When
doing invalidations, we insert commands into the command queue as such
```text
Command queue (producer -> consumer)

[ CMD_TLBI_IPA ] -> [ CMD_TLBI_NHALL ] -> [ CMD_SYNC ]
                                             ^
                                   completion barrier
```
We know that the hardware is done processing all of these commands when it
finishes processing CMD_SYNC. This is an architectural guarantee that the
consumer index for CMD_SYNC is only incremented when all previous commands and
CMD_SYNC are **processed**

The existing TLB invalidation routines went as following. We iteratred through
each of the SMMUs that we had to invalidate and inserted all three of the
commands and waited for it synchronously. This meant that there was no parallel
work happening on the SMMUs and each invalidation happend sequentially.

```text
SMMU 0: [insert][==inval/wait==]
SMMU 1:                        [insert][==inval/wait==]
SMMU N:                                               [insert][==inval/wait==]
```

The first alogrithmic improvment is to parallelize this process. This sequence
of command insertion is very analagous to the head of line blocking problem,
which appears in HTTP. The optimization that I took on was to insert all of
the commands into the queue but skip the waiting until all commands are
inserted in all command queues. Then, in a separate iteration, I waited on the
command queue to ensure the commands were consumed and executed on the SMMU
side. This overlapped the time the SMMUs spent invalidating TLB and the time
the CPU spent waiting on the SMMUs.

```text
SMMU 0: [insert][==inval/wait==]
SMMU 1:        [insert][==inval/wait==]
SMMU N:               [insert][==inval/wait==]
```

The second optimization was more architectural. Previously, we were not taking
advantage of the pKVM `host_stage2_idmap_complete` that is available to be
defined in `kvm_iommu_ops`. This API is called after a series of stage 2 map or
unmaps. The reason this API exists is because map and unmaps happen in bursts.
Whe mapping/unmapping non-contiguous memory chunks, distinct calls are made
instead of one large calls. I took advantage of this by batching
invalidation commands and waiting until the very last moment to wait on the
SMMU when inserting TLB invalidation commands. Without this optimization,
multiple unmaps look like this:

```text
SMMU 0: [insert][==inval/wait==]             [insert][==inval/wait==]
SMMU 1:        [insert][==inval/wait==]             [insert][==inval/wait==]
SMMU N:               [insert][==inval/wait==]             [insert][==inval/wait==]
```

However, with batched TLB invals, the time spent waiting is reduced.

```text
SMMU 0: [insert]             [insert][==inval/wait==]
               [==inval in background==]
SMMU 1:        [insert]             [insert][==inval/wait==]
                      [==inval in background==]
SMMU N:               [insert]             [insert][==inval/wait==]
                             [==inval in background==]
```

The amount of benefit seen from this is dependent on the batching factor, ie
the number of unmap calls per complete callback. While I cannot share exact
numbers, end to end I was able to roughly reduce the time spent doing TLB
invalidations in half.

## Page Table Management Optimizations
Another problem I worked on was freeing empty page tables. Linux uses a
multi-level page table (also known as a radix tree) to manage mappings. The
number of levels can vary between three and four, sometimes even just two
depending on factors such as page sizes, the descriptor formats that are
supported, and the type of optimizations that are available (ie huge pages).

One inherent problem with this radix tree data structure is the existance of
empty page tables. In the IOMMU subsystem and to some extent the MM subsystem,
a page table that has no valid mappings is left in memory, even though it is
technically unused. It is freed under very narrow conditions such as domain
deallocation and block size aligned IOVA and size unmaps (block sizes is the
size that can be supported by an entry in a page table at level N, at the
lowest level N it is 4K assuming a 4K page system, at N-1 it is 2MB, at N-2 it
is 1GB). This is due to various factors such as not wanting to make the unmap
path slow and the fact that these page tables in theory can be used again if
there is a mapping to that IOVA again.

The telemetry system that I worked on informed us that the degree to which
these empty page tables occured was significant (though I won't share the exact
number), and that freeing these empty page tables could reduce memory pressure.

To free these empty page tables, I had to solve two problems: 1) figuring out
which tables could be freed efficiently 2) figuring out when and how to free
these empty page tables.

Naively, we can figure out if a page table is empty by doing a full scan of the
page (512 iterations for 4K, 2048 for 16K pages). This was not an acceptable
solution as there could be 512^3 empty page tables just in the leaf tables and
doing a full scan of each of these pages would be too expensive regardless of
when we do it. A solution we came to was to use extra metadata to avoid this
full scan. The question then was what to store and where to store it. One thing
that we did not want to do was add additional development complexity, ie any
solution that had to manage the lifetime of another data structure along with
the page table trees. Thus the solution we came to was to use the available
bits in the page table descriptors to encode the number of valid entries there
were in the next level page table. This way, we could avoid doing a full 512
iteration scan of an entire page on every attempt at freeing a table.

```text
Parent page-table descriptor

+--------------------------+------------------------------+
| next-level table address | unused bits: valid_count = 3 |
+--------------------------+------------------------------+
             |
             v
Child page table
+--------+--------+--------+-----+--------+
| valid  | valid  | valid  | ... | empty  |
+--------+--------+--------+-----+--------+

On unmap: valid_count--

valid_count == 0  ->  child table is empty  ->  free it
valid_count > 0   ->  entries remain        ->  keep it

No 512-entry scan is needed.
```

Solving the second problem was a bit trickier. Naively, you can free as soon as
a page table becomes empty, ie when the last valid entry becomes invalid.
However, this was not feasible as it would drastically regress the performance
of the map and unmap APIs which clients depended on. Thus the only option was
to defer this work outside of the hot path. There were several ways to do this
but the approach we chose was to integrate with the MM shrinker subsystem. The
shrinker is system that gets invoked whenever there is memory pressure on the
system, ie GFP_KERNEL allocations are failing or the watermark of the buddy
allocator reaches some threshold. Other subsystems in the kernel can register
callbacks with the shrinker and get invoked whenever there is memory pressure.
This was an ideal solution for us as we could avoid prematurely freeing page
tables when there wasn't memory pressure. I won't go over the exact
implementation details but roughly, the control flow looks like the diagram
below. The host kernel shrinker gets invoked, iterates through the registered
shrinkers, and invokes the callbacks that the IOMMU subsystem registered. Our
callback makes a hypervisor call (HVC) to pKVM to free page tables. The
underlying HVC implementation will walk the page tables of elligible domains
and free empty page tables, taking advantage of the descriptor counts during
the PTW.

```text
Memory pressure
      |
      v
Host kernel MM shrinker
      |
      v
IOMMU shrinker callback
      |
      v
HVC to pKVM (EL2)
      |
      v
Walk eligible domains -> valid_count == 0? -> free empty page tables
```

## Takeaways
I really enjoyed my time at Google. The technical work was very interesting and
exactly what I was looking while working on the kernel. Initially, I was
worried about the limited scope and impact of only working on device drivers
but I was pleasantly surprised by the complexity and the importance of the
IOMMU subsystem. IOMMU is a fairly unique domain in that it is involed in
almost every step of the way, from boot to whenever you use the WiFi, display,
camera, touchscreen, AI inference, etc. I also really enjoyed working at
different abstraction levels. Some parts of the work involved working very
close to the hardware, directly influencing behavior with command queues and
MMIO. Surprisingly, understanding the intricate hardware details by reading the
ARM ARM and TRMs was rewardding and also interesting.

I also enjoyed working on the PCP Chassis team. It goes without saying that
everyone on the team was technically advanced and had lots of domain expertise.
The team was also very collaborative, both within the team and with other
external teams (think hardware, client drivers). I also thought the team did a
good job of striking a balance between year-to-year maintainance tasks and new
novel projects. Because Pixel's release every year, there is a lot of effor
that needs to go into porting codebases to newer kernel versions and
requirements. There is also the addition of new hardware components that needs
some support from the Chassis team. This can be challenging and time consuming,
especially for domains like the IOMMU where the ARM SMMU upstream driver has a
good amount of churn. Despite this, the team also took on more novel tasks such
as Rust Linux and system level design changes, telemetry being one of them.

One area that I want to explore individually is deterministic simulation
testing for the kernel. DST has been applied to userspace programs (think
Antithesis and TigerBeetle) but I have yet to see an application to kernel and
hypervisor code. Throughout the internship, one challenge I faced was reliably
testing the code I wrote. We didn't really have a unit testing framework and
even if we did, some of the code I wrote such as writing to MMIO regions or
writing a command into memory would be hard to unit test. I think DST and smart
mocking of every external dependency can help solve this challenge. This would
not have been possible with C but with the rise of Rust Linux, I believe there
is an opportunity to make **design decisions based on testablity** in the
kernel.

The only conflict I have after the internship (and something I had going into
it as well) is balancing my interests in working on kernel development but also
low level userspace programs. Naturally, there are very few jobs that require
both. Ultimately, I want to work closely with the hardware, which both kernel
development and low level userspace development provide in different ways. The
reason I am attracted to languages other than C such as C++ and Rust is because
it seems like they provide more opportunities for software design. Often times,
when working in the kernel, there aren't as many opportunities for software
design, whether that be because of limitations of C or because you are adding
to upstream drivers that already have some design baked into it. I also find
the practice of studying a programming language very deeply and understanding
its intricacies very interesting. There is naturally less of this with more
primitive languages like C. This is something I am actively trying to figure
out...
