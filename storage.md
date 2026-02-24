# Azure Storage Account

## Storage Services
- **Blob** - Object storage for unstructured data
- **File** - Managed file shares (SMB/NFS)
- **Table** - NoSQL key-value store
- **Queue** - Message queuing service

## Access Methods
- Access Key
- SAS (Shared Access Signature)
- Stored Access Policy
- Azure AD (Entra ID)

---

## Redundancy Options

### LRS (Locally Redundant Storage)
- 3 copies in the same data center
- Lowest cost option
- **Protects against:** server rack and drive failures
- **Does NOT protect against:** data center disasters (fire, flood, etc.)
- **Durability:** 99.999999999% (11 nines) over a year
- **Use case:** Non-critical data, data that can be easily recreated

### ZRS (Zone-Redundant Storage)
- 3 copies across 3 availability zones in the same region
- Each zone is a separate physical location with independent power, cooling, networking
- **Protects against:** data center-level failures
- **Does NOT protect against:** regional disasters
- **Durability:** 99.9999999999% (12 nines) over a year
- **Use case:** High availability apps, data must remain in one region for compliance

### GRS (Geo-Redundant Storage)
- 6 copies total: 3 in primary region (LRS) + 3 in secondary region (LRS)
- Secondary region is an Azure paired region (hundreds of miles away)
- Asynchronous replication to secondary (RPO ~15 minutes)
- **Protects against:** regional outages and disasters
- Secondary data is NOT readable unless failover occurs
- **Durability:** 99.99999999999999% (16 nines) over a year
- **Use case:** Business-critical data requiring disaster recovery

### GZRS (Geo-Zone-Redundant Storage)
- 6 copies total: 3 across zones in primary (ZRS) + 3 in secondary region (LRS)
- Combines ZRS high availability with GRS geo-replication
- Best protection against both zone and regional failures
- **Durability:** 99.99999999999999% (16 nines) over a year
- **Use case:** Mission-critical data requiring maximum durability and availability

### RA-GRS (Read-Access Geo-Redundant Storage)
- Same as GRS but secondary region data is always readable
- Read-only access to secondary without needing failover
- **Use case:** Read-heavy apps that need geo-distributed read access, reporting

### RA-GZRS (Read-Access Geo-Zone-Redundant Storage)
- Same as GZRS but secondary region data is always readable
- Highest availability and durability option
- **Use case:** Mission-critical apps needing maximum redundancy + read access

---

## Access Tiers (Blob Storage)

### Hot Tier
- For frequently accessed data
- Highest storage cost, lowest access cost
- No minimum storage duration
- **Use case:** Active data, websites, streaming, analytics

### Cool Tier
- For infrequently accessed data (stored at least 30 days)
- Lower storage cost, higher access cost than Hot
- **Minimum storage duration:** 30 days (early deletion fee applies)
- **Use case:** Short-term backup, older data still occasionally accessed

### Cold Tier
- For rarely accessed data (stored at least 90 days)
- Lower storage cost than Cool, higher access cost
- **Minimum storage duration:** 90 days (early deletion fee applies)
- **Use case:** Long-term backup, disaster recovery data

### Archive Tier
- For rarely accessed data (stored at least 180 days)
- Lowest storage cost, highest access cost
- Data is offline - must be rehydrated before access
- **Rehydration options:** Standard (up to 15 hours), High priority (under 1 hour)
- **Minimum storage duration:** 180 days (early deletion fee applies)
- **Use case:** Long-term archive, compliance data, historical records

### Key Points
- Tiers can be set at account level (default) or individual blob level
- Lifecycle management policies can auto-move blobs between tiers
- Only Hot and Cool available for general-purpose v2 account default
- Archive is blob-level only (cannot be default account tier)

---

## Performance

### Standard
- Uses HDD (Hard Disk Drives)
- Lower cost, good for most workloads
- Supports all storage services: Blob, File, Table, Queue
- Supports all redundancy options (LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS)
- Supports all access tiers (Hot, Cool, Cold, Archive)
- **Max IOPS:** ~20,000 per account
- **Use case:** General purpose, backup, infrequently accessed data

### Premium
- Uses SSD (Solid State Drives)
- Higher cost, low-latency performance
- Does NOT support all redundancy options (only LRS and ZRS)
- Does NOT support access tiers (always hot-equivalent)
- **Three types based on account kind:**
  - Premium Block Blobs: High transaction rates, low latency
  - Premium File Shares: Enterprise file shares, SMB/NFS
  - Premium Page Blobs: VM disks, random read/write
- **Max IOPS:** ~100,000+ per account
- **Use case:** Databases, analytics, high-performance apps, VM disks

---

## Account Kind

### Storage (legacy)
- Original storage account type (v1)
- Supports Blob and File only
- No access tiers support
- ⚠️ Not recommended for new deployments
- Microsoft recommends upgrading to General-purpose v2

### StorageV2 (General-purpose v2) ✅ RECOMMENDED
- Current standard account type
- Supports all storage services: Blob, File, Table, Queue
- Supports all access tiers (Hot, Cool, Cold, Archive)
- Supports all redundancy options
- Supports both Standard and Premium performance
- Lowest per-GB pricing
- **Use case:** Most scenarios - default choice for new accounts

### BlobStorage (legacy)
- Blob-only storage account
- Supports block blobs and append blobs only
- Supports access tiers
- Superseded by General-purpose v2
- ⚠️ Not recommended for new deployments

### BlockBlobStorage (Premium)
- Premium performance for block blobs and append blobs
- SSD-based, low latency
- Only supports LRS and ZRS redundancy
- No access tiers (always premium/hot)
- **Use case:** High transaction workloads, IoT, analytics

### FileStorage (Premium)
- Premium performance for file shares only
- SSD-based, low latency
- Supports SMB and NFS protocols
- Only supports LRS and ZRS redundancy
- **Use case:** Enterprise file shares, databases, HPC

### Key Points
- Always choose StorageV2 unless you need premium performance
- Premium accounts are specialized (Block Blob, File, or Page Blob)
- Account kind and performance cannot be changed after creation
- Legacy accounts (v1, BlobStorage) can be upgraded to v2
