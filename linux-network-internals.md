Linux Network Internals

* Understanding Linux Network Internals
* Christian Benvenuti
* 2005


OSI model refresher:

* L1 physical layer. (Ethernet physical layer: 10BASE-T, 100BASE-TX, 1000BASE-T)
* L2 data link layer. (Ethernet data link layer: IEEE 802 MAC)
* L3 network layer. (IPv4: RFC 791)
* L4 transport layer. (TCP: RFC 793)
* L5 session layer.
* L6 presentation layer.
* L7 application layer. (HTTP/1.1: RFC 2616)

A few terms (not necessarily used this way in the text):

* **Frame**: Protocol data unit on a layer that has the means to determine the unit's boundaries, e.g. ethernet data-link layer, which has Preamble and SFD (start frame delimiter) that precede the frame.
* **Packet**: Protocol data unit that relies on a lower layer to determine its boundaries. Term also signifies that connection is reliable. May fit into one frame or be broken into fragments. E.g. TCP.
* **Datagram**: Same thing as packet, but connection is unreliable. E.g. UDP.


1.2.5. When a packet is ready for transmission on the networking hardware, it's handed to the `hard_start_xmit` function pointer of the `net_device` data structure.

1.2.11. Kernel converts between network and host byte order (ex. protocol defines endian-ness, IP & TCP are Big Endian, but Intel processors are Little Endian)

1.2.14. Kernel time is measured in ticks. Tick is the amount of time between consecutive expirations of the timer interrupt. Timer interrupt increments global var `jiffies`.

2.0. Critical data structures

  1. **IMPORTANT** `struct sk_buff`
      * `include/linux/skbuff.h`
      * where a packet is stored
      * same struct used by all network layers
        * FAT interface: struct definition is monstrous to support all types of packets on all layers
        * peppered with preprocessing `#ifdef` directives to handle linux configuration options
      * each `sk_buff` is owned by a `sock` struct
      * each `sk_buff` has a `net_device`, which is updated by the device driver when packet is received
  2. **IMPORTANT** `struct net_device`
      * `include/linux/netdevice.h`
      * each network device is represented by this data structure
      * Ex names for Ethernet devices `eth0`, `eth1`

2.1.5.3. `sk_buff` keeps pointers to head and data. Each network layer inserts its headers into free space in the reserved (fixed size) head section of `sk_buff`.

2.2.2. Each net device has an MTU (maximum transmission unit), which is the max size of frames device can handle

* Ethernet MTU == 1500 (payload), 1514 (payload + header), 1518 (payload + header + frame check sequence). Numbers in octets (== bytes).

2.2.4. `net_device->xmit_lock_owner`

  * Serializes accesses to the driver function `net_device->hard_start_xmit`
   * **IMPORTANT** this means that each CPU can carry out only ONE transmission at a time on any given network device
     * **QUESTION** though I assume the duration of the lock is per frame(s), and that the CPU can interleave transmission of multiple abstract "connections" (TCP) simultaneously
  * `xmit_lock_owner` is ID of the CPU that holds the lock on the net device
  * Note that lockless device mode is available when the driver supports it

2.2.6. `net_device->ingress_lock`

* Traffic Control subsystem defines a similar `egress_lock` for each net device

3.1. Three special interfaces between kernel and user space

1. procfs (`/proc` virtual filesystem)
  * files don't actually exist on disk (virtual)
  * read-only access
2. sysctl (`/proc/sys` directory)
  * read/write kernel values
  * kernel controls which values can be modified
3. sysfs (`/sys` filesystem)
  * newer version of procfs/sysctl

3.3. syscall `socket(family, type, protocol)` returns a sockfd, invokes `sock_ioctl` under the hood

3.4. Netlink sockets

  * RFC 3549
  * Preferred interface between kernel and user space for IP networking configuration
  * `int socket(int domain, int type, int protocol)`
  * params for TCP/IP sockets: `socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)`
  * 0 in the protocol field means default for domain (address family) and type, so you can just write `socket(AF_INET, SOCK_STREAM, 0)`

4.0. Kernel's subsystems are heavily inter-dependent, so events are passed along on notifications chains

* pub/sub

5.2. Hardware initialization

* Done by device driver in cooperation with bus (e.g. PCI)
* Driver configures IRQ and I/O address so they can interact with kernel

5.3. NIC initialization

* NIC == Network Interface Card
* NIC needs an IRQ line to call kernel's attention
 * Virtual devices, like loopback, don't require an IRQ line
* I/O ports and memory registration
  * Driver maps an area of device's memory onto system memory
     * Kernel has a pointer to device's memory that it can read from/write to directly (via bus) 

