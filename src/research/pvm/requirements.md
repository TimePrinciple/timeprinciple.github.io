# PVM Background

In cloud-native environments, containers are often deployed within lightweight
virtual machines (Light-VMs) to ensure strong security isolation and privacy
protection. With the growing demand for customized cloud services, third-party
vendors are turning to infrastructure-as-a-service (IaaS) cloud providers to
build their own cloud-native platforms, necessitating the need to run a VM or a
guest that hosts containers inside another VM instance leased from an IaaS
cloud. State-of-the-art nested virtualization in the x86 architecture relies
heavily on the host hypervisor to expose hardware virtualization support to the
guest hypervisor, not only complicating cloud management but also raising
concerns about an increased attack surface at the host hypervisor.


