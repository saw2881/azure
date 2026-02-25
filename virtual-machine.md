# Azure Virtual Machine

## 1. Virtual Network (VNet)

A logically isolated network in Azure that enables VMs and other resources to communicate securely.

### Subnet
- Logical subdivision of a VNet
- Used to segment network traffic and apply security rules
- Each subnet has its own IP address range (CIDR block)
- VMs are deployed into subnets, not directly into VNets
- **Types:**
  - Default subnet - general workloads
  - GatewaySubnet - required for VPN/ExpressRoute gateways
  - AzureBastionSubnet - for Azure Bastion host
  - AzureFirewallSubnet - for Azure Firewall

### CIDR (Classless Inter-Domain Routing)

A method for allocating IP addresses and defining network ranges using prefix notation.

**Format:** `IP_Address/Prefix_Length`

The prefix length (after the `/`) indicates how many bits are used for the network portion. The remaining bits are for host addresses.

#### Common CIDR Blocks

| CIDR | Subnet Mask | Usable Hosts | Example Range |
|------|-------------|--------------|---------------|
| /8 | 255.0.0.0 | 16,777,214 | 10.0.0.0 - 10.255.255.255 |
| /16 | 255.255.0.0 | 65,534 | 10.0.0.0 - 10.0.255.255 |
| /24 | 255.255.255.0 | 254 | 10.0.0.0 - 10.0.0.255 |
| /28 | 255.255.255.240 | 14 | 10.0.0.0 - 10.0.0.15 |
| /32 | 255.255.255.255 | 1 | Single IP address |

#### Example: VNet with Subnets

```
VNet Address Space: 10.0.0.0/16 (65,536 addresses)
│
├── Subnet-Web:      10.0.1.0/24  (256 addresses: 10.0.1.0 - 10.0.1.255)
├── Subnet-App:      10.0.2.0/24  (256 addresses: 10.0.2.0 - 10.0.2.255)
├── Subnet-DB:       10.0.3.0/24  (256 addresses: 10.0.3.0 - 10.0.3.255)
└── GatewaySubnet:   10.0.255.0/27 (32 addresses: 10.0.255.0 - 10.0.255.31)
```

#### Azure Reserved IPs (Per Subnet)
Azure reserves **5 IP addresses** in each subnet:
- `.0` - Network address
- `.1` - Default gateway
- `.2, .3` - Azure DNS
- `.255` - Broadcast address

**Example:** In a /24 subnet (256 IPs), only **251 usable** for VMs.

#### Quick Reference
- **Larger prefix** (e.g., /28) = Smaller network, fewer hosts
- **Smaller prefix** (e.g., /16) = Larger network, more hosts
- Azure VNet minimum: **/29** (8 IPs, 3 usable)
- Azure VNet maximum: **/8** (for address space)

### Key Points
- VNets are region-specific (cannot span regions)
- VNet peering allows communication between VNets (same or different regions)
- Address space must not overlap with on-premises or other VNets you want to connect
- Private IP addresses assigned from subnet range

---

## 2. Virtual Hard Disk (VHD)

Storage disks attached to VMs for operating system and data.

### OS Disk
- Required for every VM
- Contains the operating system
- Automatically created when VM is provisioned
- **Max size:** 4,095 GB (4 TB)
- Registered as SATA drive

### Data Disk
- Optional additional storage
- Used for application data, databases, etc.
- Number of data disks depends on VM size
- **Max size:** 32,767 GB (32 TB)
- Registered as SCSI drive

### Temporary Disk
- Non-persistent storage (data lost on redeployment/resize)
- Included with most VM sizes
- Used for pagefile/swap, temporary data
- **Do NOT store important data here**

### Disk Types (Performance Tiers)

| Type | IOPS | Throughput | Use Case |
|------|------|------------|----------|
| **Standard HDD** | Up to 2,000 | Up to 500 MB/s | Dev/test, backup, infrequent access |
| **Standard SSD** | Up to 6,000 | Up to 750 MB/s | Web servers, light enterprise apps |
| **Premium SSD** | Up to 20,000 | Up to 900 MB/s | Production, databases, I/O intensive |
| **Premium SSD v2** | Up to 80,000 | Up to 1,200 MB/s | High-performance databases, analytics |
| **Ultra Disk** | Up to 400,000 | Up to 10,000 MB/s | SAP HANA, top-tier databases, transaction-heavy |

### Managed vs Unmanaged Disks
- **Managed Disks (Recommended):** Azure handles storage account; easier, scalable, 99.999% availability
- **Unmanaged Disks (Legacy):** You manage storage accounts; page blobs in your storage account

---

