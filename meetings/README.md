# Meetings

## June

### Experiment Idea

1. Start with hdf5 program that we know has "gaps"
2. Run with [Darshan](https://github.com/darshan-hpc/darshan), [IOR](https://github.com/hpc/ior), [score-p](https://www.vi-hps.org/projects/score-p) etc. Are there gaps?
3. Figure out calls we want in eBPF and try to instrument
 - First try local: https://bpfman.io/main/getting-started/example-bpf-local/
 - Then move into Kubernetes
 - If that works, metrics server or similar (need to digest nom nom data)

### Storage Patterns:

- store in memory while running
- store events as is, keep everything
- ship it to somewhere else (streaming) dashboard or database 🛳️ 
 
### System Calls we care about

- Hari wants all the calls!! All the calls!
- "Exporatory analysis" use a subset of traces
  - MangoIO
  - Look at paper to get sense of calls 
  - There is an exhaustive list but then they filter down to interesting subset
  
> What can ebpf capture? What calls are at the kernel level? How do we store it correctly?

