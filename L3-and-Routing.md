# IGP
## EIGRP

## OSPF
### OSPF Router IDs
1. Hard coded  under process
2. Highest IP of non-shut loopback which is not RID in any other OSPF process
3. Highest IP of non-shut physical iface which is not RID in any other OSPF process

### OSPF Message Types
![alt text](pics/OSPF01.png "OSPF Messages")

![alt text](pics/OSPF02.png "OSPF LSDB Exchange")

### OSPF Neighbour States
* Down
  * Initial state
  * Seen when a working adjacency is torn down
  * Or when a manually configured neighbour stops responding to Hellos
* Attempt
  * Valid only for NBMA
  * Neighbours are placed into this state and Hellos are sent normally
  * Placed back into Down if the neigh doesn't respond within the Dead Int
* Init
  * Placed in Init when a valid Hello has been received but the router **can't** see his RID in it
* 2-Way
  * Placed in 2-Way when a valid Hello has been received but the router **can** see his RID in it
* ExStart
  * Purpose - to establish a Master/Slave relationship
  * Empty DBD packets are sent to compare RIDs, determine M/S relation and agree on starting Seq Num
* Exchange
  * M/S relationship established
  * DBD packets are sent populated with LSAs Headers
* Loading
  * Neighbour is kept in this state while the LSAs are downloaded
* Full
  * Full adjacency
  * All missing/outdated LSAs have been downloaded
  
### Hello Process
* Cisco OSPF routers listen for `224.0.0.5` - All OSPF Routers
* Hellos are sourced from the primary IP address of the OSPF enabled iface
* Neigbour checks:
  * Authentication
  * Same subnet/mask
  * Same OSPF area
  * Same area type (regular, stub etc)
  * Must not have duplicate RIDs
  * OSPF timers must match
  * **MTU must be equal forthe DBD packets to be exchanged correctly. It would affect ExStart and 
  Exchange but routers can become neighbours up to and including 2-Way**

### DBD Exchange: Master/Slave Relationship
* Only a Master is allowed to send DBD packets on its own and increase the SeqNum
* Slaves can only reply
* Flags
  * Master (MS) - Set in all Master packers and cleared in all Slave packets
  * More (M) - Set to indicate that more DBD packets are coming
  * Init (I) - Set to indicate it's the first packet in an exchange
* LSA Exchange Process:
  * in ExStart
    * Both routers set all flags and send out an empty DBD packet
    * They compare RIDs and the lower RID changes its role to Slave and clears MS and I
  * they move to Exchange
    * The Master increases the SeqNum and sends out anther DBD (optionally with one or more LSA 
  Headers)
    * The Slave will reply with the same SeqNum (optionally advertising its LSA Headers)
    * The process keeps repeating until all LSAs have been advertised and the M flags are cleared
    * Routers compare received LSA Headers SeqNum with what they know
    * Newer SeqNum are prefered
  * in Loading
    * Link-State Req (LSR) packets are sent to get the info of the missing/outdated LSAs
    * Link-State Update (LSU) are sent with the requested LSA info
    * Link-State Ack (LSAck) are used for the reliable transport of LSA info

### DR/BDR Election
* `224.0.0.6` - All OSPF DR Routers
* All routers in the area for full adjacencies only with DR/BDR
* All other neigbours are stuck in a **2WAY/DROTHER** state
  * Neighbours - Routers sharing the same data link and exchanging Hellos with matching params
  * Adjacent - Neighbours which have completely exchanged DBDs and LSUs 
* DR Election
  * Routers with the OSPF priority 1-255 can participate in election (0 is ignored)
  * They collect all the other RIDs and priorities during the wait interval (set to the Dead Int)
  * When they have all info they choose the highest priority to be DR (if not elected already) and 
  second highest priority for BDR (if not elected already)
  * If all priorities are equal - highest RID becomes DR and second BDR
  * New routers don't preempt when coming online with a better priority
  
## ISIS

## BGP