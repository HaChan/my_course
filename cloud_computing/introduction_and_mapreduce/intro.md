#What is a Cloud

Cloud = Lots of storage resources compute cycles located nearby.

Because the data is enormous so the computing need to be moved nearby the data

There are two kind of cloud:

  1. A single-site cloud or a geodistributed cloud (aka "Datacenter") consists of:

    - Compute nodes (servers, grouped in to racks)

    - A Rack is a unit of several servers which typically share the same power and often share a top-of-the-rack switch.

    - The switches is often connected via a network topology (e.g tree-like, hierarchical...)

    - Backend nodes used for storage that connected to the network.

    - User facing server for handling client requests or submitting jobs

    - Software services: operating system, user-level application, IP protocol...

  2. Multiple geographically distributed clouds consists of:

    - Multiple sites

    - Each sites might have different different structure and services

#New feature in Today's Clouds

1. Massive scale

Datacenters are very large, they may contain hundreds of thousands servers.

2. On-demand access

Pay for what you use, no upfront commitment. Anyone can access it.

Like renting a cab (taxi) vs renting a car.

Today services allow you to pay a few cents - dollars to a few per CPU hour.

*aaS classification:

  - HaaS: Hardware as a Service:

    + Get access to a bare bones hard ware machines and do whatever you want.  E.g: private clouds...

    + Security risks: giving root access of the hardware to user

  - IaaS: Infrastructure as a Service:

    + Allow users access to machines (computing, storage, infrastructure)and install their own Operating System. Virtualization is one way of archiving this (or using linux).

    + Ex: Amazon Web Services (EC2, S3), Microsoft Azure.

  - Paas: Platform as a service:

    + A more tightly congealed, consolidated service form of Iaas. User don't access to VM but user's code and programs is tightly intergrated with the software platform

    + Ex: Google's AppEngine (for Java, Python and Go).

  -Saas: Software as a Service:

    + Access software services when needed. Ex: Google docs...

3. Data-intensive Nature (large data every day)

  - Computation-Intensive Computing:

    + MPI (Message passing interface), High-performance computing (where you have a small amount of data but run fairly intensive computation on it. Ex: weather modeling)

  - Data-Intensive:

    + Large amount of data stored in datacenters.

    + Use compute nodes that are nearby.

    + Compute nodes run computation services to process data.

  - The shift in computation intensive computing toward data intensive computing is a shift away from computation to the data: CPU utilization no longer the most important resource metric, instead I/O is (disk and/or network)

4. New cloud programming paradigms

  - Easy to write and run highly parallel programs in new cloud programming paradigms

MapReduce/Hadoop, NoSQL...


A problem is consider a cloud computing problem when it has all of these four aspect. If none of these aspect is present, it might not be a cloud computing problem, it might be an already solved problem which has well known solutions. It the problem has one of these aspect, it might be a new problem that can be solve with cloud computing.
