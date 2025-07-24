# 01. Virtualization Main idea:
      - Virtualization is a software trick that lets one physical computer pretend to be many separate computers, each running its own 
        operating system and apps.


# 02. What “virtual” means in computing
      - "Virtual" means simulated by software rather than physically present.

      - Virtualization inserts a thin layer of code called a hypervisor between the hardware and the operating systems.

      - The Hypervisor slices the real CPU, memory, disk and network into chunks and gives each chunk a separate virtual machine (VM).

      - Hypervisor = the building manager.

      - Virtual machine = an apartment that looks and feels like a full house but shares the same concrete foundation.

      - Host = the actual building (your server).

      - Guest = the tenant OS living inside an apartment.


# 03. A simple real-world example
      - Imagine a large single-family house with four empty bedrooms. Only one person lives there, so most rooms stay unused wasted space,
        power and rent. A landlord decides to convert the house into 4 studio apartments by adding locks and utility meters for each room.
        Now 4 different tenants can live independently, cook and pay only for what they need while sharing the same roof, wiring and plumbing.
      
      - Virtualization does the same to a server: one machine is partitioned into multiple VMs so that several "tenants" (Windows Server, Ubuntu, a 
        database, a legacy payroll app) run side-by-side and use just the resources they need.


# 04. Key definitions
      a. Virtualization
         - Plain-English definition: The creation of a software-based (virtual) version of hardware such as servers, storage or networks	
         - Why it matters: Lets one physical machine act as many, improving utilization and lowering costs

      b. Hypervisor	
         - Plain-English definition: Software that slices hardware resources and hands them to VMs; called a Virtual Machine Monitor (VMM)
         - Why it matters: The core engine that makes virtualization possible
      
      c. Virtual Machine (VM)	
         - Plain-English definition: A software “computer” containing its own virtual CPU, memory, disk and OS, isolated from other VMs
         - Why it matters: Runs any OS or app just as if it were on real hardware


# 05. Types of hypervisors
      a. Type 1 (bare-metal)
         - installed directly on hardware, used in data-centers for efficiency (e.g., VMware ESXi, Microsoft Hyper-V)
      
      b. Type 2 (hosted)
         - runs as an app on top of an existing OS, handy for laptops (e.g., VirtualBox, Parallels)


# 06. How virtualization works—step by step
      a. Install hypervisor on the physical server (host).

      b. Create VM: Allocate 2 CPU cores, 4 GB RAM, 40 GB disk, pick an ISO file.

      c. Boot VM: The guest OS thinks it owns real hardware, but every instruction is intercepted by the hypervisor and scheduled on the host CPU

      d. Run multiple VMs: Repeat step 2 for each workload; the hypervisor juggles resources so they don’t clash.


# 07. Why people use virtualization
      a. Better hardware use: One server hosting 10 VMs often runs at 60-70% CPU instead of 10% on separate under-used boxes

      b. Cost and energy savings: Fewer physical servers to buy, power and cool

      c. Flexibility: Spin up a new VM in minutes for testing, then delete it with no hardware waste

      d. Isolation & safety: If one VM crashes or gets malware, others keep running—like tenants in separate apartments

      e. Foundation for cloud: Public clouds (AWS, Azure, Google Cloud) rely on massive fleets of virtualized servers to sell compute by the hour


# 08. Where you meet virtualization in real life
      - Streaming services such as Netflix add VMs on demand during evening traffic peaks instead of buying new servers for every surge

      - Developers run Linux VMs on Windows laptops to test software without dual-booting.

      - Businesses consolidate dozens of old servers onto a few powerful hosts, freeing floor space and cutting electricity bills
      



