*<font color = red>Public servers of a scalable web serve are hidden behind a load balancer</font>*

<br>

**First golden rule for scalability:**
- Every server exactly contains the same codebase and does not store any user related-data, like sessions or profile pictures, on local dics or memory.  

**Second rule for Database :**
-  We can stay with MySQL, but use it like NoSQL database, or we can switch to a easier and better scale NoSQL database like MongoDB or CouchDB. 

**Third rule for cache:**
-  Let your Class assemble a dataset from your database and then store the complete instance of the Class or the assembled dataset in the cache.
-  some ideas of objects to cache:
   -  user sessions 
   -  fully rendered blog articals
   -  activity streams
   -  user<->friend relationships

**Fourth rule for Async:**
-  Do the time-consuming work in advance and serve the finished work with a low request time.
-  Handle tasks asynchronously.

## <font color=red>Performance vs Scalability</font> 
Sum: A serve is scalable if it reaults in increased performance in a manner proportional to resourses added.  

### Why is scalability so hard ?
-  Scalability cannot be after-thought, it requires applications and platforms to be designed with scalaing in mind.  
-  A second problem area is that growing a system through scale-out generally results in a system that has to come to terms with hereogeneity.
  

## <font color=red>Latency vs throughout</font>
*Latency* is the time to perform some action or to produce some result.
*Throughout* is the number of such actions or results per unit if time.  

## <font color=red>Availability vs consistency</font>
### CAP Theorem
-  Consistency - A read is guaranteed to return the most recent write for a given client.
-  Availability - A non-falling node will return reasonable response within a mount of reasonable time (no error or time out).
-  Partition Tolerance - The system will continue to function when network Partitions occur.
    -  CP - Consistency/Partition Tolerance - Waiting for a response from the partitioned node might result in a timeout error. CP is a good choice if your business needs require atomic reads and writes.
    -  AP - Availability/Partition Tolerance - Responses return the most readily available version of the data available on any node, which might not be the latest. Writes might take some time to propagate when the partition is resolved.

### Consistency patterns
#### Weak consistency
After a write, reads may or may not see it. A best effort approach is taken.
#### Eventual consistency
After a write, reads will eventually see it (tipycally within milliseconds). Data is replicated ansychronously.
#### 