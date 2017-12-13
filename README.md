MPDK (Memory Platform Development Kit)
================================

The Memory Platform Development Kit (MPDK) will provide proper SW stack of OS and applicaiton to enabling NVM media based solution devices (NVMe SSDs and future solution devices).  It will include user level drivers, kernel drivers, sample applications and tools.

The development kit currently includes:
* User Level NVMe driver (uNVMe driver).

# uNVMe driver 

uNVMe driver is an user space NVMe driver implemented as a library 
to which the sample applications can be linked with. 
It achieves high performance by moving all of the necessary drivers into user space
and operating in a polled mode instead of relying on interrupts,
which avoids kernel context switches and eliminates interrupt handling overhead.
Using the library, the sample will initialize the NVMe devices, submit and process the
I/Os directly to the NVMe devices.

 This release also includes the sample applications and tools for the uNVMe driver.
An application will be combined with the uNVMe driver into one process 
and takes the ownership of a device upon execution and controls the device.
An NVMe device can only be accessed by one application at any given time and
an application can access multiple devices simultaneously.
The sample applications can control the device using the provided APIs described below.

### In this readme:
* [System Requirements](#requirements)
* [Prerequisites](#prerequisites)
* [Source Code](#source)
* [Build](#build)
* [Advanced Build Options](#advanced)
* [Hugepages and Device Binding](#huge)
* [Sample Applications](#examples)
* [Application Programming Interface](#api)


<a id="requirements"></a>
## System Requirements
* Linux Kernel version must be 3.17 or above.
* Platform OS must be 64-bit Ubuntu 16.04 or above / Fedora-20 OS(FC20) or above.

<a id="prerequisites"></a>
## Prerequisites
* IOMMU feature has to be disabled in the UEFI / BIOS to use the uNVMe driver.
* The following packages must be installed.

In case of Ubuntu 16.04 or above, do the following:

	sudo apt-get install -y libaio-dev zlibc zlib1g zlib1g-dev graphviz \
			libcunit1 libcunit1-dev libnuma-dev \
			build-essential libtool autoconf automake xutils-dev git

In case of Fedora 20 or above, do the following:

	sudo dnf install -y gcc gcc-c++ make CUnit-devel libaio-devel libtool \
			autoconf automake patch xorg-x11-util-macros git \
			numactl-devel or libnuma-devel

* Executing the sample applications requires root privilege to access a NVMe device.

<a id="source"></a>
## Source Code
You can access to the source code as follows:

	git clone https://github.com/samsung/MPDK
	cd MPDK
	git submodule update --init

Then, you can see the directories and the following shows their brief explaination.
The `drivers` directory contains the uNVMe driver source code 
and the `app` directory contains the soruce code of some applications or tools which use the uNVMe driver. The `build` directory contains the binary files or some deliverables created by building.


<a id="build"></a>
## Build
You can build the whole as follows:

	cd $PATH_TO_MPDK/scripts
	./build.sh

<a id="advanced"></a>
## Advanced Build Options

Compile-time configurations are controlled by settings in the `config` file which is present in the MPDK directory.
These compile-time configurations can be enabled or disabled by setting the `y` or `n` against the specific feature.
The features include the following:

1. HotPlug - Feature to detect the hotplug insertion and hotplug removal of the NVMe devices. uNVMe driver initializes the devices upon insertion and de-initializes them upon the removal.

2. Congestion Handling - Feature to handle the congestion of the I/O queues. I/O more than the configured QD can be handled by the uNVMe driver for all the I/O queues.

3. Queue Sharing - Feature to enable the Queue sharing among the I/O threads. I/O Queues can be shared among multiple threads of execution that perform I/O to the NVMe devices.

4. SGL Support - SGL feature in the uNVMe driver can be enabled or disabled using this configuration setting. This setting can be disabled if the NVMe devices used do not support the SGL feature.

5. Metadata Support - Metadata feature in the uNVMe driver can be enabled or disabled using this configuration setting. This setting can be disabled if the NVMe devices used do not support the Metadata feature.

6. CMB Feature - CMB feature in the uNVMe driver can be enabled or disabled using this configuration setting. This setting can be disabled if the NVMe devices used do not support the CMB feature.

7. Security Commands - Security features in the uNVMe driver can be enabled or disabled using this configuration setting. This setting can be disabled if the NVMe devices used do not support the Security Send / Receive features.

<a id="huge"></a>
## Binding Device and Adjusting Hugepages

Since The linux kernel driver occupies the NVMe devices,
to unbound them from the kernel driver, execute as follows:

	cd $PATH_TO_MPDK/scripts
	./install_driver.sh

The `install_driver.sh` file allocates hugepages for the uNVMe driver buffer
and the default size is 1 GB.
The size can be changed as following example: 

	./uninstall_driver.sh
	export NR_HUGEPAGES=4096 
	./install_driver.sh

where the `NR_HUGPAGES` means the number of huge pages.
If huge page size is 2 MB, the 8 GB sized huge pages will be allocated for the uNVMe driver.
	
<a id="examples"></a>
## Sample Applications

The sample applications are present in the `$PATH_TO_MPDK/app` directory 
where `PATH_TO_MPDK` has to be set to the MPDK directory path.
The sample applications are compiled as part of the build process.
Execute the sample applications without any arguments to see the `HELP` output
or refer to the `README.md` file of each application in the directory.

<a id="api"></a>
## uNVMe driver's Application Programming Interfaces (API)

The following shows each uNVMe driver's API and its brief explanation.

	nvm_init()                                         - Initialize the uNVMe driver.
	nvm_exit()                                         - De-Initialize the uNVMe driver.
	nvm_ctlr_open()                                    - Open the NVMe Controller.
	nvm_ctlr_close()                                   - Close the NVMe Controller.
	nvm_ctlr_get_num_ns()                              - Get Number of Namespaces in the NVMe Controller.
	nvm_ctlr_read_reg()                                - Read bar registers of the NVMe Controller.
	nvm_get_driver_version()                           - Get the current driver version.
	nvm_ctlr_admin_cmd()                               - Issue Raw Admin commands to the NVMe Controller.
	nvm_ctlr_admin_vendor_unique_cmd()                 - Submit Admin vendor Unique commands to the NVMe Controller.			
	nvm_ctlr_get_smart_log()                           - Get SMART / Health Information Log from the NVMe Controller.
	nvm_ctlr_get_error_log()                           - Get Error Log / Logs from the NVMe Controller.
	nvm_ctlr_get_fw_slotinfo_log()                     - Get Firmware Slot Information Log from the NVMe Controller.
	nvm_ctlr_get_reservation_notification_log()        - Get Reservation notification Log from the NVMe Controller.
	nvm_ctlr_get_data()                                - Get the Identify Controller Data Structure from the NVMe Controller.
	nvm_ctlr_register_aer()                            - Register an Asynchronous Event Request Callback Function for the NVMe Controller.
	nvm_ctlr_format()                                  - Format the NVMe Controller.
	nvm_ctlr_firmware_download()                       - Download the Firmware to the NVMe Controller.
	nvm_ctlr_firmware_activate()                       - Activate the Firmware that was earlier downloaded to the NVMe Controller.
	nvm_ctlr_reset()                                   - Reset the NVMe Controller.
	nvm_ns_open()                                      - Open the Namespace in the NVMe Controller.
	nvm_ns_close()                                     - Close the Namespace.
	nvm_ns_get_data()                                  - Get the Identify Namespace Data Structure.
	nvm_ns_get_block_count()                           - Get the Number of Blocks(Sectors) in a Namespace.
	nvm_ns_get_block_size()                            - Get the Block Size(Sector Size) in a Namespace.
	nvm_ns_get_metadata_size()                         - Get the Metadata Size in a Namespace.
	nvm_ns_get_pi_type()                               - Get the Protection Information Type of a Namespace.
	nvm_ns_get_pi_location()                           - Get the Protection Information Location of a Namespace.
	nvm_ns_get_namespace_id()                          - Get the namespace id for the Namespace.
	nvm_ns_get_namespace_name()                        - Get the name of the namespace.
	nvm_ns_format()                                    - Format the Namespace.
	nvm_ns_get_smart_log()                             - Get SMART / Health Information Log for the particular namespace
	nvm_ns_sync_io_submit()                            - Synchronously perform IO from the Namespace.
	nvm_ns_vendor_unique_cmd()                         - Submit nvm vendor Unique commands to the Namespace.
	nvm_ns_raw_io_cmd()                                - Issue Raw IO command to the Namespace.
	nvm_ns_write_uncorrectable()                       - Synchronously mark a range of lba as invalid.
	nvm_ns_write_zeroes()                              - Synchronously write zero to a range of lba.
	nvm_ns_compare()                                   - Compare the data between lba range and compare buffer.
	nvm_ctlr_attach_ns()                               - Attach the namespace to the controller.
	nvm_ctlr_detach_ns()                               - Detach the namespace from the controller.
	nvm_ctlr_create_ns()                               - Create the namespace.
	nvm_ctlr_destroy_ns()                              - Destroy the namespace.
	nvm_ns_set_feature()                               - Set a feature in the NVMe Controller.
	nvm_ns_get_feature()                               - Get a feature of a Namespace in the NVMe Controller.
	nvm_ns_flush()                                     - Flush the Data in the Namespace to the Hardware.
	nvm_ns_reservation_acquire()                       - Acquire reservation on a Namespace if supported by the controller.
	nvm_ns_reservation_release()                       - Release a reservation for the particular Namespace.
	nvm_ns_reservation_report()                        - Get the reservation report for the particular Namespace.
	nvm_ns_reservation_register()                      - Register, unregister or replace a reservation key.
	nvm_ns_dsm_deallocate()                            - De-Allocate the LBAs in the Namespace.
	nvm_io_ses_init()                                  - Initialize an IO Session(to be invoked once per thread, associate one queue per session)
	nvm_io_ses_free()                                  - De-Initialize an IO Session.
	nvm_ns_async_io_submit()                           - Asynchronously perform IO from the Namespace. Note: Max data limit of 2MB can be transferred at a time.
	nvm_ns_async_io_get_completion()                   - Get a single AIO Completion.
	nvm_ns_async_io_poll_completion()                  - Poll for AIO Completion for the ios of specific session in case of Callback mode.
	nvm_ctlr_update_polling_duration()                 - Update controller polling duration. Note Poll duration should be between 0 & 1000 us.
	nvm_malloc()                                       - Allocate Physically Contiguous Memory.
	nvm_get_phys_addr()                                - Get the physical address for a virtual address.
	nvm_free()                                         - Free the Physically Contiguous Memory Allocated earlier.

For more information about the APIs, 
do the following

	cd $PATH_TO_MPDK/doc
	make

and refer to the doxygen documents in `html` directory.
