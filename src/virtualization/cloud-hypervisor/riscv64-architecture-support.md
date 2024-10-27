# RISC-V 64-bit Architecture Support

To get a full picture of current cloud-hypervisor layout, we can utilize
`cargo-depgraph` tool, with command:

```sh
cargo depgraph --dedup-transitive-deps --features kvm --include arch,block,devices,event_monitor,hypervisor,net_gen,net_util,option_parser,pci,performance-metrics,rate_limiter,serial_buffer,test_infra,tracer,vhost_user_block,vhost_user_net,virtio-devices,vm-allocator,vm-device,vm-migration,vm-virtio,vmm,acpi_tables,kvm-bindings,kvm-ioctls,linux-loader,mshv-bindings,mshv-ioctls,seccompiler,vfio-bindings,vfio-ioctls,vfio_user,vhost,virtio-bindings,virtio-queue,vm-fdt,vm-memory,vmm-sys-util,cloud-hypervisor | dot -Tpng > architecture.png
```

With this command, a PNG will be generated demonstrating the overall
architecture of current cloud-hypervisor. The crates shown are crates within the
Rust Type II virtualization stack, and are composed primarily by
`cloud-hypervisor` team and `rust-vmm` team.

Crates from `rust-vmm` are the foundation of `cluod-hypervisor`, so these works
should be finished ahead of supporting `cloud-hypervisor` on RISC-V.

The crates used by `cloud-hypervisor` are:

- [x] `acpi_tables`
- [x] `kvm-bindings`
- [x] `kvm-ioctls`
- [x] `vm-memory`
- [x] `linux-loader`
- [x] `mshv-bindings`
- [x] `mshv-ioctls`
- [ ] `seccompiler`
- [x] `vfio-bindings`
- [x] `vfio-ioctls`
- [x] `vfio_user`
- [ ] `vhost`
- [x] `virtio-bindings`
- [x] `virtio-queue`
- [x] `vm-fdt`
- [x] `vmm-sys-util`
