# Copy Fail: When Copies Stop Being Copies

## 1. Introduction

[Last time](README.md), I explained Copy Fail from a beginner-friendly perspective.

However, during my research, I realized that Copy Fail is not simply a memory corruption bug. 

It also reveals deeper issues involving Linux optimization design and security boundaries.

In this article, I would like to explore the mechanisms and design choices that made Copy Fail possible.

## 2. Three Conditions That Enable Copy Fail

Xint researchers described Copy Fail as "portable, tiny, stealthy, and cross-container."

However, this vulnerability is much more than a catchy phrase or a simple memory corruption bug.

### 2-1. Write Primitive 

This vulnerability originates from the `authencesn` template in the Linux kernel crypto subsystem. 

Authencesn is normally used for encryption. 

However, this component can modify memory in an unintended way under specific conditions.

In exploit terminology, this kind of capability is called a “write primitive”.

### 2-2. Trigger Surface

Normally, user space programs cannot directly access kernel crypto operations.

Then what acts as the entry point for Copy Fail?

The answer is `AF_ALG`.

Usually AF_ALG is used to interact with the kernel crypto API from user space.

In this vulnerability, AF_ALG becomes the entry point to the vulnerable kernel functionality.

### 2-3. Target Selection

According to the Linux kernel documentation, the kernel tries to avoid unnecessary copy operations. 

This approach is known as zero-copy.

`splice()` follows the same philosophy by moving data within the kernel without repeatedly transferring it between kernel space and user space.

As a result, splice() can end up using the target file's cached memory pages directly. 

This can allow the attacker to influence which cached pages are targeted.

## 3. The Critical Optimization

In-place cipher operations are allowed through the AF_ALG interface.

Instead of copying data into a separate buffer, the same memory pages are modified directly during the encryption process.

This design helps reduce unnecessary memory copies and improve performance.

However, this optimization blurs the separation between private processing memory and shared cached memory.


## 4. When Optimizations Collide

Individually, authencesn, AF_ALG, splice(), and in-place operations are designed for performance and efficiency.

This is because Linux heavily values throughput, cache locality, and zero-copy processing.

However, when these optimizations interact together, they can unintentionally blur the boundary between temporary processing memory and shared cached memory.

As part of the mitigation, the Linux kernel developers removed the complexity added for in-place operations and returned to direct copying instead.


## 5. Conclusion

Traditionally, copied data is expected to remain isolated from shared cached memory.

However, Copy Fail shows that such traditional assumptions fail.

This is because modern optimizations reduce the separation between temporary processing memory and shared cached memory.

This kind of optimization collision has already contributed to other vulnerabilities as well.

I would like to continue exploring similar vulnerabilities in the future.


## References

- [NVD: CVE-2026-31431](https://nvd.nist.gov/vuln/detail/CVE-2026-31431)
- [Xint Original Writeup](https://xint.io/blog/copy-fail-linux-distributions)
- [Linux Kernel Documentation](https://docs.kernel.org/)
