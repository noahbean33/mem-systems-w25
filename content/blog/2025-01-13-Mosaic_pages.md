+++
title = "Mosiac Pages: Big TLB Reach with Small Pages"
[extra]
bio = """
  [Sami Aljabery] is a graduate student studying Electrical and Computer Engineering at Oregon State University. He's passionate about computer architecture, loves exploring, and enjoys the Metroid series.
"""
[[extra.authors]]
name = "Sami Aljabery"
[[extra.authors]]
name = "Gabriel Rodgers (Scribe) "
[[extra.authors]]
name = "Noah Bean (Blogger) "
+++


## Background and Context
The Bottleneck Problem - Modern workloads like machine learning have massive working sets of memory that often exceed the TLB’s capacity which leads to frequent misses and performance degradation. Previous approaches rely on physical memory contiguity which increases overhead from memory defragmentation.

## KEYWORDS 
virtual memory - an abstraction of memory which provides the illusion of contiguous physical memory. 

address translation - the process of converting virtual addresses into physical addresses by using page tables and TLBs

Translation Look-aside Buffer (TLB) - special memory cache that holds translations between virtual and physical memory, relatively small compared to caches because it is designed to be faster

paging - physical memory is divided into frames and virtual memory is divided into pages and the operating system maps pages to frames in the page table

Hashing - A computational technique that maps input data to a fixed range of output values using a hash function.

Iceberg hashing - An advanced hashing technique used in Mosaic Pages to optimize memory allocation. It features low associativity, meaning each virtual page has limited potential physical frame mappings, and stability, ensuring consistent mappings.

Radix tree - A tree-like data structure used to store key-value pairs compactly, where each node represents a prefix of the keys.

Arity - The number of arguments or inputs a function takes.
Address Space Identifier (ASID) - A unique identifier assigned to each process’s TLB mappings. ASIDs prevent TLB entries from one process being accessed by another during a context switch.

Detailed summary of the main contributions of the paper

## Summary of 1. Introduction:
The paper introduces Mosaic Pages, which is a technique to increase the reach of Translation Lookaside Buffers (TLBs) without relying on physical contiguity, thus eliminating the need for memory defragmentation. Mosaic Pages leverages virtual contiguity and Iceberg hashing to compress multiple address translations into a single TLB entry. Some highlights from the paper include: 
Mosaic Pages compresses translations by mapping virtual pages to a limited number of physical page frames using hashing which reduces the size of each translation so that multiple mappings can fit into one TLB entry.
A full-system prototype implemented in gem5 demonstrated up to 98% reduction in TLB misses compared to a baseline model. 
A Verilog implementation of the technique indicates minimal impact on clock speed or area.
Mosaic Pages swaps fewer pages than Linux while maintaining similar performance when under memory pressure.

## Summary of 2. Mosaic Pages:
The section discusses the design of Mosaic Pages. A Mosaic Page comprises a virtually consecutive 4 KiB base pages where a is between 4 and up to 64. Translations for all a base pages are compressed into one TLB entry using Compressed Physical Frame Numbers (CPFNs). These CPFNs mean that virtual pages are mapped to a small set of physical frames (h = 104 in experiments) using hashing, reducing the number of bits required for translation. Physical memory is structured as a hash table with buckets containing slots. Each virtual page is hashed to specific buckets, and its placement is recorded as a CPFN. The Mosaic Pages technique uses Iceberg Hashing, which ensures low associativity, stability, and high utilization of memory. Physical memory is divided into buckets with "front yard" and "back yard" sections for load balancing. Pages are allocated to the least-filled bucket, minimizing conflicts while maintaining near-full memory utilization. Mosaic Pages use a new Horizon LRU algorithm for managing swapping when no slots are available. 

## Summary of 3. Implementation of Mosaic Pages:
This section describes the implementation of Mosaic Pages using the gem5 simulator and a Linux prototype. The components include a Gem5 Simulation of the Mosaic TLB and a Linux Prototype that uses the reserved memory and LRU replacement policy.

