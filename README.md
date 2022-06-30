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

**Securing a Pod**

Note: AppArmor is currently in beta, so options are specified as annotations. Once support graduates to general availability, the annotations will be replaced with first-class fields

AppArmor profiles are specified per-container. To specify the AppArmor profile to run a Pod container with, add an annotation to the Pod's metadata:

![image](https://user-images.githubusercontent.com/88305831/176437138-a5efa8b1-4459-46f2-be39-272c9f9de78b.png)

Where <container_name> is the name of the container to apply the profile to, and <profile_ref> specifies the profile to apply. The profile_ref can be one of:

* runtime/default to apply the runtime's default profile
* localhost/<profile_name> to apply the profile loaded on the host with the name <profile_name>
* unconfined to indicate that no profiles will be loaded

**Let's start with AppArmor Installation and Configuration"


Kubernetes AppArmor enforcement works by first checking that all the prerequisites have been met, and then forwarding the profile selection to the container runtime for enforcement. If the prerequisites have not been met, the Pod will be rejected, and will not run.

To verify that the profile was applied, you can look for the AppArmor security option listed in the container created event:

*$ kubectl get events | grep Created*

![image](https://user-images.githubusercontent.com/88305831/176437841-87b9988d-ee83-40d9-98fa-1a9b7db1693d.png)

You can also verify directly that the container's root process is running with the correct profile by checking its proc attr:

*$ kubectl exec <pod_name> -- cat /proc/1/attr/current*

![image](https://user-images.githubusercontent.com/88305831/176438566-f70757f5-82e0-4e16-b788-f7535de37dd4.png)

**Example**

This example assumes you have already set up a cluster with AppArmor support.

First, we need to load the profile we want to use onto our nodes. This profile denies all file writes to /etc, /var and /tmp filesystem:

![image](https://user-images.githubusercontent.com/88305831/176439861-c2ef97df-ce16-44f5-9c44-3d505636498b.png)

Since we don't know where the Pod will be scheduled, we'll need to load the profile on all our nodes. For this example we'll use SSH to install the profiles, but other approaches are discussed in later section.

Copy the k8s-apparmor-example-deny-write file to /etc/apparmor.d location on all nodes.

*$ scp k8s-apparmor-example-deny-write root@node1:/etc/apparmor.d/*

Next, we'll run a simple "Hello AppArmor" pod with the deny-write profile:

![image](https://user-images.githubusercontent.com/88305831/176441168-92bec4a7-437d-475f-adb5-733e22f4e252.png)

*$ kubectl apply -f pod.yaml*

We can verify that the container is actually running with that profile by checking its proc attr:

*$ kubectl exec hello-apparmor -- cat /proc/1/attr/current*

![image](https://user-images.githubusercontent.com/88305831/176450660-79c03108-fa40-4797-92c3-e6c43d50d127.png)

Finally, we can see what happens if we try to violate the profile by writing to a file:

*$ kubectl exec hello-apparmor -- touch /etc/test*

![image](https://user-images.githubusercontent.com/88305831/176451268-30da452d-5085-4c93-a084-e467155a2db0.png)

**Now let's focus on some AppArmor commands**

* You can view the loaded AppArmor profile with below command.

*$ aa-status* or *$ apparmor_status*

* aa-complain places a profile into complain mode.

*$ sudo aa-complain /path/to/bin*

* aa-enforce places a profile into enforce mode.

*$ sudo aa-enforce /path/to/bin*


* The /etc/apparmor.d directory is where the AppArmor profiles are located. It can be used to manipulate the mode of all profiles.

* Enter the following to place all profiles into complain mode:

*$ sudo aa-complain /etc/apparmor.d/**

* To place all profiles in enforce mode:

*$ sudo aa-enforce /etc/apparmor.d/**

* apparmor_parser is used to load a profile into the kernel. It can also be used to reload a currently loaded profile using the -r option after modifying it to have the changes take effect.
To reload a profile:

*$ sudo apparmor_parser -r /etc/apparmor.d/profile.name*

* systemctl can be used to reload all profiles:

*$ sudo systemctl reload apparmor.service*

* The /etc/apparmor.d/disable directory can be used along with the apparmor_parser -R option to disable a profile.

*$ sudo ln -s /etc/apparmor.d/profile.name /etc/apparmor.d/disable/*

*$ sudo apparmor_parser -R /etc/apparmor.d/profile.name*

* To re-enable a disabled profile remove the symbolic link to the profile in /etc/apparmor.d/disable/. Then load the profile using the -a option.

*$ sudo rm /etc/apparmor.d/disable/profile.name*

*$ cat /etc/apparmor.d/profile.name | sudo apparmor_parser -a*

* AppArmor can be disabled, and the kernel module unloaded by entering the following:

*$ sudo systemctl stop apparmor.service*
*$ sudo systemctl disable apparmor.service*

* To re-enable AppArmor enter:

*$ sudo systemctl enable apparmor.service*
*$ sudo systemctl start apparmor.service*

**Note**

Replace profile.name with the name of the profile you want to manipulate. Also, replace /path/to/bin/ with the actual executable file path. For example for the ping command use /bin/ping

Now think of the challenges of implementing AppArmor in production.

First, you will have to build robust profiles for each of your containers to prevent attacks without blocking daily tasks.

Then, you will have to manage several profiles across all the nodes in your cluster.

kube-apparmor-manager tool will help to manage AppArmor profiles for Kubernetes cluster.










