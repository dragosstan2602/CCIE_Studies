Case study - bgp deterministic-med AND bgp always-compare-med
For example, consider the following routes:
```
    entry1: ASPATH 1, MED 100, internal, IGP metric to NEXT_HOP 10
    entry2: ASPATH 2, MED 150, internal, IGP metric to NEXT_HOP 5
    entry3: ASPATH 1, MED 200, external
```
The order in which the BGP routes were received is entry3, entry2, and entry1 (entry3 is the oldest 
entry in the BGP table and entry1 is the newest one).

A BGP router with bgp deterministic-med disabled chooses entry2 over entry1, due to a lower IGP 
metric to reach the NEXT_HOP (MED was not used in this decision because entry1 and entry2 are from 
two different ASs). It then prefers entry3 over entry2 because it's external. However, entry3 has a 
higher MED than entry1.

If bgp deterministic-med is enabled, routes from the same AS are grouped together, and the best 
entries of each group are compared. In the given example, there are two ASs, AS 1 and AS 2.
```
    Group 1:  entry1: ASPATH 1, MED 100, internal, IGP metric to NEXT_HOP 10
              entry3: ASPATH 1, MED 200, external
    Group 2:  entry2: ASPATH 2, MED 150, internal, IGP metric to NEXT_HOP 5
```
In Group 1, the best path is entry1 because of the lower MED (MED is used in this decision since 
the paths are from the same AS). In Group 2, there is only one entry (entry2). The best path then 
is determined by comparing the winners of each group (MED is not used in this comparison by default 
because the winners of each group are from different ASs - enabling bgp always-compare-med changes 
this default behavior). Now, when comparing entry1 (the winner from Group 1) and entry2 (the winner 
from Group 2), entry2 will be the winner since it has the better IGP metric to the next hop.

If bgp always-compare-med was also enabled then comparing entry1 (the winner from Group 1) and 
entry 2 (the winner from Group 2), entry 1 will be the winner because of lower MED.

Cisco recommends enabling bgp deterministic-med in all new network deployments. In addition, if 
bgp always-compare-med is enabled, BGP MED decisions are always deterministic. 