## Summary of 4. Evaluation:
This section discusses the performance and feasibility of Mosaic Pages. Using workloads like Graph500 and XSBench, it was found that Mosaic Pages reduces TLB misses across all workloads and achieves up to 98% reduction with a Mosaic-64 configuration. Mosaic experiences associativity conflicts only after 98% memory utilization. Swapping was found to be comparable to base Linux.

## Summary of 5. Related Work:
This section reviews prior research addressing TLB performance challenges. Previously, “huge pages” were used to improve TLB reach but this would lead to overhead due to defragmentation. Mosaic avoids physical contiguity by using low-associative hashing for flexibility and reduced hardware costs. A redesign of TLB entries that combines adjacent virtually and physically contiguous pages into one entry reduces TLB misses. Mosaic uses Iceberg hashing to increase TLB reach and hit rate, while hashed page tables focus on reducing miss costs.


## Important results and what they mean
1. Reduction in TLB Misses Mosaic Pages reduced TLB misses by 6–81% across various workloads compared to a traditional TLB. Using Graph500, Mosaic reduced misses with an improvement of up to 98%.
2. Memory Utilization and Swapping Behavior Mosaic Pages achieved 98% memory utilization before experiencing associativity conflicts, compared to 99% for Linux.
3. Hardware Feasibility Mosaic hardware synthesized on a 28nm CMOS process demonstrated 220 ps latency at 4 GHz.

## Strengths and Weaknesses of the Paper 
Strengths of the Paper
1. Innovative Concept: Mosaic Pages introduces a novel approach to increasing the reach of Translation Lookaside Buffers (TLBs) without relying on physical memory contiguity. By leveraging virtual contiguity and Iceberg Hashing, it compresses multiple address translations into a single TLB entry.

2. Comprehensive Evaluation: The paper provides extensive performance evaluations using a variety of workloads, such as Graph500 and XSBench, and simulations implemented in gem5. 

3. Scalability: Mosaic Pages performs well under high memory utilization, maintaining efficiency up to 98% memory usage without significant degradation. 

4. Low Hardware Overhead: The hardware implementation of Mosaic Pages adds minimal area and latency overhead. The 28nm CMOS implementation demonstrates only 220 ps of latency at 4 GHz, making it feasible for integration into existing systems.


## Weaknesses of the Paper
1. Dependency on Iceberg Hashing Scheme: Mosaic Pages relies heavily on the Iceberg Hashing algorithm to achieve its goals. This dependency limits flexibility and introduces a single point of failure if the hashing algorithm does not perform well under specific conditions or workloads.

2. Worse memory pressure performance compared to Linux: Under high memory pressure, Mosaic Pages performs slightly worse than Linux. While Mosaic Pages avoids memory overhead, its inability to match Linux’s performance under these conditions limits its appeal for memory-constrained environments.

3. Low number of citations might indicate that paper is not sufficiently novel: The paper has a limited number of citations (21 at the time of writing), which could indicate that the concept is either not widely known or lacks sufficient novelty to generate significant interest in the academic community.

4. Page table sharing is not available: Mosaic Pages does not support sharing of page tables across processes. For example, shared libraries, which are common in multi-process applications, cannot be mapped efficiently, leading to higher memory usage.


5. Did not discuss NUMA or GPU implementations: The paper does not address Non-Uniform Memory Access (NUMA) architectures or GPU memory systems, both of which have unique challenges and requirements. NUMA systems require careful handling of memory locality, while GPUs rely on massively parallel processing with their own memory hierarchies.

## Class Discussion: 

## Gaming workloads?
The lack of improvement on the GUPS benchmark, which measures random memory access performance, suggests that Mosaic Pages may face challenges with workloads heavily reliant on randomness. While gaming workloads share some aspects of random access, their diverse memory access patterns, including sequential access for assets and textures, might still allow Mosaic Pages to provide performance benefits in certain scenarios.

## GPUs?
While the paper focuses on CPU workloads, it raises questions about the applicability of Mosaic Pages to GPUs. GPUs, which typically have TLBs as part of their memory management units (MMUs), rely heavily on high-bandwidth and parallel processing architectures. The integration of Mosaic Pages into GPU systems could reduce TLB misses, particularly for workloads with predictable memory access patterns. However, unique architectural challenges in GPUs such as managing parallel threads and high throughput might require significant adaptations of Mosaic Pages to align with GPU memory subsystems. 