5.4. Devices interact with kernel in two ways

* Polling
  * Driven by kernel.
* Interrupts
  * Driven by driver. Hardware signal.
  * Types:
     * Reception of frame
     * Transmission failure
     * Device has enough memory for a new transmission
         * Common for NIC to stop accepting onto egress queue when out of space, and re-enable it when memory becomes available

5.4.1 Hardware interrupts

* When a device driver registers an NIC, it requests and assigns an IRQ line. It then registers a handler function pointer for a given IRQ.
* Calls`request_irq` in `include/linux/interrupt.h`
* Ex handler: `el3_start_xmit(struct sk_buff *skb, struct net_device *dev)` in `drivers/net/3c509.c`
  * Sets `el3_start_xmit` as its `net_device->hard_start_xmit` function pointer.

5.7. Per-CPU ingress queues are initialized at boot time by `net_dev_init`

* `net/core/dev.c`
* Run before NIC device drivers register themselves
* Random component is added to delay of timers to try to keep them out of phase so that they don't all interrupt the CPU at the same times
* NICs (may) contribute to system entropy -- `SA_SAMPLE_NET_RANDOM`

5.8. User-space helper `/sbin/hotplug` (Plug and Play == PnP)

* script that's invoked whenever kernel detects insertion or removal of a hot-pluggable device

8.6. Organization of `net_device` structures

* `dev_base` global list of all `net_device` instances
* `dev_name_head` hash table indexed on device name
* `dev_index_head` hash table indexed on device ID
* lookup routines are in `net/core/dev.c`

8.13. `ethtool` and `ifconfig` allow user-space queries on net devices

* `net_device` initialized with useful function pointers, such as `ethtool_ops` for ethernet devices
* Ex `ethtool` -> `ioctl` (user space) -> `dev_ioctl` (kernel space) -> `dev_ethtool` -> `ethtool_ops` (driver)

Part III. Generally "transmission" means communication in any direction. But in kernel discussion, "transmission" only means sending frames outward, and "reception" refers to frames coming in. Also, "egress" == "transmission" and "ingress" == "reception".

9.2.2. Interrupt per frame doesn't perform well under high traffic loads.

* **receive-livelock**: condition where
  * CPU spends all its time handling interrupts, which preempt processing
  * NIC runs out of queue space because CPU is not processing frames out
  * New frames cannot be queued because there is no space (dropped)
  * Old frames cannot be processed because there is no CPU time available for them
* Solution is to process multiple frames per interrupt, which is what most NIC drivers do.

9.2.3. Linux NAPI ("New API")

* Hybrid of interrupts and polling.
* Driver disables interrupts on devices with frames in their ingress queue and delegates polling to kernel.

9.2.4. Timer interrupts

* Based on device hardware timer, not kernel timer.
* Wasteful to CPU under low load.
* Alleviates interrupts on CPU under high load.

9.2.5. Tulip driver

* `drivers/net/tulip/interrupt.c`
* Use frame-driven interrupts under low load and timer-driven interrupts under high load

9.3. **IMPORTANT** Interrupt handlers

* CPU cannot be interrupted while it's handling an interrupt.
  * Interrupt handlers are run in "interrupt context": non-preemptable and non-reentrant.
* Handlers are split into top-half and bottom-half handlers. Hardware interrupts stop the world, so we want to minimize the work we have to do until we can start the world again (top-half).
  * **Top-half**: Do only the critical stuff that must be done in a timely fashion to prevent data loss. Allocate an `sk_buff`, copy data from device into it, initialize some protocol params, enqueue the packet for later processing (`netif_rx` queue).
  * **Bottom-half**: Do the processing you saved for later, i.e. everything above the L2 (data link) layer. Scheduled separately from the top-half (if you consider the top-half "scheduled"). These handlers *are* preemptable (though as we will see, the way they are implemented is non-reentrant).
     * **QUESTION** Does the bottom half entail writing to the socket buffer after running all protocol handlers (IP, TCP)? Or does "writing to socket buffer" really just mean marking this `sk_buff` as ready for read? Recall that each `sk_buff` points to its owning `sock`.

9.3.5.2. **IMPORTANT** Software interrupts (softirqs)

