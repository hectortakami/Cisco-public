# Cisco Unified Computing System

**Stateless Computing:** Through the use of **Service Profiles** abstracts the hardware resources from the machine logic determining the characteristics that previously were contained in the physicial host like MAC, UUIDs, Boot Order, Firmware, WWNN and Policies. Allows to transfer the host logic and configurations from one server blade to another in case of failure.

**UCS Manager:** Runs over the **Fabric Interconnects**, it´s the manager of the operation and can manage up to 20 chasis (160 blades).

**Data Management Engine (DME):** Manage sub-processes of the FI. Centralizes the logs and information from the Application Gateways (AG) that are agents from the system that report every detail part from the FI-UCSM.

# Log Analysis

Software: Agent Ransack (Windows Software)

Log Recollection: https://www.cisco.com/c/en/us/support/docs/servers-unified-computing/ucs-infrastructure-ucs-manager-software/211587-Visual-Guide-to-collect-UCS-Tech-Support.html

## Error file sources

**UCSM LOGS**

- `sam_techsupportinfo` (FI with the highest file size): Retreive PIDs
- `tech_support.frupids` (Only on UCS-C standalone): Retreive PIDs
- `sw_techsupportinfo` FI status, interface status and health

**SERVER LOGS**

- **System Failure:** _Quick server review logs at `var/log/sel/log` file and `obfl.log` for more detail_
- **Health Status (disks/RAID)** _Storage server status at `storage-data` file_
- **Health Status (DIMM memory)** _Memory server status at `DIMMBL`, `DIMM-BL_Status` or `MrcOut` file_
- **Health Status (sensors)**: _Server (standalone), firmware & sensors info and status at `tmp/tech_support` or `XXXX_TechSupport` file_
