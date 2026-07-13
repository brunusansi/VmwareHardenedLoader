[!IMPORTANT]

## 🚀 Looking for the next-generation project?

**Development of VMwareHardenedLoader has effectively moved forward into a more complete and actively developed project:**

### 👉 **https://github.com/templarsco/pvgpu**

**pvgpu** expands on the ideas of VMwareHardenedLoader and aims to provide a more comprehensive virtualization stealth and GPU virtualization framework. While still under active development, it already includes a broader architecture and is intended to supersede many of the techniques used here.

If you're starting a new project or looking for the latest work, **we strongly recommend using pvgpu instead of VMwareHardenedLoader.**

This repository remains available for historical reference, research, and compatibility with existing implementations.

# VMwareHardenedLoader - Modernized Edition
VMware Hardened VM detection mitigation loader with multi-layer anti-detection system

**Supported OS:** Windows Vista, 7, 8, 8.1, 10, and **11** (x64 only)

Successfully defeats VM detection in VMProtect 3.x, Themida, Safengine, Pafish, and Al-Khaser.

## What it does

The VmLoader driver implements a comprehensive multi-layer anti-detection system:

### Core Features

1. **Enhanced SMBIOS/ACPI/FIRM Spoofing** 
   - Replaces VMware signatures with realistic hardware identifiers (Dell, HP, Lenovo)
   - Runtime patching of SystemFirmwareTable
   - Removes "VMware", "Virtual", "VMWARE" signatures

2. **CPUID Masking**
   - Detection and logging of hypervisor CPUID signatures
   - Guidance for VMX configuration to hide hypervisor leaves (0x40000000-0x40000010)
   - Hypervisor present bit (ECX[31]) masking strategy

3. **Timing Normalization**
   - RDTSC/RDTSCP timing analysis
   - VM timing signature detection
   - Recommendations for timing normalization via VMX settings

4. **VMware Backdoor Blocking**
   - Comprehensive backdoor port blocking guidance (0x5658, 0x5659)
   - VMware Tools detection warnings
   - Guest-to-host communication mitigation

5. **Windows 11 Support**
   - Full compatibility with Windows 11 (builds 22000+)
   - Kernel structure offset updates
   - VBS (Virtualization-Based Security) awareness
   - KDP (Kernel Data Protection) compatibility

6. **Additional Anti-Detection Modules**
   - MSR interception strategy
   - PCI device spoofing framework
   - GPU masking framework
   - Disk serial spoofing framework
   - VMware process hiding framework
   - MAC address spoofing framework

## Legitimate Use Cases

This software is intended for legitimate purposes only:

- **Security Research**: Testing and developing VM detection techniques
- **Malware Analysis**: Creating isolated analysis environments
- **Privacy-Focused Computing**: Preventing fingerprinting in virtualized environments
- **Red Team Operations**: Authorized security assessments and penetration testing
- **Software Development**: Testing software behavior in various environments

**Disclaimer**: Users are responsible for ensuring their use complies with applicable laws and regulations.

## Build

