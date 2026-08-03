# VirtualBox Networking
I initially configured the VMs with different Internal Network names.
Although all three machines were running, they could not communicate.

This reinforced that the virtual network configuration is independent
of the operating system's IP configuration.

Then I configured them to the same NAT Network (Homelab-NAT), then they were able to communicate with each other.
