# Castle Rysen - Scaling OSPF to Larger Networks (Lab)

This lab demonstrates the practical application of scaling OSPF for a growing network, as outlined in the "Castle Rysen" theory. By breaking OSPF into areas, summarising routes, and centralising internet access, we design a network that is both efficient and scalable.

## Lab Topology
*Refer to the provided Packet Tracer diagrams for the visual layout.*

The network consists of:
*   **Central Location (Fallout Shelter):** Controlled by `FO-RT01`. This router will act as the hub, connecting to the ISP and other branches.
*   **Branch Location (District Shop #1 / Cafe):** Controlled by `cafe01-RT01`.
*   **ISP Router:** Provides public internet access (`216.0.5.1`).

---

## Step 1: Establishing Centralised Internet Connectivity

**Why we are doing this:** Instead of providing separate, expensive internet connections to each individual cafe branch, internet access is centralised at the Fallout Shelter. This saves money and makes the network easier to secure and manage.

### 1. Configure the WAN Interface on `FO-RT01`
*   **What we are changing:** We are enabling the interface connected to the ISP router and assigning it a public-facing IP address.
*   **Command:**
    ```text
    FO-RT01(config)#int fa0/1
    FO-RT01(config-if)#no shutdown 
    FO-RT01(config-if)#ip address 216.0.5.2 255.255.255.0
    ```
*   **Result:**
    ```text
    %LINK-5-CHANGED: Interface FastEthernet0/1, changed state to up
    ```

### 2. Verify Connectivity to the ISP
*   **Command:**
    ```text
    FO-RT01#ping 216.0.5.1
    ```
*   **Result:**
    ```text
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 216.0.5.1, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
    ```

### 3. Configure the Gateway of Last Resort (Default Route)
*   **What we are changing:** We are creating a static default route on the hub router pointing to the ISP.
*   **Why we are doing this:** This tells the `FO-RT01` router, "If you don't have a specific route for an IP address (like a website), send the traffic to the ISP."
*   **Command:**
    ```text
    FO-RT01(config)#ip route 0.0.0.0 0.0.0.0 216.0.5.1
    ```
*   **Result:** Verifying the routing table shows the static default route (`S*`).
    ```text
    FO-RT01#show ip route
    ...
    Gateway of last resort is 216.0.5.1 to network 0.0.0.0
    ...
    S*   0.0.0.0/0 [1/0] via 216.0.5.1
    ```

---

## Step 2: Implementing Multi-Area OSPF

**Why we are doing this:** In a growing network, keeping every router in a single area (Area 0) becomes a liability. Every router would need to know the full map of the network, causing massive memory usage and CPU spikes whenever a link flaps. Splitting the network into multiple areas isolates these updates.

### 1. Intentional Disruption: Changing Area Assignment on `FO-RT01`
*   **What we are changing:** We are moving the serial link (which connects to the cafe) out of **Area 0** and into **Area 1**.
*   **Command:**
    ```text
    FO-RT01(config)#router ospf 1
    FO-RT01(config-router)#no network 172.16.0.1 0.0.0.0 area 0
    FO-RT01(config-router)#network 172.16.0.1 0.0.0.0 area 1
    ```
*   **Result:** As expected, the OSPF adjacency immediately drops. 
    ```text
    %OSPF-5-ADJCHG: Process 1, Nbr 216.0.5.2 on Serial0/0/0 from FULL to DOWN, Neighbor Down: Interface down or detached
    %OSPF-4-ERRRCV: Received invalid packet: mismatch area ID, from backbone area must be virtual-link but not found from 172.16.0.1, Serial0/0/0
    ```
*   **Why did this happen?** OSPF neighbors *must* agree on the area they are in. Since `cafe01-RT01` is still configured for Area 0 on that link, it rejects the mismatch.

### 2. Restoring Adjacency: Updating `cafe01-RT01`
*   **What we are changing:** We are updating `cafe01-RT01` to match the new multi-area design, placing its networks into Area 1.
*   **Command:**
    ```text
    cafe01-RT01(config)#router ospf 1
    cafe01-RT01(config-router)#no network 10.0.18.1 0.0.0.0 area 0
    cafe01-RT01(config-router)#no network 172.16.0.2 0.0.0.0 area 0
    cafe01-RT01(config-router)#network 10.0.18.1 0.0.0.0 area 1
    cafe01-RT01(config-router)#network 172.16.0.2 0.0.0.0 area 1
    ```
*   **Result:** The adjacency forms again because the areas now match.
    ```text
    %OSPF-5-ADJCHG: Process 1, Nbr 1.1.1.1 on Serial0/1/0 from LOADING to FULL, Loading Done
    ```
*   **Outcome:** `FO-RT01` now has interfaces in Area 0 and Area 1. It is officially functioning as an **Area Border Router (ABR)**.

---

## Step 3: Route Summarization (Shrinking the Routing Table)

**Why we are doing this:** Before summarization, the branch router (`cafe01-RT01`) has to memorize every individual tiny subnet located at the Fallout Shelter. By summarizing, we hide the topology details of Area 0 from Area 1, shrinking the routing table and saving RAM/CPU.

### 1. View the Routing Table BEFORE Summarization
*   **Result:** `cafe01-RT01` sees four separate `/25` subnets. The `O IA` tag means they are OSPF Inter-Area routes (coming from a different area).
    ```text
    cafe01-RT01#show ip route
    ...
    O IA    10.0.16.0/25 [110/65] via 172.16.0.1, 00:31:31, Serial0/1/0
    O IA    10.0.16.128/25 [110/65] via 172.16.0.1, 00:31:31, Serial0/1/0
    O IA    10.0.17.0/25 [110/65] via 172.16.0.1, 00:31:31, Serial0/1/0
    O IA    10.0.17.128/25 [110/65] via 172.16.0.1, 00:31:31, Serial0/1/0
    ```

### 2. Configure Summarization on the ABR (`FO-RT01`)
*   **What we are changing:** We configure the ABR to collapse those four contiguous `/25` subnets into a single `/23` summary block before advertising it to Area 1.
*   **Command:**
    ```text
    FO-RT01(config)#router ospf 1
    FO-RT01(config-router)#area 0 range 10.0.16.0 255.255.254.0
    ```

### 3. Verify the Cleaned-Up Routing Table
*   **Result:** Checking `cafe01-RT01` again, the multiple routes have disappeared and are replaced by one clean summary route.
    ```text
    cafe01-RT01#show ip route
    ...
    O IA    10.0.16.0/23 [110/65] via 172.16.0.1, 00:00:02, Serial0/1/0
    ```
> [!TIP]
> **Real World Tip:** This is why contiguous IP addressing is critical. Because the original subnets were neatly grouped in sequence (10.0.16.x and 10.0.17.x), summarizing them into a single `/23` was mathematically possible.

---

## Step 4: Injecting a Default Route (The ASBR)

**Why we are doing this:** We created a default route on `FO-RT01` pointing to the internet in Step 1, but the branch router (`cafe01-RT01`) still doesn't know how to reach the outside world. Instead of manually typing static routes on every branch router across the company, we will let OSPF dynamically hand out the default route.

### 1. Inject the Default Route
*   **What we are changing:** We tell `FO-RT01` to advertise its static default route into the OSPF database. 
*   **Command:**
    ```text
    FO-RT01(config)#router ospf 1
    FO-RT01(config-router)#default-information originate 
    ```
*   **Outcome:** Because `FO-RT01` is bringing in a route from *outside* of OSPF, it is now acting as an **Autonomous System Boundary Router (ASBR)**.

### 2. Verify Internet Reachability at the Branch
*   **Result:** Looking at `cafe01-RT01`'s routing table, we see the Gateway of Last Resort has been dynamically learned via OSPF. The `E2` tag signifies an OSPF External Type 2 route.
    ```text
    cafe01-RT01#show ip route
    ...
    Gateway of last resort is 172.16.0.1 to network 0.0.0.0
    ...
    O*E2 0.0.0.0/0 [110/1] via 172.16.0.1, 00:00:01, Serial0/1/0
    ```

### 3. Verify the OSPF Database (Type 5 LSA)
*   **Result:** If we peek into the OSPF database, we can see exactly *how* that route arrived. It was delivered as a **Type 5 AS External LSA** generated by the ASBR.
    ```text
    cafe01-RT01#show ip ospf database 
    ...
                    Type-5 AS External Link States
    Link ID         ADV Router      Age         Seq#       Checksum Tag
    0.0.0.0         1.1.1.1         782         0x80000001 0x00fecf 1
    ```

## Conclusion
By splitting the network into areas, summarizing routes at the ABR, and using an ASBR to dynamically inject a default route, the network is now robust, scalable, and behaves like a large-scale enterprise network.
