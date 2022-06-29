# apparmor
**Restrict a Container's Access to Resources with AppArmor**

AppArmor is a Linux kernel security module that supplements the standard Linux user and group based permissions to confine programs to a limited set of resources. AppArmor can be configured for any application to reduce its potential attack surface and provide greater in-depth defense. 
It is configured through profiles tuned to allow the access needed by a specific program or container, such as Linux capabilities, network access, file permissions, etc. Each profile can be run in either enforcing mode, which blocks access to disallowed resources, or complain mode, which only reports violations.

AppArmor can help you to run a more secure deployment by restricting what containers are allowed to do, and/or provide better auditing through system logs. However, it is important to keep in mind that AppArmor is not a silver bullet and can only do so much to protect against exploits in your application code. It is important to provide good, restrictive profiles, and harden your applications and cluster from other angles as well.

**Let's check for some concepts.**

**Linux SysCalls**

As the name suggests, syscalls are system calls, and they're the way that you can make requests from user space into the Linux kernel. 
The kernel does some work for you, like creating a process, then hands control back to user space.
![AppArmor1](https://user-images.githubusercontent.com/88305831/176425926-ef186985-45c4-4af0-8948-672fb3ea6bfa.png)

**Use case of AppArmor**

![AppArmor2](https://user-images.githubusercontent.com/88305831/176426716-52a71d06-4e84-4bc0-ba83-7911000ff4f8.png)


**AppArmor Implementation Steps**

![AppArmor3_Implementation txt](https://user-images.githubusercontent.com/88305831/176427141-3b5221c7-75c7-4253-8279-6eb4e75ef247.png)

**Sample AppArmor Profile**

![AppArmor4_Sample_AppArmor_profile](https://user-images.githubusercontent.com/88305831/176427250-4107855b-4f74-4290-8107-c3296d8f1d2e.png)

**Apply AppArmor Profile to Pod**

![AppArmor5_Apply_to_Pod](https://user-images.githubusercontent.com/88305831/176427353-48993c02-0732-4333-9806-ffa1b51d70c0.png)

**Prereqisites for AppArmor**

1. Kubernetes version is at least v1.4 -- Kubernetes support for AppArmor was added in v1.4. Kubernetes components older than v1.4 are not aware of the new AppArmor annotations, and will silently ignore any AppArmor settings that are provided. To ensure that your Pods are receiving the expected protections, it is important to verify the Kubelet version of your nodes:



**Let's start with AppArmor Installation and Configuration"




