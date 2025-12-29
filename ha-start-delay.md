## Problem
HA-manager begins starting VMs too soon after a cluster reboot (e.g. full power loss). Ceph is not ready yet. Start of VM fails, but ha-manager does not retry the start, nor does it migrate the machine.

## Workaround
Use the systemd-stuff that is already in place, to delay the start of pve-ha-lrm.service a little. Most easily done by creating a file ```/etc/systemd/system/pve-ha-lrm.service.d/override.conf``` with a delay like this:  

    # Add small delay before HA starts to allow ceph to be ready
    
    [Service]
    ExecStartPre=/bin/sleep 20

Doing it using an override-file rather than putting it directly into the respective *.service file should prevent this from being lost on subsequent system upgrades (so far untested). Based on [gurubert's post on the proxmox forums](https://forum.proxmox.com/threads/delaying-vms-until-ceph-cephfs-is-mounted.120116/post-522290).