## Context Switching in TLB?
What happens to TLB when a context switch occurs? Context switches, an essential part of multitasking in modern operating systems, pose a significant challenge to TLB efficiency and security. During a context switch, the TLB must be managed to prevent unauthorized access to memory pages from the previous process. Typically, this is achieved by entirely invalidating the TLB, which ensures security but introduces performance overhead as new mappings must be repopulated. A more optimal solution involves the use of Address Space Identifiers (ASIDs), which assign a unique identifier to each process. ASIDs allow the TLB to selectively clear only the mappings associated with the previous process, leaving shared mappings intact. This reduces the performance cost of context switches while maintaining strict memory isolation. While the paper does not explicitly discuss context switch management, integrating Mosaic Pages with an ASID-based approach could further enhance TLB efficiency and security.

## How does Iceberg Hashing Work?
Iceberg Hashing is an essential part of Mosaic Pages ability to compress virtual-to-physical address mappings into Compressed Physical Frame Numbers (CPFNs). This advanced hashing technique reduces the number of bits required per mapping, enabling the TLB to store multiple mappings in a single entry. To achieve this, Iceberg Hashing must meet three critical properties: 
1. Range: The hash function should evenly distribute virtual addresses across the available physical frames to ensure efficient memory utilization.
2. Stability: The function must produce consistent, deterministic mappings for the same inputs to maintain system reliability.
3. Reliability: It should minimize collisions and conflicts, even under high memory pressure. 

## Mapping Conflict?
Mapping conflicts can occur in systems using hashed memory allocation like Mosaic Pages. When a new virtual-to-physical mapping conflicts with an existing one, Mosaic Pages resolves the issue by evicting the previous mapping. This eviction follows a heuristic akin to the Least Recently Used (LRU) policy, ensuring that the mapping least likely to be accessed is replaced. Mosaic Pages operates with a fixed associativity. This level of associativity ensures that mappings are evenly distributed across memory, reducing the frequency of conflicts. 

## Front Yard vs Back Yard?
Mosaic Pages introduces a memory organization strategy by dividing physical memory buckets into two distinct regions called the front yard and the backyard. The front yard, which constitutes the larger portion of the bucket, is reserved for frequently accessed data. This prioritization ensures low-latency access to high-demand pages, enhancing performance for workloads that depend on fast memory access. On the other hand, the backyard serves as a storage area for less frequently accessed, 'ghost' pages. 

## Dealing with Conflict?
Mosaic Pages deals with memory allocation conflicts with a fixed associativity of 104, derived from the properties of Iceberg Hashing. This level of associativity ensures a well-distributed mapping of virtual pages to physical frames, reducing the likelihood of conflicts even as memory utilization approaches 98%. In comparison, the standard Linux allocator begins swapping at similar utilization levels but incurs higher memory overhead. Mosaic Pages avoids this overhead by managing conflicts efficiently, relying on the hashing algorithm to spread entries evenly across the available physical frames. Even under high memory pressure, Mosaic Pages maintains performance comparable to Linux.

## NUMA?
Mosaic Pages applicability to NUMA (Non-Uniform Memory Access) architectures remains unexplored. NUMA introduces additional dimensions to the physical address space, as memory is divided across multiple nodes with varying access latencies. Integrating Mosaic Pages into NUMA systems would require enhancements to the Iceberg Hashing algorithm to handle 2D load balancing mappings not only across buckets but also across NUMA nodes. This adaptation would involve node-aware hashing and dynamic monitoring to ensure even distribution of memory accesses and minimize the performance penalties associated with remote access. Exploring this area could unlock further scalability for Mosaic Pages in high-performance and distributed computing environments.

## ISA Adaptation?
Adapting Mosaic TLB to different Instruction Set Architectures (ISAs) would involve primarily hardware-level modifications to the Memory Management Unit (MMU), rather than changes to the ISA itself. For example, implementing Mosaic TLB on x86 would require integrating Iceberg Hashing and support for Compressed Physical Frame Numbers (CPFNs) into the MMU, alongside minor adjustments to the page allocator. This adaptation is feasible, as demonstrated by the Gem5-based prototype. In contrast, RISC-V offers greater flexibility due to its open-source nature, allowing deeper customization of page tables and memory management. Researchers could design a RISC-V processor with a custom MMU tailored for Mosaic Pages, ensuring seamless integration with the ISA.

