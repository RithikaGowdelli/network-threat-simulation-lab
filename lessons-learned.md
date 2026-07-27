# Lessons Learned

## The Most Frustrating Technical Problem

The most frustrating issue across this project was VM networking. Every reboot caused the lab IP addresses to disappear, and on one occasion I accidentally assigned Kali the same IP as the target machine. That made `ping` appear successful even though HTTP and SSH connections failed, because Kali was actually talking to itself rather than the target. Once I checked the interface configuration with `ip addr show`, I found the duplicate IP, removed it, and reassigned the correct addresses, which restored connectivity. Since this happened more than once, I built checking the IP configuration into my standard troubleshooting process before starting any scenario.

## What I'd Do Differently

I'd spend more time automating the lab setup. The repeated IP configuration after every reboot slowed things down, and I also learned to verify prerequisites before collecting evidence — for example, confirming Wireshark was already capturing before launching an attack, rather than after. Doing those checks up front would have prevented having to restart some scenarios and would have made the overall workflow far more efficient.

## What I Learned About Attacker Behavior

I have a much better appreciation for how much information attackers can gather before they ever exploit anything. A single Nmap scan identified open services, software versions, and even existing backdoors. That reinforced that reconnaissance isn't just collecting information — it's what attackers use to prioritize the easiest path into a system before attempting exploitation.

## How This Changed My Approach to a Real SOC Alert

Before this project, I would probably have focused only on the alert itself. Now I would always validate it with supporting evidence before making a decision. Throughout this lab, I correlated detections with packet captures, verified findings from multiple sources, and distinguished between positive and negative findings instead of assuming every alert meant a successful compromise. That gave me a much more evidence-driven approach to incident investigation.
