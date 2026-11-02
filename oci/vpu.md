# OKE Block Volume Performance Tiers
## Teleprompter Script for Trainees

**Slide 8: Block volume performance tiers**
**Total reading time: about 15 to 18 minutes**

> **How to use this script**
> - Lines in **[brackets]** are cues for you. Do not read them aloud.
> - **[PAUSE]** means stop for a breath, or let the room think.
> - **[ASK]** means ask the trainees the question and wait for answers.
> - **[SLIDE]** means point at the slide.
> - Prices are USD list prices from Oracle, checked on 2 November 2026. Tell trainees to confirm in Oracle's cost estimator.

---

## PART 1: OPENING

Good morning, everyone.

Today we are going to talk about disks.

Specifically, the disks behind your applications on Kubernetes.

[PAUSE]

Here is the question for today:

**How do you choose how fast, and how expensive, your storage should be?**

By the end of this session, you will be able to do three things.

One. Explain what a VPU is.

Two. Set the disk speed in a StorageClass.

Three. Estimate what that disk will cost per month.

[PAUSE]

Let's start with why we need this at all.

---

## PART 2: WHY DO WE NEED A DISK?

When you run an app on Kubernetes, your containers are temporary.

They can stop. They can restart. They can move to another machine.

When a container goes away, everything inside it goes away too.

[PAUSE]

Now imagine that container is a database.

You do not want your data to disappear.

So we give the app a **disk that lives outside the container**.

On Oracle Cloud, that disk is called a **Block Volume**.

Think of it like a USB drive that you plug into a computer.

The computer can change. The USB drive keeps your data.

[PAUSE]

Here is the next idea.

Not every app needs the same disk speed.

A backup job can use a slow, cheap disk.

A busy database needs a fast one.

This slide is about how you **choose** that speed.

[SLIDE: point to the title, "Block volume performance tiers"]

---

## PART 3: KEY TERMS

Before we go further, let me give you the words we will use.

I will go slowly.

**OKE.**
That is Oracle Kubernetes Engine.
It is Oracle's managed Kubernetes service.

**Pod.**
That is your running application.

**Block Volume.**
That is the virtual hard disk in Oracle Cloud.

**PVC.**
It stands for PersistentVolumeClaim.
It is the pod's request. It says: "I need a 100 gigabyte disk."

**StorageClass.**
This is a template.
It says what kind of disk to create.

**CSI driver.**
This is the helper that talks to Oracle Cloud and actually creates the disk.

**VPU.**
This is the speed dial for the disk.
We will spend the next section on it.

**IOPS.**
This means reads and writes per second.
More IOPS means a faster disk.

[PAUSE]

[ASK: "Can anyone tell me, in one sentence, what a StorageClass is?"]

[Expected answer: a template that says what kind of disk to create.]

---

## PART 4: WHAT IS A VPU?

VPU stands for **Volume Performance Unit**.

[PAUSE]

Now, an important warning.

**A VPU is not a vCPU.**

It has nothing to do with processors.

It has nothing to do with the VPC network either.

A VPU only measures how fast the **disk** is.

[PAUSE]

Here is how it works.

More VPUs means a faster disk.

More VPUs also means a higher price.

[PAUSE]

We write the setting as **vpusPerGB**.

That means VPUs for every gigabyte of disk.

The formula is simple:

**Total VPUs equals the disk size in gigabytes, times vpusPerGB.**

[PAUSE]

Example.

A 100 gigabyte disk.
vpusPerGB is 10.
That gives 1,000 VPUs.

---

## PART 5: THE FOUR TIERS

[SLIDE: point to the four coloured boxes]

Look at the four boxes on the slide.

They get taller from left to right.

Taller means faster.

It also means more expensive.

[PAUSE]

**First box. Zero.**
The tier is called **Lower Cost**.
Use it for backups, archives, and data you rarely touch.

**Second box. Ten.**
The tier is called **Balanced**.
This is the usual choice for most general applications.

**Third box. Twenty.**
The tier is called **Higher Performance**.
Use it for busy databases and apps that read and write a lot.

**Fourth box. Thirty to one hundred twenty.**
The tier is called **Ultra High Performance**.
This is for the most demanding, latency-sensitive workloads.

[PAUSE]

Notice the small label under the boxes.

It says "vpusPerGB value."

That tells you the numbers are the setting you put in the StorageClass.

---

## PART 6: WHO GETS THE VPUs?

This is the part that confuses people, so listen carefully.

[PAUSE]

**The block volume, the disk, gets the VPUs.**

Not the pod.

Not the node.

Not the user.

[PAUSE]

Let me explain each one.

The **StorageClass** only holds a setting. It is a template.

The **block volume** is the real disk. It gets the VPUs.

The **pod** just uses the disk. It benefits from the speed.

[PAUSE]

And where is this setting stored?

**On the storage side.**

It lives in Oracle's Block Volume service.

You can see it in the Oracle Cloud Console.

Go to Block Storage, then Block Volumes, then open the volume.

[PAUSE]

[ASK: "If I change the StorageClass today, what happens to disks that already exist?"]

[Answer: nothing. Existing disks keep their performance. Only new disks use the new setting.]

