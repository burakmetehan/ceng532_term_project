.. include:: substitutions.rst

|bfms_synchronizer|
=========================================



Background and Related Work
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Clock synchronization in distributed systems ensures consistent timekeeping across geographically dispersed processes, which is critical for coordinated operation. Traditional algorithms like Lamport's logical clocks [2]_ and Cristian's algorithm [3]_ rely on the assumption of well-behaved processes. However, these approaches fail when faced with Byzantine failures, where processes exhibit arbitrary behavior including sending incorrect information, withholding data, or even actively trying to disrupt the synchronization process.

Byzantine Fault Tolerance (BFT) protocols offer solutions for tolerating failures in distributed systems. However, these protocols can be complex and computationally expensive. The inefficiency of traditional BFT approaches in the context of clock synchronization motivated the development of more lightweight and efficient solutions specifically designed for this task.

The Mahaney-Schneider synchronizer falls within this category. It leverages message exchange and statistical analysis to achieve clock synchronization even in the presence of Byzantine processes. While earlier work on clock synchronization with failures considered omission failures (processes simply failing to respond) [4]_, the Mahaney-Schneider synchronizer tackles the more challenging scenario of Byzantine failures, making it a valuable contribution to the field.

.. [2] Lamport, L. (1978). Time, clocks, and the ordering of events in a distributed system. Communications of the ACM, 21(7), 558-565.
.. [3] Cristian, F. (1989). Analyzing clock synchronization algorithms. IEEE Transactions on Parallel and Distributed Systems, 10(8), 814-828.
.. [4] Srikanta, P., & Toueg, S. (1987). Optimal clock synchronization in fault-tolerant distributed systems. In Proceedings of the 7th ACM Symposium on Principles of Distributed Computing (PODC '88) (pp. 124-136).


Distributed Algorithm: |bfms_synchronizer| 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Mahaney-Schneider synchronizer operates in rounds. Each process broadcasts its local clock value and the received values from other processes in each round. The algorithm leverages message exchange and statistical analysis to identify reliable estimates of the true system time. Here's the pseudocode for a simplified version:

.. code-block:: RST
    :linenos:
    :caption: Mahaney-Schneider Synchronizer.
    
    function Synchronize():
        while True:
            local_time <- ReadLocalClock()
            broadcast (local_time, received_values)  // received_values stores messages from other processes
            received_values <- collect messages from other processes
            
            // Filter messages based on predefined criteria (e.g., median)
            filtered_values <- FilterMessages(received_values)
            estimated_time <- CalculateMedian(filtered_values)
    
            UpdateLocalClock(estimated_time)


**1. function Synchronize():**

This line defines a function named `Synchronize` which encapsulates the core logic of the algorithm.

**2. local_time <- ReadLocalClock()**

This line retrieves the current time from the process's local clock and stores it in the `local_time` variable.

**3. broadcast (local_time, received_values)**

This line performs two actions:

  * **broadcast(local_time):** The process broadcasts its local time (`local_time`) to all other participants in the distributed system. This message sharing allows each process to gather timestamps from all others.
  * **received_values:** This variable acts as a container to store messages received from other processes during the current round of synchronization.

**4. received_values <- collect messages from other processes**

This line waits for messages containing timestamps from other processes and stores them in the `received_values` variable. This essentially collects the broadcasted information from all participants. 

**5. filtered_values <- FilterMessages(received_values)**

This line calls a separate function named `FilterMessages`. This function takes the received messages (`received_values`) as input and performs some filtering operation to eliminate potentially misleading information from Byzantine processes. The filtered messages are then stored in the `filtered_values` variable.

**6. estimated_time <- CalculateMedian(filtered_values)**

This line calls another function named `CalculateMedian`. This function takes the filtered messages (`filtered_values`) as input and calculates the median value. The median represents the "middle" value in a sorted list, offering some resilience to outliers (potentially incorrect timestamps from Byzantine processes). The calculated median is stored in the `estimated_time` variable, representing the estimated system time based on the filtered data.

**7. UpdateLocalClock(estimated_time)**

This line attempts to adjust the process's local clock based on the estimated system time (`estimated_time`). The specific update mechanism might involve gradually adjusting the local clock towards the estimated time to avoid abrupt jumps.

**8. while True:**

This line indicates a loop that ensures the `Synchronize` function continues to run indefinitely. In essence, the process keeps participating in synchronization rounds repeatedly.

**Note:** This explanation assumes a simplified version. The actual implementation might involve additional details.

Example
~~~~~~~~

Imagine a scenario with three processes (P1, P2, and P3). P1 and P2 have accurate clocks, while P3 exhibits Byzantine behavior. In a round, P1 and P2 broadcast their correct times, while P3 broadcasts a random value. Filtering eliminates P3's message, and the remaining messages provide an accurate estimate for clock synchronization.

Correctness
~~~~~~~~~~~

The Mahaney-Schneider synchronizer can be proven to converge to a correct system time as long as a sufficient number of honest processes exist. The filtering mechanism helps eliminate the influence of Byzantine processes, ensuring convergence towards a reliable estimate.


Complexity 
~~~~~~~~~~

The complexity of the algorithm depends on the chosen filtering method and message exchange frequency. In general, it scales linearly with the number of processes participating in the synchronization process.
