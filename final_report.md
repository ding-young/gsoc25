# GSoC'25 project report 
Contributor: SeoYoung Lee([@ding-young](https://github.com/ding-young))

Mentor: Yongting You([@2010YOUY01](https://github.com/2010YOUY01))

Project: [Improving Spilling Execution in DataFusion](https://summerofcode.withgoogle.com/programs/2025/projects/3Ooae7Uu)

Organization: [Apache DataFusion](https://github.com/apache/datafusion)

## Overview
Apache DataFusion is an in-memory query execution engine optimized for OLAP workloads using Apache Arrow's columnar format. However, to support queries that exceed available memory, DataFusion must spill intermediate results (e.g., from sort or aggregate operators) to disk. As a continuation of the community effort on external query execution, this project aims to improve the robustness of spilling execution and explore further performance optimizations including compression. 

In particular, the goals include:
- Enhancing tests and benchmarks for spilling execution

- Improving user experience when queries fail under memory pressure

- Identifying and addressing the root causes of query failures during external sorting
## What work was done
### Onboarding: Understanding Memory Pool and External Sort
- [Improve error message on Out of Memory](https://github.com/apache/datafusion/pull/16050)
- [Set TrackConsumersPool as default in datafusion-cli](https://github.com/apache/datafusion/pull/16081)
- [Closed: Track peak_mem_used in ExternalSorter
](https://github.com/apache/datafusion/pull/16192)

To familiarize myself with the codebase, I started by exploring how DataFusion handles memory usage and external query execution. I focused particularly on the ExternalSorter, which is responsible for spilling intermediate results to disk. During this phase, I contributed small but valuable improvements, such as enhancing the error message for OOM conditions and enabling a more informative memory tracking pool by default in the CLI.

### Enable compression on spill
- [Add compression option to SpillManager](https://github.com/apache/datafusion/pull/16268)
- [Add microbenchmark for spilling with compression](https://github.com/apache/datafusion/pull/16512)
- [Update spilled_bytes metric to reflect actual disk usage](https://github.com/apache/datafusion/pull/16535)

Spilling intermediate data to disk previously involved writing raw data without compression. To reduce disk I/O and spill file size, I implemented support for general-purpose compression codecs (e.g., lz4_framed, zstd) in the SpillManager. I also updated the spilled_bytes metric to reflect actual file size after compression, and added benchmarks to compare performance and disk usage across compression options.

### Enhancing benchmarks and Memory Profiling 
- [Update tpch, clickbench, sort_tpch to mark failed queries](https://github.com/apache/datafusion/pull/16182)
- [Add benchmark utility to profile peak memory usage](https://github.com/apache/datafusion/pull/16814)
- [Validate the memory consumption in SPM created by multi level merge](https://github.com/apache/datafusion/pull/17029)

Benchmark suites such as TPC-H and ClickBench are commonly used in DataFusion to evaluate memory-bounded query execution. However, failures due to memory pressure were not clearly reported. I improved the benchmark scripts to track and display failed queries explicitly. Furthermore, to verify memory accounting accuracy, I introduced a utility (mem_profile) that launches each benchmark query as a subprocess and reports system-level peak RSS and allocator-level memory usage (via mimalloc).

### Introducing & Stabilizing Multi-level merge
- [feat: add multi level merge sort](https://github.com/apache/datafusion/pull/15700)
- [Chore: remove 'spill_record_batch_by_size' api](https://github.com/apache/datafusion/pull/16958)

Once data is spilled to disk, queries later reading these spilled files back. In the case of sorting, spilled files contain sorted record batches, which need to be merged into a globally sorted output. However, reading too many spill files at once can lead to memory exhaustion. To address this, datafusion needed multi-level merge functionality to allows merging files in multiple stages, reducing memory pressure.

I participated in the review and validation process for enabling multi-level merge and worked on stabilizing its behavior. This included removing obsolete APIs and validating that the memory used during the merge phase matches expectations.

- [Fix incorrect memory accounting for sliced StringViewArray](https://github.com/apache/datafusion/pull/17315)
- [fix: set IPC alignment based on schema](https://github.com/apache/datafusion/pull/17363)

This validation work also helped discover several bugs related to memory accounting. These fixes improve the accuracy of memory estimation and help prevent unexpected memory blow-up when reading spilled files back into memory.

### and others
- [Add tests for yielding in SpillManager::read_spill_as_stream](https://github.com/apache/datafusion/pull/16616)
- [Fix RowConverter panic when encoding DictionaryArrays in StructArray / ListArray](https://github.com/apache/arrow-rs/pull/7627)
- [Improve memory usage for arrow-row -> String/BinaryView when utf8 validation disabled](https://github.com/apache/arrow-rs/pull/7917)
- [Add benchmark for converting StringViewArray with mixed short and long strings](https://github.com/apache/arrow-rs/pull/8015)

In addition, I contributed to issues related to the spilling and the arrow-row module, which plays a critical role in the merge phase of sorting.

## Current status & Future works
- [Improve memory accounting for cursor in external sort](https://github.com/apache/datafusion/pull/17163)

TODO 

## Any challenges or important things I learned during the project.
TODO

## Acknowledgement
TODO