[PAUSE]

To change an existing disk, you change it on the volume itself in Oracle Cloud.

You can do that in the Console, the API, or the command line.

---

## PART 7: HOW IT WORKS, STEP BY STEP

[SLIDE: point to the box that says "Where it is set"]

The box on the left says:

**vpusPerGB goes under parameters in the StorageClass.**

Here is the whole journey.

[PAUSE]

Step one.
You write a StorageClass with the speed you want.

Step two.
A pod's PVC asks for a disk, and names that StorageClass.

Step three.
The CSI driver sees the request.
It asks Oracle Cloud to create a block volume with that vpusPerGB.

Step four.
Oracle Cloud creates the disk, with those VPUs, on the storage side.

Step five.
The disk is attached to the node running your pod.
Then it is mounted into the pod.

Step six.
Your app reads and writes at the speed that VPU level allows.

[PAUSE]

Let me show you what it looks like.

### Example: the StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: oci-bv-high-perf
provisioner: blockvolume.csi.oraclecloud.com
parameters:
  vpusPerGB: "20"
```

Read it out loud with me.

The name is `oci-bv-high-perf`.
The provisioner is the Oracle block volume CSI driver.
And under parameters, `vpusPerGB` is `"20"`.

**One thing to notice.** The twenty is in quotes. It is a string.

### Example: the PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-data
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: oci-bv-high-perf
  resources:
    requests:
      storage: 100Gi
```

This PVC asks for 100 gigabytes.

It names our StorageClass.

The result is a 100 gigabyte disk with 2,000 VPUs.

That is faster than the default.

[PAUSE]

---

## PART 8: THE ULTRA HIGH PERFORMANCE WARNING

[SLIDE: point to the box on the right]

The right-hand box has a warning.

For the Ultra High Performance tier, there are extra requirements.

[PAUSE]

First, the worker node needs a **supported shape**.

Second, the disk must be attached with **multipath**.

Multipath means more than one network path between the node and the disk.

It gives more speed and more reliability.

[PAUSE]

And finally, the slide says:

**Verify limits in the current Oracle docs.**

Why? Because limits can change.

Do not trust a number someone wrote down last year.

Always check the current documentation.

[PAUSE]

If these requirements are not met, the volume may not reach ultra performance.

You would pay for speed you do not get.

---

## PART 9: WHAT DO YOU ACTUALLY GET?

Let me give you real numbers.

Above ten VPUs, speed grows in a steady pattern.

Here is what each tier gives.

**At ten VPUs.**
Sixty IOPS per gigabyte.
Up to twenty-five thousand IOPS for a whole volume.

**At twenty VPUs.**
Seventy-five IOPS per gigabyte.
Up to fifty thousand IOPS for a whole volume.

**At thirty VPUs.**
Ninety IOPS per gigabyte.
Up to seventy-five thousand IOPS for a whole volume.

**At one hundred twenty VPUs.**
Two hundred twenty-five IOPS per gigabyte.
Up to three hundred thousand IOPS for a whole volume.

[PAUSE]

And the zero tier?

It gives only about **2 IOPS per gigabyte**, capped at 3,000.

That is why it only suits cold data.

[PAUSE]

One more thing.

The node also limits speed.

A very fast disk on a weak node will not reach its full potential.

The node shape and its network bandwidth both put a cap on it.

---

## PART 10: COSTING

Now the part everyone cares about.

**What does this cost?**

[PAUSE]

Block volume cost has **two parts**. They add together.

Part one is **storage**. You pay for the size of the disk.

Part two is **performance**. You pay for the VPUs.

[PAUSE]

Here is the formula.

**Monthly cost equals the size in gigabytes, times the storage rate, plus the VPU rate times vpusPerGB.**

Oracle's list prices are:

Storage: about **two point five five cents** per gigabyte per month.

Performance: about **one point seven cents** for every ten VPUs, per gigabyte per month.

[PAUSE]

Now let me turn that into a price for each tier.

I will say the price **per gigabyte per month**.

**Zero VPUs, Lower Cost.**
Two point five five cents. That is storage only.

**Ten VPUs, Balanced.**
Four point two five cents.

**Twenty VPUs, Higher Performance.**
Five point nine five cents.

**Thirty VPUs, Ultra High Performance.**
Seven point six five cents.

**One hundred twenty VPUs, the maximum.**
Twenty-two point nine five cents.

[PAUSE]

Those cents are small, so let me make them real.

### A 100 gigabyte disk, per month

Lower Cost: **two dollars fifty-five.**
Balanced: **four dollars twenty-five.**
Higher Performance: **five dollars ninety-five.**
Ultra at thirty: **seven dollars sixty-five.**
Ultra at one hundred twenty: **twenty-two dollars ninety-five.**

### A 1 terabyte disk, per month

Lower Cost: **twenty-five dollars fifty.**
Balanced: **forty-two dollars fifty.**
Higher Performance: **fifty-nine dollars fifty.**
Ultra at thirty: **seventy-six dollars fifty.**
Ultra at one hundred twenty: **two hundred twenty-nine dollars fifty.**

[PAUSE]