## 3. Network Interface Card (NIC)

The interconnection between a VM and a virtual network.

### Features
- Every VM requires at least one NIC
- Number of NICs supported depends on VM size
- Each NIC can have multiple IP configurations
- NICs are attached to a subnet within a VNet

### IP Configuration
- **Private IP:** Communication within VNet and connected networks
  - Dynamic (DHCP from subnet) or Static
- **Public IP:** Communication from the internet
  - Dynamic or Static
  - Basic SKU (free, no zone redundancy) or Standard SKU (zone-redundant, secure by default)

### Key Points
- NICs can be added/removed from stopped (deallocated) VMs
- Can associate NSG at NIC level for granular security
- Accelerated Networking available for high-performance scenarios

---

## 4. Network Security Group (NSG)

A firewall that filters network traffic to and from Azure resources.

### Security Rules
- **Inbound rules:** Control traffic coming into the VM
- **Outbound rules:** Control traffic leaving the VM

### Rule Properties
| Property | Description |
|----------|-------------|
| **Priority** | 100-4096; lower number = higher priority |
| **Source/Destination** | IP, CIDR, Service Tag, or ASG |
| **Port** | Single port, range, or * (all) |
| **Protocol** | TCP, UDP, ICMP, or * (any) |
| **Action** | Allow or Deny |

### Default Rules (Cannot be deleted)
- **AllowVNetInBound** - Allow traffic within VNet
- **AllowAzureLoadBalancerInBound** - Allow health probes
- **DenyAllInBound** - Deny all other inbound (priority 65500)
- **AllowVNetOutBound** - Allow outbound within VNet
- **AllowInternetOutBound** - Allow outbound to internet
- **DenyAllOutBound** - Deny all other outbound (priority 65500)

### Association
- **Subnet level:** Applies to all resources in the subnet
- **NIC level:** Applies only to the specific VM
- Both can be applied; most restrictive rule wins

### Service Tags
Pre-defined groups of IP addresses (e.g., Internet, VirtualNetwork, AzureLoadBalancer, Storage, Sql)

### Application Security Groups (ASG)
Group VMs logically for easier NSG rule management without managing IP addresses

---

## 5. Availability Options

Strategies to ensure VM uptime and resilience.

### No Infrastructure Redundancy
- Single VM, no high availability
- SLA: 99.9% (with Premium SSD)
- **Use case:** Dev/test, non-critical workloads

### Availability Zones
Physically separate data centers within an Azure region, each with independent power, cooling, and networking.

#### Zone Structure
```
Azure Region (e.g., East US)
┌─────────────────────────────────────────────────────┐
│                                                     │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐       │
│  │  Zone 1  │   │  Zone 2  │   │  Zone 3  │       │
│  │  (DC A)  │   │  (DC B)  │   │  (DC C)  │       │
│  │          │   │          │   │          │       │
│  │  VM1     │   │  VM2     │   │  VM3     │       │
│  │  DB-Pri  │   │  DB-Sec  │   │  LB      │       │
│  └──────────┘   └──────────┘   └──────────┘       │
│       │              │              │              │
│       └──────────────┼──────────────┘              │
│            High-speed, low-latency                 │
│            interconnect (<2ms RTT)                 │
└─────────────────────────────────────────────────────┘
```

#### Key Characteristics
- **3 zones** per supported region (labeled 1, 2, 3)
- Zones are **physically isolated** (separate buildings, often miles apart)
- Connected via high-bandwidth, low-latency fiber (<2ms round-trip)
- Zone mapping is **unique per subscription** (Zone 1 in your subscription may be different DC than Zone 1 in another)

#### Zone-Redundant Services
| Service Type | Description | Example |
|--------------|-------------|---------|
| **Zonal** | Resource pinned to specific zone | VM in Zone 1 |
| **Zone-redundant** | Automatically replicated across zones | ZRS Storage, Zone-redundant LB |

#### Supported Resources
- Virtual Machines
- Managed Disks (ZRS)
- Public IP addresses (Standard SKU)
- Load Balancer (Standard SKU)
- VPN Gateway / ExpressRoute Gateway
- Azure SQL Database
- Azure Kubernetes Service (AKS)
- Storage Accounts (ZRS, GZRS)

