# Understanding Linux Virtualization

### 1. What is Virtualization

> Virtualization (short for virtualization technology) is the function of physical hardware that is presented to an operating system.
>
> The physical system that runs the virtualization software is called a host.  
> The virtual machines installed are call guests. 

---

### 2. Types of Virtualization

> Virtualization can happen to any of the components: hardware, network, storage, application, access, ..

![Type of Virtualization](../img/type-of-virtualization.png)

---

### 3. Advantages of Virtualization

- **Server Consolidation (Hợp nhất máy chủ):** Reduce physical server, networking stack components, physical components -> power savings. Help with energy utilization by provising vm with the exact amount of CPU, mem, storage.

- **Service Isolation (Cô lập dịch vụ):** Consolidating many of virtual machines across fewer physical server -> application isolation and remove compatibility issues, simplify administration of services.

- **Faster Server Provisioning (Cung cấp máy chủ nhanh chóng):** Sprawn vm from prebuilt images (template) or snapshots. Not to worry about physical resource configuration.

- **Disaster Recovery (Khôi phục sau sự cố):** Allow up-to-date snapshoots. Virtualization offers vm migration -> move vm in data center.

- **Dynamic Load Balancing (Cân bằng tải động):** (depend on the policy) vm overultilizing the resources can be moved to underultilizing servers.

- **Faster Dev and Test Env (Phát triển và thử nghiệm nhanh):** Enable rapid deployment by isolating the application in a known and controlled environment.

- **Improved System Reliability and Secure (Cải thiện độ tin cậy và bảo mật):** Virtualization adds a layer of abstraction between the vm and the physical hardware, helps reduce risks caused by the infection.

- **OS Independence or a reduced hardware vendor lock-in (Độc lập hệ điều hành và giảm phụ thuộc vào nhà cung cấp):** VM don't care about the hardware they run on -> more flexibility when it comes to the server equipment choosing