### Requirements
- Visual Studio 2015 / 2017 / 2019 / 2022
- [Windows Driver Kit 10](https://docs.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk)

### Build Steps
1. Open `VmLoader.sln` with Visual Studio
2. Build VmLoader as **x64/Release** (x86 is not supported)
3. Test-sign `bin/vmloader.sys` for test-sign mode:
   ```cmd
   signtool sign /f your_test_cert.pfx /p password /t http://timestamp.digicert.com bin/vmloader.sys
   ```

# Installation

## Critical Warning

**DO NOT INSTALL VMWARE TOOLS!** It will expose VM presence through multiple channels.

Use remote access alternatives:
- TeamViewer
- AnyDesk  
- Windows Remote Desktop (mstsc)
- VNC viewer
- Chrome Remote Desktop

## 1st Step: Add following settings into .vmx

**Essential VMX Configuration** - Add these settings to your `.vmx` file:

### Basic Hypervisor Hiding
```
hypervisor.cpuid.v0 = "FALSE"
board-id.reflectHost = "TRUE"
hw.model.reflectHost = "TRUE"
serialNumber.reflectHost = "TRUE"
smbios.reflectHost = "TRUE"
SMBIOS.noOEMStrings = "TRUE"
```

### Backdoor Communication Blocking
```
isolation.tools.getPtrLocation.disable = "TRUE"
isolation.tools.setPtrLocation.disable = "TRUE"
isolation.tools.setVersion.disable = "TRUE"
isolation.tools.getVersion.disable = "TRUE"
monitor_control.restrict_backdoor = "TRUE"
```

### VMware Tools Features (Disable All)
```
isolation.tools.ghi.autologon.disable = "TRUE"
isolation.tools.hgfs.disable = "TRUE"
isolation.tools.memSchedFakeSampleStats.disable = "TRUE"
isolation.tools.getCreds.disable = "TRUE"
isolation.tools.ghi.trayicon.disable = "TRUE"
isolation.tools.unity.disable = "TRUE"
isolation.tools.unityInterlockOperation.disable = "TRUE"
isolation.tools.unity.push.update.disable = "TRUE"
isolation.tools.unity.taskbar.disable = "TRUE"
isolation.tools.unityActive.disable = "TRUE"
isolation.tools.vmxDnDVersionGet.disable = "TRUE"
isolation.tools.guestDnDVersionSet.disable = "TRUE"
isolation.tools.dnd.disable = "TRUE"
isolation.tools.copy.disable = "TRUE"
isolation.tools.paste.disable = "TRUE"
```

### Timing Normalization
```
monitor_control.disable_directexec = "TRUE"
monitor_control.disable_chksimd = "TRUE"
monitor_control.disable_ntreloc = "TRUE"
monitor_control.disable_selfmod = "TRUE"
monitor_control.disable_reloc = "TRUE"
monitor_control.disable_btinout = "TRUE"
monitor_control.disable_btmemspace = "TRUE"
monitor_control.disable_btpriv = "TRUE"
monitor_control.disable_btseg = "TRUE"
```

### Virtual RDTSC (Optional but Recommended)
```
monitor_control.virtual_rdtsc = "TRUE"
monitor_control.virtual_rdtsc_offset = "0"
```

### Disk Configuration (SCSI)

If you have a SCSI virtual disk at scsi0 slot (first slot) as your system drive:

```
scsi0:0.productID = "Samsung SSD 860"
scsi0:0.vendorID = "Samsung"
```

Example configuration:
```
scsi0:0.productID = "SSD 860 EVO"
scsi0:0.vendorID = "Samsung"
```

## 2nd Step: Modify MAC address

Change the guest's MAC address to avoid VMware OUI detection:

**Avoid these VMware OUIs:**
```
00:05:69  # VMware, Inc.
00:0C:29  # VMware, Inc.
00:1C:14  # VMware, Inc.
00:50:56  # VMware, Inc.
```

**Recommended approach** - Add to `.vmx` file:
```
ethernet0.address = "00:1B:21:XX:XX:XX"  # Intel OUI
```

Or use other common manufacturer OUIs:
```
00:1B:21  # Intel Corporate
00:50:B6  # Dell
B8:27:EB  # Raspberry Pi (less suspicious in labs)
```

Example:
```
ethernet0.address = "00:1B:21:3A:4F:C2"
ethernet0.addressType = "static"
```

![MAC Address Configuration](https://github.com/hzqst/VmwareHardenedLoader/raw/master/img/4.png)

## 3rd Step: Run install.bat in VM guest

1. Copy `bin/vmloader.sys` to your VM guest
2. Run `install.bat` as **Administrator**
3. Check output:
   - If errors occur, use [DebugView](https://docs.microsoft.com/en-us/sysinternals/downloads/debugview) to capture kernel debug output
   - Post issues with DebugView output and your `ntoskrnl.exe` if needed
4. If no errors, the driver is successfully loaded!

## Verification

After installation, you can verify the driver is working by:

1. **Check DbgView output** - Look for initialization messages:
   ```
   [VmLoader] Windows Build: 22000
   [VmLoader] Operating System: Windows 11
   [VmLoader] [OK] CPUID Masking initialized
   [VmLoader] [OK] Timing Normalization initialized
   ...
   [VmLoader] VmLoader initialized successfully!
   [VmLoader] Active modules: 10/10
   ```

2. **Run detection tools**:
   - VMProtect 3.x packed executables (anti-vm enabled)
   - Pafish
   - Al-Khaser
   - Custom CPUID/timing checks

## Showcase

VMware guest Windows 8.1/10/11 x64 with VMProtect 3.2 packed program (anti-vm option enabled):

**Before:**
![Before VmLoader](https://github.com/hzqst/VmwareHardenedLoader/raw/master/img/1.png)

**Detected Signatures:**
![Signatures](https://github.com/hzqst/VmwareHardenedLoader/raw/master/img/2.png)

**After VmLoader:**
![After VmLoader](https://github.com/hzqst/VmwareHardenedLoader/raw/master/img/3.png)

## Architecture

### Module Structure

```
VmLoader/
├── main.cpp                    # Core driver, SMBIOS/ACPI hooking, module orchestration
├── Config.h                    # Global configuration and feature flags
├── Utils.h                     # Shared utility functions
├── CpuidMasking.cpp/h         # CPUID instruction spoofing and detection
├── TimingNormalization.cpp/h  # RDTSC/RDTSCP timing analysis
├── BackdoorBlocking.cpp/h     # VMware backdoor port blocking guidance
├── MsrInterception.cpp/h      # MSR access interception framework
├── PciSpoofing.cpp/h          # PCI device ID spoofing framework
├── GpuMasking.cpp/h           # GPU device masking framework
├── DiskSpoofing.cpp/h         # Disk serial spoofing framework
├── ProcessHider.cpp/h         # VMware process hiding framework
└── MacSpoofing.cpp/h          # MAC address spoofing framework
```

### Detection Vectors Covered

| Detection Method | Mitigation Status | Implementation |
|-----------------|-------------------|----------------|
| SMBIOS Strings | ✅ Active | Enhanced runtime patching with realistic values |
| ACPI Tables | ✅ Active | Runtime table patching |
| CPUID Leaves | ✅ Guided | VMX configuration + detection logging |
| Timing Attacks | ✅ Guided | Analysis + VMX configuration recommendations |
| Backdoor Ports | ✅ Guided | Comprehensive VMX configuration |
| MAC Address | ✅ Manual | Configuration guidance |
| PCI Device IDs | 🔶 Framework | Module framework ready |
| GPU Detection | 🔶 Framework | Module framework ready |
| Disk Serials | 🔶 Framework | Module framework ready |
| Process Names | 🔶 Framework | Module framework ready |
| MSR Access | 🔶 Framework | Module framework ready |

**Legend:**
- ✅ Active: Fully implemented and active
- ✅ Guided: Provides guidance and VMX configuration
- ✅ Manual: Requires manual configuration
- 🔶 Framework: Infrastructure ready for future implementation

## Technical Notes

### Limitations

Some anti-detection features require hypervisor-level capabilities:

1. **Direct CPUID Interception**: Requires VT-x EPT hooking or exception handlers
2. **RDTSC/RDTSCP Interception**: Requires hypervisor or CPU microcode-level access
3. **I/O Port Filtering**: Requires hypervisor I/O bitmap configuration
4. **MSR Interception**: Requires hypervisor MSR bitmap configuration

**Current Approach**: The driver provides comprehensive detection, analysis, and VMX configuration guidance. Combined with proper VMX settings, this creates an effective multi-layer defense.

**Future Enhancement**: Could be extended with:
- Hypervisor-based interception (Type-1 thin hypervisor)
- Binary patching of detection code (more invasive)
- Memory scanning and patching of known detection patterns

### Windows 11 Compatibility

The modernized driver fully supports Windows 11:
- Build detection (22000+)
- Compatible with VBS (Virtualization-Based Security)
- Aware of KDP (Kernel Data Protection)
- Updated kernel structure offsets
- Works with Secure Boot in test-signing mode

## Troubleshooting

### Common Issues

**Issue**: Driver fails to load
- **Solution**: Ensure test-signing is enabled: `bcdedit /set testsigning on`
- Reboot after enabling test-signing
- Verify driver is properly signed

**Issue**: Detection still occurs
- **Solution**: Verify ALL VMX settings are applied
- Ensure VMware Tools is NOT installed
- Check MAC address is not using VMware OUI
- Review DbgView output for module initialization status

**Issue**: System crash or BSOD
- **Solution**: Incompatible Windows version or kernel structure mismatch
- Check DbgView output before crash
- Report issue with Windows version and ntoskrnl.exe

**Issue**: "ExpFirmwareTableResource not found" error
- **Solution**: Kernel structure offsets may have changed
- Report Windows build number and ntoskrnl.exe for analysis

## License

This software is released under the MIT License, see LICENSE.

## Credits and Attribution

- Original VmwareHardenedLoader: [hzqst](https://github.com/hzqst/VmwareHardenedLoader)
- Some utility procedures from [HyperPlatform](https://github.com/tandasat/HyperPlatform)
- Disassembly powered by [Capstone Engine](https://github.com/aquynh/capstone)

## Security Advisory

This tool is provided for educational and legitimate security research purposes. Misuse of this software for:
- Bypassing software license protection
- Evading malware analysis systems
- Unauthorized access to systems
- Other malicious purposes

...is strictly prohibited and may be illegal in your jurisdiction.

**Responsible Use**: Always obtain proper authorization before using this tool in any professional capacity.

## Future Roadmap

Potential enhancements for future versions:

- [ ] Full hypervisor-based CPUID/RDTSC interception
- [ ] Runtime binary patching of detection code
- [ ] Advanced GPU/DXGI spoofing with driver modification guidance  
- [ ] Automated detection tool testing framework
- [ ] Configuration GUI for easier setup
- [ ] Support for other hypervisors (VirtualBox, Hyper-V)
- [ ] Enhanced process hiding with kernel callback manipulation
- [ ] Network adapter spoofing beyond MAC address

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Test thoroughly on multiple Windows versions
4. Submit a pull request with detailed description

For security vulnerabilities, please report privately to the maintainers.

## Disclaimer

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED. USE AT YOUR OWN RISK. THE AUTHORS ARE NOT RESPONSIBLE FOR ANY DAMAGE OR LEGAL CONSEQUENCES ARISING FROM THE USE OF THIS SOFTWARE.

---

**Note**: This is a research tool. Always comply with applicable laws, regulations, and terms of service.