#### Deployment Considerations
- Select zone at **VM creation time** (cannot change later)
- Deploy **minimum 2 VMs across 2+ zones** for HA
- Use **Standard Load Balancer** (Basic doesn't support zones)
- **Zone-redundant storage (ZRS)** for shared data across zones
- Managed disks are **zonal by default** (pinned to VM's zone)

#### When to Use
| Scenario | Recommendation |
|----------|----------------|
| Max availability within region | Availability Zones |
| Legacy apps, can't redesign | Availability Sets |
| Auto-scaling stateless apps | VMSS across zones |
| Disaster recovery (region failure) | Azure Site Recovery + paired region |

#### Key Points
- Not all regions support Availability Zones
- Inter-zone traffic incurs **bandwidth charges**
- SLA: **99.99%** (requires 2+ VMs across 2+ zones)
- **Use case:** Mission-critical production, databases, enterprise apps

### Availability Sets
A logical grouping of VMs within a single data center that ensures VMs are distributed across multiple physical hardware nodes.

#### Fault Domains (FD)
- Separate physical racks with independent power and network switch
- Protects against **hardware failures** (rack-level)
- **Maximum: 3 FDs** per availability set
- VMs spread across FDs won't all fail if one rack goes down

#### Update Domains (UD)
- Groups of VMs that can be rebooted together during **planned maintenance**
- Azure updates one UD at a time, waiting 30 minutes between each
- **Maximum: 20 UDs** per availability set (default: 5)
- Ensures at least some VMs remain running during updates

#### How It Works
```
Availability Set (3 FD x 5 UD)
┌─────────────┬─────────────┬─────────────┐
│   FD 0      │   FD 1      │   FD 2      │
│  (Rack 1)   │  (Rack 2)   │  (Rack 3)   │
├─────────────┼─────────────┼─────────────┤
│ VM1 (UD0)   │ VM2 (UD1)   │ VM3 (UD2)   │
│ VM4 (UD3)   │ VM5 (UD4)   │ VM6 (UD0)   │
└─────────────┴─────────────┴─────────────┘
```

#### Key Points
- VMs must be added to availability set **at creation time**
- All VMs in set should have the **same configuration**
- Use with **Azure Load Balancer** for traffic distribution
- Cannot span multiple regions or data centers
- SLA: **99.95%** (requires 2+ VMs)
- **Use case:** Legacy HA, workloads that don't support Availability Zones

### Virtual Machine Scale Sets (VMSS)
A group of identical, auto-scaling VMs managed as a single resource for high availability and elasticity.

#### Key Features
- All VMs created from the **same base image** and configuration
- Supports up to **1,000 VMs** (or 600 with custom images)
- Integrated with **Azure Load Balancer** or **Application Gateway**
- Can span **Availability Zones** for zone redundancy
- **Uniform** or **Flexible** orchestration modes

#### Orchestration Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| **Uniform** | Identical VMs, optimized for large stateless workloads | Web frontends, container hosts, batch |
| **Flexible** | Mix VM sizes/types, manual or auto-scaling | More control, gradual VMSS migration |

#### Scaling Options

**Autoscale (Metric-based)**
- Scale based on CPU, memory, queue length, custom metrics
- Define min/max instance count and scale rules
- Example: Add 2 VMs when CPU > 75% for 5 minutes

**Scheduled Scaling**
- Scale at specific times (e.g., business hours)
- Predictable workload patterns

**Manual Scaling**
- Set instance count directly via portal/CLI/API

#### Scaling Configuration Example
```
VMSS Configuration:
├── Min instances: 2
├── Max instances: 10
├── Default: 3
│
├── Scale-out rule:
│   └── CPU > 75% for 5 min → Add 2 VMs (cooldown: 5 min)
│
└── Scale-in rule:
    └── CPU < 25% for 10 min → Remove 1 VM (cooldown: 10 min)
```

#### Upgrade Policies
- **Automatic:** VMs updated immediately when model changes
- **Rolling:** Updates in batches with configurable batch size and pause
- **Manual:** You control when each VM is upgraded

#### Health Monitoring
- **Application Health Extension:** Probe endpoint on each VM
- **Automatic Repairs:** Replace unhealthy instances automatically
- Grace period configurable for startup time

#### Key Points
- Use **custom images** or **Azure Marketplace** images
- Supports **Spot VMs** for cost savings (can be evicted)
- Data disks attached to all instances
- Use **cloud-init** or **Custom Script Extension** for initialization
- Works with **Azure DevOps** and **GitHub Actions** for CI/CD
- SLA: **99.95%** (single zone) or **99.99%** (multi-zone)
- **Use case:** Stateless apps, web/API tiers, batch processing, container hosts

### Comparison Table

| Option | Protects Against | SLA | Max VMs |
|--------|------------------|-----|---------|
| Single VM (Premium SSD) | - | 99.9% | 1 |
| Availability Set | Rack failure, maintenance | 99.95% | 200 |
| Availability Zone | Data center failure | 99.99% | Per zone |
| VMSS + Zones | Data center failure + auto-scale | 99.99% | 1,000 |

---

## 6. VM Sizes (Series)

### General Purpose (B, D, Dv2-v5, DC)
- Balanced CPU-to-memory ratio
- **Use case:** Dev/test, small-medium databases, web servers
- **B-series:** Burstable, cost-effective for variable workloads

### Compute Optimized (F, Fs, Fsv2)
- High CPU-to-memory ratio
- **Use case:** Batch processing, gaming servers, analytics

### Memory Optimized (E, Ev3-v5, M, Mv2)
- High memory-to-CPU ratio
- **Use case:** Relational databases, in-memory caching, SAP HANA

### Storage Optimized (Ls, Lsv2, Lsv3)
- High disk throughput and IOPS
- **Use case:** Big data, SQL/NoSQL databases, data warehousing

### GPU (NC, ND, NV, NCas, NDm)
- GPU-powered for graphics and compute
- **Use case:** Machine learning, rendering, video encoding, simulations

### High Performance Compute (HB, HC, HBv2-v4)
- Fastest CPUs with high-bandwidth networking
- **Use case:** Scientific simulations, financial modeling, weather forecasting

---

## 7. Additional VM Features

### Azure Bastion
- Secure RDP/SSH access without public IP
- Connects via Azure Portal over TLS
- No need to expose ports 3389/22

### Boot Diagnostics
- Screenshot and serial console logs for troubleshooting
- Stored in storage account

### Azure Backup
- Automated backup with retention policies
- Uses Recovery Services Vault

### Azure Site Recovery (ASR)
- Disaster recovery replication to secondary region
- Automated failover and failback

### Extensions
- Post-deployment configuration and automation
- Examples: Custom Script, DSC, Antimalware, Monitoring Agent

### Spot VMs
- Unused Azure capacity at up to 90% discount
- Can be evicted when Azure needs capacity
- **Use case:** Batch jobs, dev/test, fault-tolerant workloads

### Reserved Instances
- 1 or 3-year commitment for up to 72% savings
- Best for predictable, steady-state workloads

---

## 8. Hands-On Exercises

### Exercise 1: Create a VNet with Subnets
**Objective:** Create a virtual network with multiple subnets using Azure CLI

```bash
# Create resource group
az group create --name rg-lab-networking --location eastus

# Create VNet with address space 10.0.0.0/16
az network vnet create \
  --resource-group rg-lab-networking \
  --name vnet-lab \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-web \
  --subnet-prefix 10.0.1.0/24

# Add additional subnets
az network vnet subnet create \
  --resource-group rg-lab-networking \
  --vnet-name vnet-lab \
  --name subnet-app \
  --address-prefix 10.0.2.0/24

az network vnet subnet create \
  --resource-group rg-lab-networking \
  --vnet-name vnet-lab \
  --name subnet-db \
  --address-prefix 10.0.3.0/24

# Verify subnets
az network vnet subnet list \
  --resource-group rg-lab-networking \
  --vnet-name vnet-lab \
  --output table
```

---

### Exercise 2: Deploy a VM with NSG Rules
**Objective:** Create a VM and configure inbound rules for SSH/RDP access

```bash
# Create NSG
az network nsg create \
  --resource-group rg-lab-networking \
  --name nsg-web

# Add SSH rule (Linux) - only from your IP
az network nsg rule create \
  --resource-group rg-lab-networking \
  --nsg-name nsg-web \
  --name AllowSSH \
  --priority 100 \
  --source-address-prefixes "YOUR_PUBLIC_IP" \
  --destination-port-ranges 22 \
  --protocol Tcp \
  --access Allow

# Add HTTP rule
az network nsg rule create \
  --resource-group rg-lab-networking \
  --nsg-name nsg-web \
  --name AllowHTTP \
  --priority 110 \
  --source-address-prefixes "*" \
  --destination-port-ranges 80 \
  --protocol Tcp \
  --access Allow

# Create VM in subnet with NSG
az vm create \
  --resource-group rg-lab-networking \
  --name vm-web-01 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --vnet-name vnet-lab \
  --subnet subnet-web \
  --nsg nsg-web \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard
```

---

### Exercise 3: Create an Availability Set with VMs
**Objective:** Deploy multiple VMs in an availability set for high availability

```bash
# Create availability set (3 FD, 5 UD)
az vm availability-set create \
  --resource-group rg-lab-networking \
  --name avset-web \
  --platform-fault-domain-count 3 \
  --platform-update-domain-count 5

# Create VM 1 in availability set
az vm create \
  --resource-group rg-lab-networking \
  --name vm-avset-01 \
  --availability-set avset-web \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --vnet-name vnet-lab \
  --subnet subnet-web \
  --admin-username azureuser \
  --generate-ssh-keys \
  --no-wait

# Create VM 2 in availability set
az vm create \
  --resource-group rg-lab-networking \
  --name vm-avset-02 \
  --availability-set avset-web \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --vnet-name vnet-lab \
  --subnet subnet-web \
  --admin-username azureuser \
  --generate-ssh-keys \
  --no-wait

# Verify availability set
az vm availability-set show \
  --resource-group rg-lab-networking \
  --name avset-web \
  --output table
```

---

### Exercise 4: Deploy VMs Across Availability Zones
**Objective:** Create VMs in different zones with a load balancer

```bash
# Create public IP for load balancer
az network public-ip create \
  --resource-group rg-lab-networking \
  --name pip-lb-web \
  --sku Standard \
  --zone 1 2 3

# Create load balancer
az network lb create \
  --resource-group rg-lab-networking \
  --name lb-web \
  --sku Standard \
  --public-ip-address pip-lb-web \
  --frontend-ip-name frontend-web \
  --backend-pool-name backend-web

# Create VM in Zone 1
az vm create \
  --resource-group rg-lab-networking \
  --name vm-zone1 \
  --zone 1 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --vnet-name vnet-lab \
  --subnet subnet-web \
  --admin-username azureuser \
  --generate-ssh-keys \
  --no-wait

# Create VM in Zone 2
az vm create \
  --resource-group rg-lab-networking \
  --name vm-zone2 \
  --zone 2 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --vnet-name vnet-lab \
  --subnet subnet-web \
  --admin-username azureuser \
  --generate-ssh-keys \
  --no-wait

# Create VM in Zone 3
az vm create \
  --resource-group rg-lab-networking \
  --name vm-zone3 \
  --zone 3 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --vnet-name vnet-lab \
  --subnet subnet-web \
  --admin-username azureuser \
  --generate-ssh-keys
```

---

### Exercise 5: Create a Virtual Machine Scale Set
**Objective:** Deploy a VMSS with autoscaling rules

```bash
# Create VMSS with 2 initial instances
az vmss create \
  --resource-group rg-lab-networking \
  --name vmss-web \
  --image Ubuntu2204 \
  --vm-sku Standard_B2s \
  --instance-count 2 \
  --vnet-name vnet-lab \
  --subnet subnet-web \
  --admin-username azureuser \
  --generate-ssh-keys \
  --upgrade-policy-mode Automatic \
  --zones 1 2 3

# Add autoscale rule - scale out when CPU > 70%
az monitor autoscale create \
  --resource-group rg-lab-networking \
  --resource vmss-web \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale-vmss \
  --min-count 2 \
  --max-count 10 \
  --count 2

az monitor autoscale rule create \
  --resource-group rg-lab-networking \
  --autoscale-name autoscale-vmss \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 2

# Add autoscale rule - scale in when CPU < 30%
az monitor autoscale rule create \
  --resource-group rg-lab-networking \
  --autoscale-name autoscale-vmss \
  --condition "Percentage CPU < 30 avg 10m" \
  --scale in 1

# View VMSS instances
az vmss list-instances \
  --resource-group rg-lab-networking \
  --name vmss-web \
  --output table
```

---

### Exercise 6: Attach Data Disks to a VM
**Objective:** Add and configure data disks

```bash
# Create a managed data disk
az disk create \
  --resource-group rg-lab-networking \
  --name disk-data-01 \
  --size-gb 128 \
  --sku Premium_LRS

# Attach disk to existing VM
az vm disk attach \
  --resource-group rg-lab-networking \
  --vm-name vm-web-01 \
  --name disk-data-01

# Or create and attach in one command
az vm disk attach \
  --resource-group rg-lab-networking \
  --vm-name vm-web-01 \
  --name disk-data-02 \
  --size-gb 64 \
  --sku StandardSSD_LRS \
  --new

# List disks attached to VM
az vm show \
  --resource-group rg-lab-networking \
  --name vm-web-01 \
  --query "storageProfile.dataDisks" \
  --output table
```

---

### Cleanup
```bash
# Delete all resources when done
az group delete --name rg-lab-networking --yes --no-wait
```

### Portal Exercises
1. **Create VM via Portal:** Walk through the wizard, observe all options
2. **Resize a VM:** Stop VM → Change size → Start VM
3. **Enable Boot Diagnostics:** VM → Settings → Boot diagnostics → Enable
4. **View Metrics:** VM → Monitoring → Metrics → Add CPU, Network, Disk charts
5. **Connect via Bastion:** Deploy Bastion → Connect without public IP