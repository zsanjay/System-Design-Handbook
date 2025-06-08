
Change Data Capture (CDC) is the process of recognizing when data has changed in source system so that a downstream system can take an action based on that change.

It’s a very good way to to move data from your transactional databases to your data warehouses or data lakes with minimal latency. It’s also a good way to setup a real time data pipeline where other processes, like stream processors, can listen for changes in data and take actions accordingly.

**Change Data Capture (CDC)** is a technique used in database management and data processing that identifies and captures changes made to data in a database. It allows systems to track and record any modifications (such as inserts, updates, and deletes) in real time or near real time, enabling other systems to react to those changes. CDC is often used in data replication, data warehousing, and data integration processes.

### Key Concepts of Change Data Capture (CDC):

1. **Capture of Changes**:
    
    - **Insertions**: Captures new records added to the database.
    - **Updates**: Tracks modifications made to existing records.
    - **Deletions**: Monitors records that are removed from the database.
2. **Real-time or Near Real-time Data Synchronization**:
    
    - CDC enables systems to react to changes in near real-time by pushing the changes to downstream systems, like data warehouses, data lakes, or analytics platforms.
3. **Data Replication and Integration**:
    
    - One of the main uses of CDC is in **data replication**, where changes made to a source database are continuously copied and applied to a target database.
    - It's also used in **ETL (Extract, Transform, Load)** processes to move only the changed data between systems, reducing unnecessary data processing and increasing efficiency.
### References

https://www.youtube.com/watch?v=5KN_feUhtTM