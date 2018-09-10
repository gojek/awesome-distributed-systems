# Serializability

### What is Serializability?
It is a mechanism to deal with concurrency in database transactions. Basically a concurrency control system.

 To understand serializability let’s first understand the transaction.


### What is a transaction?
 - A transaction is a group of commands that change the data stored in a database. 
 - It is treated as a single unit of work. 
 - It ensures that, either all of the commands succeed, or none of them. If one of the commands in the transaction fails,
    all of the commands fail, and any data that was modified in the database is rolled back. In this way, transactions maintain the integrity of data in a database.


### What is concurrency?
- Multiple transaction happening at the same point to the same set of data


### What are the concurrency problems?
- Dirty reads
    - A dirty read happens when one transaction is permitted to read data that has
    been modified by another transaction that has not yet been committed. 
    In most cases this would not cause a problem. However, if the first transaction is 
    rolled back after the second reads the data, the second transaction has dirty data that does not exist anymore.


- Lost update
    - A lost update occurs when two different transactions are trying to update the same column on the same
    row within a database at the same time. Typically, one transaction updates a particular column in a
    particular row, while another that began very shortly afterward did not see this update before updating
    the same value itself. The result of the first transaction is then "lost", as it is simply overwritten
    by the second transaction.


- Non repeatable reads
    - Non repeatable read problem happens when one transaction reads the same data twice and another
    transaction updates that data in between the first and second read the transaction. In that case the value
      wil be different each time for the same query in the same transaction.


- Phantom reads
    - Phantom read happens when one transaction executes a query twice and it gets a different number of rows
    in the result set each time. This happens when a second transaction inserts a new row after first query in
    first transaction.


These problem happens when there are no isolation levels in the database. To solve these problems we try to create
some isolation layers. The goal of these isolation levels is to achieve Serializability

### Serializability:
- Serializability insures that all the concurrent transaction are being performed in such a way that the end
output is same as performing them in serial order.
- This job is being done by the Isolation property of ACID.


There are two type of isolation levels

### Weak Isolation levels
- Weak isolation levels does not guaranty 100% serializability.
- It compromises serializability over speed
- Weaker isolation levels are useful for performance under certain circumstances, but they need
to be used carefully because they can produce incorrect results.
- examples
    - Read committed:

        This isolation level makes sure that no transaction can read any uncommitted data.
        This will resolve the problems like `dirty reads` but all the other concurrency problems will still occur

    - Repeatable reads:

        Repeatable read is a higher isolation level, that in addition to the guarantees of the read committed
        level, it also guarantees that any data read cannot change, if the transaction reads the same data
         again, it will find the previously read data in place, unchanged, and available to read.



### Strong isolation levels
- Strong isolation levels try to insure 100% serializability 
- It compromises speed over serializability
- example 
    - Actual serial execution
        - The simplest way of avoiding concurrency problems is to remove the concurrency entirely: to
        execute only one transaction at a time, in serial order, on a single thread. By doing so,
        we completely sidestep the problem of detecting and preventing conflicts between transactions:
        the resulting isolation is by definition serializable.
        - RAM became cheap enough that for many use cases is now feasible to keep the entire active dataset
        in memory (see “Keeping everything in memory”). When all data that a transaction needs to access is
        in memory, transactions can execute much faster than if they have to wait for data to be loaded from
        disk.
        - In the early days of databases, the intention was that a database transaction could encompass an
        entire flow of user activity. For example, booking an airline ticket is a multi-stage process
        (searching for routes, fares, and available seats; deciding on an itinerary; booking seats on each
        of the flights of the itinerary; entering passenger details; making payment). Database designers
        thought that it would be neat if that entire process was one transaction so that it could be
        committed atomically
        - With stored procedures and in-memory data, executing all transactions on a single thread becomes
        feasible. As they don’t need to wait for I/O and they avoid the overhead of other concurrency control
        mechanisms, they can achieve quite good throughput on a single thread.
     ```
    Limitations
           - Every transaction must be small and fast, because it takes only one slow transaction to stall all transaction processing
           - It is limited to use cases where the active dataset can fit in memory. Rarely accessed data
           could potentially be moved to disk, but if it needed to be accessed in a single-threaded
           transaction, the system would get very slow.x
           - Bad write thoughput
           - Executing all transactions serially makes concurrency control much simpler, but limits the
            transaction throughput of the database to the speed of a single CPU core on a single machine.
     ```

    - 2 Phase locking
        - We saw previously that locks are often used to prevent dirty writes (see “No dirty writes”): if
        two transactions concurrently try to write to the same object, the lock ensures that the second
        writer must wait until the first one has finished its transaction (aborted or committed) before it
        may continue.
        - Two-phase locking is similar, but makes the lock requirements much stronger. Several transactions
        are allowed to concurrently read the same object as long as nobody is writing to it. But as soon as
        anyone wants to write (modify or delete) an object, exclusive access is required:
        - If transaction A has read an object and transaction B wants to write to that object, B must wait
         until A commits or aborts before it can continue. (This ensures that B can’t change the object
         unexpectedly behind A’s back.)
        - If transaction A has written an object and transaction B wants to read that object, B must wait
         until A commits or aborts before it can continue.
        - In 2PL, writers don’t just block other writers; they also block readers and vice versa.

   ```
    Limitations
        - The big downside of two-phase locking, and the reason why it hasn’t been used by everybody
        since the 1970s, is performance: transaction throughput and response times of queries are
         significantly worse under two-phase locking than under weak isolation.
        - This is partly due to the overhead of acquiring and releasing all those locks, but more
        importantly due to reduced concurrency. By design, if two concurrent transactions try to do
         anything that may in any way result in a race condition, one has to wait for the other to
         complete.
        - Traditional relational databases don’t limit the duration of a transaction, because they are 
        designed for interactive applications that wait for human input. Consequently, when one
        transaction has to wait on another, there is no limit on how long it may have to wait. Even 
        if you make sure that you keep all your transactions short, a queue may form if several
        transactions want to access the same object, so a transaction may have to wait for several 
        others to complete before it can do anything.

  ```

  - Snapshot
    - This video explains it. https://www.youtube.com/watch?v=9NVu17LjPSA
