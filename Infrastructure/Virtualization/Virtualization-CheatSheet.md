# 虚拟化与容器技术原理与实践导论

**Machine-level virtualization**, such as [KVM](https://www.linux-kvm.org/) and [Xen](https://www.xenproject.org/), exposes virtualized hardware to a guest kernel via a Virtual Machine Monitor (VMM). This virtualized hardware is generally enlightened (paravirtualized) and additional mechanisms can be used to improve the visibility between the guest and host (e.g. balloon drivers, paravirtualized spinlocks). Running containers in distinct virtual machines can provide great isolation, compatibility and performance (though nested virtualization may bring challenges in this area), but for containers it often requires additional proxies and agents, and may require a larger resource footprint and slower start-up times.

[![Machine-level virtualization](https://github.com/google/gvisor/raw/master/g3doc/Machine-Virtualization.png)](https://github.com/google/gvisor/blob/master/g3doc/Machine-Virtualization.png)

**Rule-based execution**, such as [seccomp](https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt), [SELinux](https://selinuxproject.org/) and [AppArmor](https://wiki.ubuntu.com/AppArmor), allows the specification of a fine-grained security policy for an application or container. These schemes typically rely on hooks implemented inside the host kernel to enforce the rules. If the surface can be made small enough (i.e. a sufficiently complete policy defined), then this is an excellent way to sandbox applications and maintain native performance. However, in practice it can be extremely difficult (if not impossible) to reliably define a policy for arbitrary, previously unknown applications, making this approach challenging to apply universally.

[![Rule-based execution](https://github.com/google/gvisor/raw/master/g3doc/Rule-Based-Execution.png)](https://github.com/google/gvisor/blob/master/g3doc/Rule-Based-Execution.png)

Rule-based execution is often combined with additional layers for defense-in-depth.

**gVisor** provides a third isolation mechanism, distinct from those mentioned above.

gVisor intercepts application system calls and acts as the guest kernel, without the need for translation through virtualized hardware. gVisor may be thought of as either a merged guest kernel and VMM, or as seccomp on steroids. This architecture allows it to provide a flexible resource footprint (i.e. one based on threads and memory mappings, not fixed guest physical resources) while also lowering the fixed costs of virtualization. However, this comes at the price of reduced application compatibility and higher per-system call overhead.

[![gVisor](https://github.com/google/gvisor/raw/master/g3doc/Layers.png)](https://github.com/google/gvisor/blob/master/g3doc/Layers.png)

On top of this, gVisor employs rule-based execution to provide defense-in-depth (details below).

gVisor's approach is similar to [User Mode Linux (UML)](http://user-mode-linux.sourceforge.net/), although UML virtualizes hardware internally and thus provides a fixed resource footprint.
