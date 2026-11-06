## 2026-11-06 Step 1: KVM, libvirt, virt-manager
- Checked kvm-ok: KVM acceleration available
- Free disk: __ GB, RAM: 7.5 GB
- Installed qemu-kvm, libvirt, virt-manager; added user to libvirt/kvm groups
- Test: virsh list --all returned an empty table, no permission error

## 2026-11-06 Step 2: Virtual networks
- Chose Option 2: one isolated libvirt network per segment (simpler than OVS trunk tagging)
- Created vlan10-users, vlan20-servers, vlan30-mgmt, vlan40-dmz
- Problem: vlan30-mgmt IP reverted to 10.10.30.1 in virt-manager after Apply
  - Cause: network was running; virt-manager can't change a live network's IP
  - Fix: virsh net-destroy, virsh net-edit to 10.10.30.2, virsh net-start
- Problem: default network was inactive/no autostart
  - Fix: virsh net-start default; virsh net-autostart default
- Verified: all 5 networks active, autostart yes; bridge mapping below