## Cost?
The cost of implementing Mosaic Pages is described as 'not prohibitively expensive,' primarily due to its efficient use of resources. By compressing multiple mappings into a single TLB entry through Compressed Physical Frame Numbers (CPFNs), Mosaic Pages significantly reduce the memory footprint of TLB entries. This enables increased TLB capacity without requiring additional memory resources. The Verilog implementation further highlights the practicality of Mosaic Pages and suggest that Mosaic Pages can be integrated into existing hardware with modest design changes to the Memory Management Unit (MMU). 

## Origin of Iceberg Hashing?
The authors of Mosaic Pages did not invent Iceberg Hashing but recognized its potential for their TLB optimization technique. Iceberg Hashing excels in three critical areas:
1. Low Associativity: By minimizing collisions, it ensures efficient distribution of virtual-to-physical mappings across available memory frames.
2. Stability: Its deterministic behavior guarantees consistent mappings, an essential feature for reliable memory management.
3. High Utilization: Iceberg Hashing optimizes memory usage, achieving near-full capacity without significant performance penalties.

## Shared Memory Support?
One notable limitation of Mosaic Pages is its lack of support for shared memory. Shared memory allows multiple processes to reference the same data without duplication, reducing the memory footprint and improving efficiency. Without this capability, Mosaic Pages requires each process to maintain its own copy of shared data, such as libraries or frameworks, leading to increased memory usage. The primary reason for this limitation lies in the Iceberg Hashing algorithm, which maps virtual pages to physical frames in a one-to-one manner for each process. Modifying the algorithm to handle shared mappings would require significant changes to both the hashing mechanism and the TLB structure. While the paper acknowledges this shortcoming, it stops short of offering a solution. Addressing this limitation would make Mosaic Pages far more versatile. 

Arbitrary mapping between PA and VA?
Mosaic Pages imposes restrictions on arbitrary physical address (PA) to virtual address (VA) mappings due to the constraints of the Iceberg Hashing algorithm. This deterministic hashing mechanism is designed to optimize memory utilization and reduce TLB misses by compressing multiple virtual-to-physical mappings into a single TLB entry. This could pose challenges in specialized systems that require fine-grained control over address mappings. This trade-off reflects a common theme in academic research of focusing on solving specific challenges which often leads to compromises in other areas.  
 

## Sources:
One-Level Storage System Kilburn et al. IRE Transactions on Electronic Computers, 1962.
OSTEP Textbook Chapters 18-20 Remzi Arpaci-Dusseau
Mosaic Pages: Big TLB Reach with Small Pages
Computer Organization and Design RISC-V Edition The Hardware Software Interface 2nd Edition - December 11, 2020 Authors: David A. Patterson, John L. Hennessy
Computer Architecture, Sixth Edition: A Quantitative Approach December 2017 Authors: John L. Hennessy, David A. Patterson

## Generative AI 
Link to specific Tool: https://chatgpt.com/
This tool was used to help generate ideas for the outline, provide explanations of keywords, and offer feedback on specific prose.
Concrete example: changed “Radix tree - a data structure used in page tables” to “Radix tree - A tree-like data structure used to store key-value pairs compactly, where each node represents a prefix of the keys.”
Prose assistance could be overly verbose and ostentatious: “Unlike traditional systems that allow arbitrary mappings of physical addresses (PAs) to virtual addresses (VAs), Mosaic Pages imposes restrictions on these mappings due to the constraints of the Iceberg Hashing algorithm. This deterministic hashing mechanism is designed to optimize memory utilization and reduce TLB misses by compressing multiple virtual-to-physical mappings into a single TLB entry. However, the trade-off is a loss of flexibility. Addresses can only map to a limited set of physical frames determined by the hash function.”
Generative AI tools are excellent tools for brainstorming, offering feedback, and providing explanations. These tools should not be trusted for accuracy. Any specific detail mentioned must be externally validated.  
