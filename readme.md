# Custom Windows 10 Virtual Machine Image on Azure

## Overview
This project automates the creation of a custom Windows 10 virtual machine image on Microsoft Azure. The image includes specific software (Google Chrome, 7-Zip), the latest Windows updates, and is generalized using Sysprep for deployment in multiple instances.

## Technologies Used
- **Packer**: An open-source tool for creating machine images for various platforms.
- **Azure Resource Manager (ARM)**: Azure’s infrastructure as code service.
- **PowerShell**: Used for provisioning and configuring the system.
- **Windows Update**: Integrated using the `PSWindowsUpdate` module.
- **Sysprep**: Windows tool to generalize an image for reuse.

## Configuration File Explanation

### 1. Builders Section
This section specifies the configuration for Azure ARM builder to create the virtual machine.

#### Key Attributes
- `type`: Specifies Azure ARM as the provider.
- `client_id`, `client_secret`, `tenant_id`, `subscription_id`: Required credentials to authenticate to Azure.
- `managed_image_resource_group_name`: Specifies the resource group to store the created image.
- `managed_image_name`: Name of the custom image created.
- `os_type`: Specifies the operating system (Windows in this case).
- `image_publisher`, `image_offer`, `image_sku`: Specifies the base image (Windows 10 Pro, 22H2 version).
- `communicator`: Specifies `winrm` for Windows remote management.
- `vm_size`: Specifies the size of the virtual machine during the build process (e.g., `Standard_B1s`).

#### Azure Tags
Adds metadata (e.g., `dept` and `task`) for easier identification and management.

### 2. Provisioners Section
The provisioners automate software installation, system updates, and generalization.

#### Key Tasks
1. **Install PSWindowsUpdate Module**  
   Installs the `PSWindowsUpdate` PowerShell module to manage Windows updates programmatically.

2. **Install Windows Updates**  
   Uses `Install-WindowsUpdate` to install updates and reboots the system automatically.

3. **Install Google Chrome**  
   Downloads and installs Google Chrome silently using its official installer.

4. **Install 7-Zip**  
   Downloads and installs the 7-Zip compression tool silently.

5. **Run Sysprep**  
   Prepares the Windows image for deployment by generalizing it. Sysprep ensures the image can be used for creating unique instances by resetting user-specific information.

---

## PowerShell Commands in Detail
- **`Install-PackageProvider` and `Install-Module`**: Install PowerShell modules.
- **`Invoke-WebRequest`**: Downloads the installation files.
- **`Start-Process`**: Executes installation commands with specific arguments.

### Sysprep Workflow
1. Ensures necessary Azure agents (`RdAgent` and `WindowsAzureGuestAgent`) are running.
2. Executes Sysprep with `/oobe` and `/generalize` flags.
3. Waits until the system state changes to `IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE`.

### Overview of Sysprep
Sysprep (System Preparation Tool) is a Microsoft tool designed to prepare Windows operating systems for cloning, imaging, or deployment on multiple systems. It ensures that every instance created from the same image is unique by resetting system-specific data and configurations. Sysprep is essential for creating reusable, deployable Windows images.

###Key Functions of Sysprep
1. **Generalizing the Image:**
Removes all unique information like the security identifier (SID), user account settings, and other system-specific data. This allows the image to be deployed to multiple systems.

2. **Out-of-Box Experience (OOBE):**
Configures the system to boot into the Windows setup phase upon first boot, allowing administrators to specify new system settings.

3. **Audit Mode:**
Enables administrators to customize the image further without triggering OOBE.

### Sysprep Workflow in This Project
1. **Validate the Image State:**
Ensures that all required Azure agents, such as RdAgent and WindowsAzureGuestAgent, are running. These agents are crucial for Azure VM functionality.

2. **Run Sysprep Command:**
Executes the following command:
      ```
      sysprep /oobe /generalize /shutdown
      ```
        
          1.  /oobe: Configures the image to display the Out-of-Box Experience (OOBE) when booted.
          2.  /generalize: Removes system-specific data and resets the system's SID.
          3.  /shutdown: Shuts down the VM after the Sysprep process is complete.
    
3. **Verify Generalization State:**
Confirms that the system state changes to IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE before proceeding. This ensures the image is fully prepared for deployment.

---

## Process Flow

1. **Authentication and Setup**  
   Packer authenticates to Azure using provided credentials and initializes the build process in the specified resource group.

2. **VM Creation**  
   A virtual machine is created based on the specified Windows 10 base image (`win10-22h2-pro`).

3. **Provisioning**  
   Executes PowerShell scripts to install updates, software, and prepare the system.

4. **Generalization**  
   Runs Sysprep to prepare the VM for use as a reusable image.

5. **Image Creation**  
   The generalized image is captured and stored in the specified Azure resource group.

---

## Key Output

### Custom Windows 10 Image
- **Name**: `Cus_Win10__today`
- **Resource Group**: `Custom-Win10-Group`

#### Includes:
- Latest Windows updates.
- Google Chrome.
- 7-Zip.

---

## Benefits
- **Automation**: Automates the time-consuming process of preparing a custom image.
- **Consistency**: Ensures every deployment starts with an identical, pre-configured environment.
- **Scalability**: The custom image can be used to spin up multiple VMs, saving setup time.

---

## Prerequisites
- Azure account with a subscription.
- Packer installed locally.
- Necessary permissions to create resources in Azure.

---

## How to Run

1. **Install Packer**  
   Download and install from the [official Packer website](https://www.packer.io).

2. **Save the Configuration File**  
   Save the configuration file as `windows10.json`.

3. **Authenticate to Azure**  
   ```
   az login
   ```
   
4. **Validate the Packer File**
  ```
  packer validate windows10.json
  ```

5. **Build the Image**
  ```
  packer build windows10.json
  ```
