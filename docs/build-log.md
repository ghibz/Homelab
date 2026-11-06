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
  - (paste bridge mapping here)

## 2026-11-06 Step 3: OPNsense VM
- Created VM: 2 vCPU, 16 GB qcow2 (VirtIO), 5 virtio NICs: default (WAN), vlan10-users, vlan20-servers, vlan30-mgmt, vlan40-dmz
- Installed OPNsense (UFS); assigned WAN = NIC 1, LAN = NIC 2 (vlan10-users), OPT1-OPT3 = remaining NICs
- Set LAN 10.10.10.1/24 with DHCP 10.10.10.100-200; set OPT2 (vlan30-mgmt) 10.10.30.1/24
- Problem: 2048 MB RAM was not enough, OPNsense did not run properly
  - Fix: increased RAM to 3072 MB
  - Impact: less RAM left for DC1 and the client later, budget OPNsense at ~3 GB
- Test: ping -c 3 1.1.1.1 from the OPNsense shell (option 8)

## 2026-11-06 Step 4: Reach the OPNsense GUI from Mint
- Mint side of vlan30-mgmt: 10.10.30.2/24 (no DHCP); OPNsense OPT2: 10.10.30.1/24
- Problem: ping 10.10.30.1 from Mint failed
  - Cause: new OPNsense interface has no pass rules, so the firewall silently dropped the traffic (bridge, NIC mapping and IPs were fine)
  - Diagnosis: pfctl -d on the OPNsense shell -> ping worked immediately, which isolated the firewall
  - Fix: Firewall > Rules > OPT2 > + Add: Pass, Direction in, IPv4, Protocol any, Source any, Destination any; Save, then Apply changes
- Problem: the rule did not work right after saving it
  - Cause: rule was saved but not loaded into the live ruleset until applied/reloaded
  - Fix: Apply changes, then configctl filter reload from the OPNsense shell
- Lessons:
  - Direction is always "in" in this lab (rules are evaluated where traffic enters the interface)
  - Use configctl filter reload instead of pfctl -e to bring the firewall back (loads GUI rules and enables pf)
  - If locked out of the GUI, pfctl -d from the console is the escape hatch
- Commands used to diagnose:
  - Mint: ping 10.10.30.1; ip -br addr | grep 10.10.30; virsh net-list --all; bridge link; sudo tcpdump -i virbrX -n arp or icmp
  - OPNsense shell: ifconfig vtnetN; pfctl -d; pfctl -s info | head -3; pfctl -sr | grep -i "pass in"; configctl filter reload
  - OPNsense GUI: Firewall > Log Files > Live View (filter source 10.10.30.2); Interfaces > OPT2; Firewall > Rules > OPT2
  - NIC mapping check: compare the vlan30-mgmt NIC MAC in virt-manager with the MAC OPNsense shows for OPT2
- Test: GUI loads at https://10.10.30.1 from Mint
