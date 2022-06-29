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

  *$ kubectl get nodes -o=jsonpath=$'{range .items[*]}{@.metadata.name}: {@.status.nodeInfo.kubeletVersion}\n{end}'*

![image](https://user-images.githubusercontent.com/88305831/176432165-4dfc9f35-a6b3-4ef6-913b-de00911a4d97.png)

2. AppArmor kernel module is enabled -- For the Linux kernel to enforce an AppArmor profile, the AppArmor kernel module must be installed and enabled. Several distributions enable the module by default, such as Ubuntu and SUSE, and many others provide optional support. To check whether the module is enabled, check the /sys/module/apparmor/parameters/enabled file:

  *$ cat /sys/module/apparmor/parameters/enabled*

![image](https://user-images.githubusercontent.com/88305831/176432647-0d2d7402-815c-4cc0-a734-371bb20883c1.png)

If the Kubelet contains AppArmor support (>= v1.4), it will refuse to run a Pod with AppArmor options if the kernel module is not enabled.

3. Container runtime supports AppArmor -- Currently all common Kubernetes-supported container runtimes should support AppArmor, like Docker, CRI-O or containerd. Please refer to the corresponding runtime documentation and verify that the cluster fulfills the requirements to use AppArmor.

4. Profile is loaded -- AppArmor is applied to a Pod by specifying an AppArmor profile that each container should be run with. If any of the specified profiles is not already loaded in the kernel, the Kubelet (>= v1.4) will reject the Pod. You can view which profiles are loaded on a node by checking the /sys/kernel/security/apparmor/profiles file. For example:

*$ ssh node1 "sudo cat /sys/kernel/security/apparmor/profiles | sort"*

![image](https://user-images.githubusercontent.com/88305831/176433446-970875b6-9fc8-40fa-a75e-e2dbc321580b.png)

As long as the Kubelet version includes AppArmor support (>= v1.4), the Kubelet will reject a Pod with AppArmor options if any of the prerequisites are not met. You can also verify AppArmor support on nodes by checking the node ready condition message (though this is likely to be removed in a later release):

*$ kubectl get nodes -o=jsonpath=$'{range .items[*]}{@.metadata.name}: {.status.conditions[?(@.reason=="KubeletReady")].message}\n{end}'*

![image](https://user-images.githubusercontent.com/88305831/176433968-0f3b2dc5-9d56-4caf-ac72-4e79f5be1332.png)

Incase AppArmor is not installed then you can install it on Ubuntu system using below commands.

*$ apt install apparmor-profiles*

*$ apt install apparmor-utils*

To check the AppArmor Install Status

*$ systemctl status apparmor*



**Let's start with AppArmor Installation and Configuration"