Look at the last one.

The maximum tier costs about **five times** the Balanced price.

So do not pick the fastest tier by default.

[PAUSE]

### A quick example

A database needs 500 gigabytes of storage.

At twenty VPUs, that costs about **twenty-nine dollars seventy-five** a month.

At ten VPUs, it costs about **twenty-one dollars twenty-five** a month.

If Balanced already meets the database's needs, you save about eight dollars fifty a month.

For one volume, that is small.

For fifty volumes, it adds up.

---

## PART 11: THINGS THAT AFFECT YOUR BILL

Here are five things to remember.

**One. You pay for the size you provision, not the space you use.**
A 500 gigabyte disk holding 50 gigabytes of data is billed for 500.
So ask for only what you need.

**Two. Billing is prorated.**
Oracle tracks usage per second, after a one-minute minimum.
A disk deleted after two days costs about two thirtieths of a month.

**Three. Disks keep costing until they are deleted.**
If your StorageClass uses `reclaimPolicy: Retain`, the disk stays after you delete the PVC.
And you keep paying.
With `Delete`, removing the PVC removes the disk.

**Four. Other costs are separate.**
Backups, cross-region copies, boot volumes, and the node's compute are billed on their own.

**Five. Prices vary.**
These are US dollar list prices.
Your currency and contract may be different.
Use Oracle's cost estimator for a real quote.

---

## PART 12: HOW TO SAVE MONEY

Five habits.

**One. Pick the lowest tier that works.**
Zero for backups and logs.
Ten for most apps.
Twenty or higher only for heavy databases.

**Two. Keep several StorageClasses.**
For example: `cheap`, `balanced`, and `fast`.
Each app picks the one it needs.

**Three. Tune the volume itself.**
You can change VPUs on an existing disk in Oracle Cloud.

**Four. Consider auto-tuning.**
Auto-tuning lets a volume move between a low and a high VPU level.
It scales up when needed, and back down when not.
You set the range, for example ten to sixty.

**Five. Clean up.**
Delete PVCs and volumes you no longer use.
Leftover test disks are a classic source of surprise charges.

---

## PART 13: COMMON QUESTIONS

[Use these if trainees ask, or ask them yourself.]

**Can one StorageClass serve everything?**
Yes. But then every disk gets the same performance.
Most teams make a few classes and let each app choose.

**Does changing a StorageClass fix existing disks?**
No. It only affects new volumes.
Change old ones on the volume in Oracle Cloud.

**Is faster always better?**
No. Faster costs more.
Choose the lowest tier that meets your app's needs.

**Is a VPU the same as a vCPU?**
No. A VPU is disk speed. A vCPU is processor power.

**What else can limit disk speed?**
The node shape and its network bandwidth.

---

## PART 14: RECAP

Let me finish with the main points.

[PAUSE]

One.
A **VPU** is a disk speed setting, not a processor.

Two.
The setting is **vpusPerGB**.
You put it in the **StorageClass**, under parameters.

Three.
The **block volume**, the disk, gets the VPUs.
It is stored on the **Oracle storage side**.
Pods just use the speed.

Four.
The tiers are **zero, ten, twenty, and thirty to one hundred twenty**.
Lower Cost, Balanced, Higher Performance, and Ultra High Performance.

Five.
**Ultra High Performance** needs a supported shape and multipath.
Always check the current Oracle docs.

Six.
Cost is **size times the storage rate, plus VPUs**.
Balanced is about four point two five cents per gigabyte per month.
The maximum tier is about five times that.

Seven.
**Pick the lowest tier that works.**

[PAUSE]

If you remember only one sentence, remember this one:

> **You choose disk speed by setting vpusPerGB in a StorageClass. OKE uses it to create the block volume, and the volume gets the VPUs. Pods just use the speed.**

[PAUSE]

---

## PART 15: CLOSING

Thank you, everyone.

Before we move on, are there any questions?

[PAUSE]

[Suggested exercise if time allows:]

> **Exercise.** A team needs three disks.
> One is a 200 GB backup store.
> One is a 300 GB web app disk.
> One is a 500 GB busy database disk.
> Which tier would you choose for each?
> What is the total monthly cost?
>
> **Suggested answer:**
> Backup, 200 GB at tier 0: 200 × $0.0255 = **$5.10**
> Web app, 300 GB at tier 10: 300 × $0.0425 = **$12.75**
> Database, 500 GB at tier 20: 500 × $0.0595 = **$29.75**
> **Total: $47.60 per month**

---

## SOURCES (for the presenter)

- Oracle Block Volume pricing summary: https://us.fitgap.com/products/oracle-cloud-infrastructure-block-volumes
- Oracle Cloud Infrastructure blog on auto-tuning: https://blogs.oracle.com/cloud-infrastructure/?p=12020
- Oracle docs, Block Volume Performance: https://docs.oracle.com/iaas/Content/Block/Concepts/blockvolumeperformance.htm
- Oracle OCI Storage Pricing guide (2026): https://oraclelicensingexperts.com/blog/oracle-oci-storage-pricing/

*Prices and limits change. Verify against the current Oracle price list and documentation before the session.*
