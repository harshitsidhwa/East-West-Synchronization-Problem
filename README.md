# East-West-Synchronization-Problem
# Problem statement

There is a bridge which is aligned along the east-west direction. This bridge is too narrow to allow cars to go in both directions. Hence, cars must alternate going across the bridge. The bridge is also not strong enough to hold more than three cars at a time. Cars that want to get across should eventually get across. However, we want to maximize use of the bridge. Cars should travel across to the maximum capacity of the bridge (that is, three cars should go at one time). If a car leaves the bridge going east and there are no westbound cars, then the next eastbound car should be allowed to cross. We don't want a solution which moves cars across the bridge three at a time, i.e., eastbound cars that are waiting should not wait until all three cars that are eastboung and crossing the bridge have crossed before being permitted to cross.

# Solution:

I believe this problem can be recast as a multiple readers writers problem. Here's how it works. Intuitively, if you are a reader coming to the room, and you see writers in the room, you sit on the bench (if you are an eastbound car, and westbound cars are going across, you wait). If you are a reader, and there are fewer than three readers, and there are no writers waiting, you enter the room. If you are a reader, and there are already three readers, then you wait. If you are reader, and are exiting the room, then if there are no writers, but there are readers waiting, you let a single reader in. If there are writers waiting, but you are not the last reader to leave, you just leave. If you are the last reader to leave, there should be no waiting readers unless there are waiting writers. In either case, you let up to three writers into the room.
The main difference between the multiple readers-multiple writers problem (the exam problem) is that at most three readers or writers can be in the room at the same time. We still have the last reader potentially letting up to three writers in and vice versa.

Below is code for the reader processes. The writer process is analogous. Just replace "r" for "w" and vice-versa. As a reminder, r_active is the number of active readers (readers in critical section, r_waiting is the number of readers waiting to go into the critical section. r_sem is the semaphore that waiting readers sleep on. The writers have analogous variables. lock is a mutex semaphore used by both readers and writers.

 1   P( lock ); // lock is a mutex and is initializes to 1. <br>
 2   if (( nw_active + nw_waiting == 0 ) && ( nr_active < 3 )) <br>
 3     { <br>
 4        nr_active++; // Notify we are active<br>
 5        V( r_sem );  // Allow ourself to get through<br>
 6     } <br>
 7   else <br>
 8     nr_waiting ++;  // We are waiting  <br>
 9   V( lock ); <br>
10   P( r_sem );  // Readers will wait here, if they must wait.<br>
11<br>
12   READING...   <br>
13<br>
14   P( lock );  <br>
15   nr_active--;   <br>
16   if (( nr_active == 0 ) && ( nw_waiting &lt; 0 )) <br>
17     { // If we are the last reader  <br>
18        count = 0;  // Makes sure we only at most 3 writers in<br>
19        while (( nw_waiting > 0 ) && ( count &gt; 3 ))  <br>
20        { <br>
21           V( w_sem );  // wake a writer; <br>
22           w_active++;  // one more active writer <br>
23           w_waiting--; // one less waiting writer. <br>
24           count++;     <br>
25        } <br>
26      } <br>
27    else if (( nw_waiting == 0 ) && ( nr_waiting > 0 ) ) <br>
28      {  // allow another waiting reader to go in, if no waiting writers <br>
29        V( r_sem ); <br>
30        w_active++;  // one more active reader <br>
31        w_waiting--; // one less waiting reader.  <br>   
32      } <br>
33    V( lock ); <br>
Some invariants that hold: <br>
Safety conditions: Readers and writers can not be active at the same time.
  nr_active <= 3
  nw_active <= 3
  nr_active > 0 -> nw_active == 0
  nw_active > 0 -> nr_active == 0
Other conditions.
  nr_active > 0 & nr_waiting > 0 -> nw_waiting > 0
  nw_active > 0 & nw_waiting > 0 -> nr_waiting > 0
In case you are interested, this problem was on a grad level Ph.D. exam. There is a certain lack of realism here. When the three cars are going across, this implementation doesn't really force the first process that enters the bridge to complete first. I.e., if we have three cars, A, B, and C, and they enter in that order, they will also leave in that order. The above basically lets three in, and they can leave in any order.

# Refrences: 
  <-> http://www.cs.umd.edu/~hollings/cs412/s96/synch/eastwest.html