* Adds some concurrency to the bottom-half
  * Any number of *different type* softirq instances can run concurrently.
     * **QUESTION** What exactly does "type" mean here?
     * **ANSWER** See 9.3.12 below
  * But still, only one of a *given type* of a softirq instance may run at the same time per CPU.
     * **Why allowed to run one concurrently on every CPU?** I think it's because socket affinity is to the CPU on which is was created... socket state is only "global" to its owning process and cannot leak across CPUs.
     * **Why restricted to run one concurrently on a given CPU?** I think it's to guarantee that writing to socket buffer is thread-safe. On a given CPU, softirqs of same type running concurrently could be working on multiple `sk_buff` instances for the same socket (if such concurrency was allowed). Again assuming an affinity between CPU and sockets.
         * **QUESTION** I don't think my understanding is complete here, because listening sockets can be shared across CPUs via forking... which throws a wrench in the affinity idea. My assumption here is that fork() does not enforce CPU affinity between parent and child. I doubt linux enforces such fork-CPU affinity, otherwise Gunicorn's suggestion to config (2*num_cores + 1) workers is bunk; they'd all use the same core when they fork from the master process.
         * **TENTATIVE ANSWER** I think the confusion is over the level of abstraction we use when we say "socket". I assume there is an initial phase where an incoming `sk_buff` with no known TCP connection is assigned an accepting process (via new accepting socket FD). This would allow processes to share listening sockets. The socket FD is really just an address-binding mechanism for applications in user space.
      * **FORKING DETOUR** When a process is forked, everything is copied, including its file descriptor table (`/proc/PID/fd`). Kernel keeps a global ref count for each FD: `open()` increments and `close()` decrements. Kernel destroys FD when ref count is zero.
      * **SOCKETS DETOUR** The accepting socket is always a different socket than the listening socket. Kernel takes care of the internal accounting before returning from the `accept()` syscall with a new socket FD.
  * Also, kernel does not allow *interleaving* of the same type of softirq (per CPU). That is, if a softirq of a given type is currently suspended, no other softirqs of that type can run until it's finished. This is purely to simplify locking.
     * **Why?** I think it's to guarantee that ordered bytestreams (TCP) don't become un-ordered on the way from NIC to user space. If interleaving was allowed, would have to do a lot of extra checks, e.g. that `sk_buff` of this softirq doesn't have a later TCP sequence number than the `sk_buff` in the suspended softirq.

9.3.10. Background kernel threads check for and complete unexecuted softirqs. There is one such `ksoftirq` thread per CPU.

9.3.12. Types of softirqs:

1. `NET_TX_SOFTIRQ` handles outgoing traffic (transmits to NICs)
2. `NET_RX_SOFTIRQ` handles incoming traffic (receives from NICs)

9.4. Each CPU has its own queue for incoming frames.

* `softnet_data` defined in `include/linux/netdevice.h`
* ingress frames are queued to `softnet_data->input_pkt_queue`
* The per-CPU `softnet_data` structs are initialized in `net_dev_init` which runs at boot time (see 5.7)

10.4.1. NAPI disables interrupts for a device until its queue is empty, at which time the kernel stops polling and re-enables the interrupts.

* Devices that implement NAPI must provide a `poll` function, which is added to their `net_device` struct on initialization.
* If device isn't using NAPI, then `net_dev_init` sets it to `process_backlog` during initialization.

10.7.2. `net_receive_skb` is the helper function used by `poll` to process ingress frames.

* Passes a copy of the frame to the L3 (IP) protocol handler.
* Assuming L3 protocol handler determines that the packet is in fact destined for this machine's network address, then proceed to 24.0 (delivery to L4 and up).

11.0. Frame transmission

* `dev_queue_xmit` plays the role for egress that `netif_rx` plays for ingress.
  * dequeues a frame from the kernel's egress queue and feeds it to the device's `hard_start_xmit` function.
* one `output_queue` per CPU
  * used by both NAPI and non-NAPI devices
* requeue capability:
  * transmission failure
  * another CPU holds lock on driver
* `hard_start_xmit` returns:
  * `NETDEV_TX_OK` if transmission successful
  * `NETDEV_TX_BUSY` if device lacks sufficient space in transmit buffer


SKIP AHEAD TO 24.0: Layer Four Routing (skipped pages 305 - 673)


24.3.2. Delivering *raw* input datagrams to the recipient application

* This section only concerns SOCK\_RAW sockets (not SOCK\_STREAM sockets, e.g. TCP).
* Every time the kernel receives a packet that carries an L4 protocol not handled by the kernel, all the sockets that registered for that protocol receive a copy of the packet. It is up to them to accept or discard the packet. This means that the applications must have a way to understand if the packet they receive is addressed to them, a task rendered unnecessary in TCP and UDP by the port system.


Misc

* Size of the kernel TCP receive buffer: `/proc/sys/net/ipv4/tcp_rmem`