# Lessons Learned

## What was the single most frustrating technical problem across all four scenarios, and what actually fixed it?

The most frustrating issue was VM networking. Every reboot caused the lab IP addresses to reset, and on one occasion Kali was accidentally assigned the same IP as the target. This made `ping` appear successful, since Kali was effectively talking to itself, while HTTP and SSH connections failed outright. Checking the interface configuration with `ip addr show` revealed the duplicate IP; removing it and reassigning the correct addresses restored connectivity. Since this happened more than once, checking the IP configuration became a standard first step before starting any scenario.

## Looking back, what would you do differently if you started this lab again from scratch?

I would spend more time automating the lab setup. Re-configuring IP addresses after every reboot slowed things down repeatedly, and I also learned the value of verifying prerequisites before collecting evidence, for example, confirming Wireshark was already capturing before launching an attack, rather than after. Doing these checks upfront would have prevented having to restart parts of some scenarios and would have made the overall workflow more efficient.

## What's one thing you understand about attacker behavior now that you didn't before this project?

I have a much better appreciation for how much information an attacker can gather before ever attempting exploitation. A single Nmap scan identified open services, exact software versions, and even pre-existing backdoors. That reinforced that reconnaissance isn't just information-gathering for its own sake, it's how attackers prioritize the easiest path into a system before they ever attempt to exploit anything.

## Did anything in this project change how you'd approach a real SOC alert differently than you would have before starting?

Yes. Before this project, I likely would have focused on the alert itself in isolation. Now I would always validate it against supporting evidence before drawing a conclusion. Throughout this lab, I correlated detections with packet captures, verified findings across multiple independent sources, and distinguished between confirmed positive and negative findings rather than assuming every alert meant a successful compromise. That shift toward an evidence-driven approach is the biggest change in how I'd handle a real investigation.
