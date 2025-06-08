
## Functional Requirements

- **Viewers' Requirements:**
    
    - Immediate video playback upon request
    - Consistent viewing experience regardless of traffic spikes
    - Compatibility across different devices and network speeds
- **Creators' Requirements:**
    
    - Straightforward video upload URL procurement
    - Efficient file transfer to the server for various video sizes
    - Confirmation of successful upload and encoding

## Non-Functional Requirements

- 100M Daily Active Users
- Read:write ratio = 100:1
- Each user watches 5 videos a day on average
- Each creator uploads 1 video a day on average
- Assuming that each video is 500MB in size.
- Data retention for 10 years
- Low latency
- Scalability
- High availability

## Capacity Planning

We will calculate QPS and storage using our [Capacity Calculator](https://systemdesignschool.io/resource-estimator?dau=100000000&read_write_ratio=100:1&write_operations=1&data_retention=120&data_per_write_request=524288000&precision_mode=true).

### QPS

`QPS = Total number of query requests per day / (Number of seconds in a day)` Given that the DAU (Daily Active Users) is 100M, with each user performing 1 write operation per day, and a Read:Write ratio of 100:1, the daily number of read requests is: `100M * 100 = 10B`. Therefore, **`Read QPS = 10B / (24 * 3600) ≈ 115740`**.

### Storage

`Storage = Total data volume per day * Number of days retained` Given that the size of each video is 500MB, and the daily number of write requests is 100M, the daily data volume is: `100M * 500MB`. To retain data for 10 years, the required storage space is: **`100M * 500MB * 365 * 10 ≈ 183EB`**.

### Bottleneck

Identifying bottlenecks requires analyzing peak load times and user distribution. The largest bottleneck would likely be in video delivery, which can be mitigated using CDNs and efficient load balancing strategies across the network.

When planning for CDN costs, the focus is on the delivery of content. Given the 100:1 read:write ratio, most of the CDN cost will be due to video streaming. With aggressive caching and geographic distribution, however, these costs can be optimized.

## High-level Design

The high-level design of YouTube adopts a microservices architecture, breaking down the platform into smaller, interconnected services. This modular approach allows for independent scaling, deployment, and better fault isolation.

![Youtube System Design Diagram](https://systemdesignschool.io/solutions/youtube/high-level-design.png)![](https://systemdesignschool.io/search.svg)

**Read Path**: When a user initiates a request to view a video, this request is processed by the [load balancer and API Gateway](https://systemdesignschool.io/blog/api-gateway-vs-load-balancer), which then routes it to the **Video Playback Service**. This service efficiently retrieves video data through caching layers optimized for quick access, before accessing the Video Metadata Storage for the video's URL. Once retrieved, the video is streamed from the nearest CDN node to the user's device, ensuring a seamless playback experience.

1. The **Content Delivery Network (CDN)** is crucial for delivering cached videos from a location nearest to the user, significantly reducing latency and enhancing the viewing experience.
2. The **metadata database** is responsible for managing video titles, descriptions, and user interactions such as likes and comments. These databases are optimized to support high volumes of read operations efficiently.

**Write Path**: Video uploads are handled through a distinct process that begins with the load balancer and API Gateway, directing the upload request to the **Video Upload Service**.

- **Generating Signed URLs**: The Video Upload Service obtains a signed URL from the Storage Service, which facilitates direct access to the object storage systems (e.g., Amazon S3, Google Cloud Storage, or Azure Blob Storage). A signed URL contains a cryptographic signature, granting the holder permission to perform specific actions (like uploading a file) within a designated timeframe.
- **Direct Upload to Object Storage**: The client application receives the signed URL and uses it to upload the video file directly to the object storage. This method bypasses the application servers, considerably reducing their workload and enhancing the system's overall scalability.
- **Upload Completion Notification**: Following a successful upload, the client notifies the Video Upload Service, providing relevant video metadata. This triggers subsequent processes such as video processing (transcoding), thumbnail generation, and metadata recording.
- **Video Processing Pipeline**: The upload event initiates a series of processing tasks, including content safety checks (to identify any prohibited content), transcoding (to convert the video into different formats, resolutions, and compress the file), and resolution adjustments.
- **Distribution to CDNs**: The final step involves uploading the processed videos to CDN nodes, ensuring that they are readily accessible to viewers from the most efficient locations.

## Detailed Design

### Data Storage

```
Video Table
+------------------+--------------+-----------------+
| video_id (PK)    | uploader_id  | file_path       |
| title            | description  | encoding_format |
| upload_date      | thumbnail    | duration        |
+------------------+--------------+-----------------+

Users Table
+------------------+-------------+--------------+
| user_id (PK)     | username    | email        |
| password_hash    | join_date   | last_login   |
+------------------+-------------+--------------+

Video Stats Table
+------------------+------------+-------------+------------+
| video_id (FK)    | views      | likes       | dislikes   |
| shares           | save_count | watch_time  |
+------------------+------------+-------------+------------+

```

Explanation of each table:

- **Video Table**: This table stores metadata about each video, such as the video ID, uploader, storage path, and title. Title and description help in search and discovery, while file_path and encoding_format are vital for playing the video.
- **Users Table**: Retains data on registered users, including secure login information and activity logs for an enhanced, personalized experience.
- **Video Stats Table**: Crucial for content creators and recommendation algorithms; this table tracks video engagement and informs popularity metrics.

For [database sharding](https://systemdesignschool.io/fundamentals/database-partitioning), selecting the `shard id` typically depends on data distribution and access patterns. Videos may be sharded by video ID or uploader ID to balance the load among shards; users are often sharded by user ID for even distribution.

[Database replication](https://systemdesignschool.io/fundamentals/database-replication) is another critical consideration for enhancing data availability and fault tolerance. Read replicas support high read volumes, especially on workloads skewed towards queries like fetching `video stats`.

**Examples**:

- A record in the `Video Table` may relate to a specific `video_id`, storing which shard holds the corresponding video file.
- The `Users Table` might employ consistent hashing based on the `user_id` to distribute records across database instances.
- With the `Video Stats Table`, a sharding strategy using `video_id` facilitates join operations with the `Video Table`.
- Replication of the `Comments Table` could be implemented to serve hot data faster in different geographical locations while offering enhanced protection against data loss.

## Deep Dives

### Video Transcoding

Video transcoding is the process of converting a video file from one format to another, adjusting various parameters such as the codec, bitrate, and resolution to ensure compatibility across a wide range of devices and internet connection speeds. This process is crucial for platforms like YouTube, where videos need to be accessible on everything from high-end desktops to mobile phones with limited bandwidth. Transcoding makes it possible to provide an optimal viewing experience regardless of the viewer's device or network conditions, by dynamically serving different versions of the video tailored to their specific situation.

The necessity of video transcoding for YouTube arises from the diverse array of content uploaded to the platform. Videos come in from creators around the world, encoded in various formats and qualities. To standardize this content for playback on the YouTube platform—and to ensure that every user, regardless of their device or internet speed, can watch videos without excessive buffering or data usage—transcoding is a must. It allows YouTube to convert these myriad formats into a consistent set of standardized formats. For instance, a high-resolution video might be transcoded into lower resolutions (1080p, 720p, 480p, etc.) to accommodate users with slower internet connections, while still being available in its original high definition for those with the bandwidth to support it.

### Video Processing Pipeline

YouTube's video processing pipeline is structured to handle various tasks from the moment a video is uploaded until it's ready for global streaming. This pipeline ensures videos are compatible across different devices and network speeds while adhering to YouTube's content guidelines. Here are example stages:

![Youtube System Design Video Processing Pipeline](https://systemdesignschool.io/solutions/youtube/video-processing-pipeline.png)![](https://systemdesignschool.io/search.svg)

- **Upload**: The process begins when a user uploads a video to YouTube via the signed url.
- **Initial Checks**: Basic validations such as file integrity and format compatibility are performed.
- **Content Analysis**: Automated systems scan the video for copyright issues and adherence to community guidelines.
- **Transcoding**: The video is converted into multiple formats and resolutions for efficient streaming.
- **Encryption and Storage**: The processed videos are encrypted and stored in YouTube's data centers.
- **CDN Distribution**: Finally, the video is distributed across YouTube's global Content Delivery Network for fast access.

This diagram represents a streamlined view of the complex processes involved. In practice, many of these steps can occur in parallel or be subject to iterative improvements. The process forms a Directed Acyclic Graph (DAG). The pipeline leverages cloud computing and machine learning to efficiently manage the vast amount of content uploaded every minute, ensuring a consistent and reliable preparation of videos for streaming on the platform.

### How does Signed URL Work and Why Do We Need It?

Signed URLs provide secure access to private resources temporarily.

![Youtube System Design Signed Url](https://systemdesignschool.io/solutions/youtube/signed_url.png)![](https://systemdesignschool.io/search.svg)

A signed URL works by providing the uploader—a video content creator in this case—with a unique, temporary URL that grants permission to upload a file to a specific location in the cloud storage. This URL includes a cryptographic signature, which is generated by the server and validates that the upload request is authorized. The signature ensures that the URL cannot be tampered with and typically comes with an expiration time, after which the URL becomes invalid.

The use of signed URLs is particularly crucial for platforms like YouTube for several reasons. First, it offloads the heavy lifting of data transfer directly to the object storage provider, such as Google Cloud Storage or Amazon S3, which significantly reduces the bandwidth and processing load on YouTube's own servers. This allows YouTube to scale more efficiently by handling potentially millions of simultaneous video uploads without degrading performance. Second, it enhances security by ensuring that only users with the correct, time-limited URL can upload videos, thus protecting against unauthorized access and data breaches.

Moreover, signed URLs are a behind-the-scenes mechanism that users do not interact with directly in the YouTube upload UI. When a content creator uploads a video, the complexity of generating and managing signed URLs is abstracted away by the YouTube platform. The uploader simply selects a video file and uploads it through the YouTube interface, unaware of the signed URL process occurring in the background. This abstraction is intentional, designed to provide a seamless and user-friendly experience. The uploader does not need to deal with the complexities of cloud storage interactions, cryptographic signatures, or URL expirations—instead, they enjoy a straightforward and intuitive upload process, while the platform securely and efficiently handles the file transfer.

### Importance of CDN

A CDN is instrumental in reducing latency and handling the large distribution of content by caching data in multiple locations closer to end-users. To best use a CDN, place nodes strategically in various geographic regions, cache popular content aggressively, and use dynamic routing to minimize congestion and downtime.

### How to Make Video Uploading Even Faster?

To speed up video uploads, optimize the network protocol, implement parallel chunk uploads, and use client-side compression where feasible. Additionally, provide feedback mechanisms such as progress bars to keep users informed.

## Followups

### How Does YouTube Handle Increasing User Demand and Video Data?

To accommodate increasing user demand and video data, YouTube implements a robust delivery network optimized for speed and efficiency. The use of global Content Delivery Networks (CDNs) ensures that video playback is accelerated and client-side lag is minimized, regardless of the user's physical distance from the server. By caching video content at edge servers located closer to the user, the system markedly reduces the time it takes for a video to start after it's clicked. Adaptive bitrate streaming further improves the experience by adjusting video quality in real-time based on the user's current internet speed, ensuring smooth playback without buffer interruptions.

### Considering the nearly unlimited duration of storage and the need for massive storage space, how can we design the system to be cost-effective without compromising the user experience?

In designing a system with budget-conscious yet massive storage solutions, innovative approaches are required. The use of a hierarchical storage system segregates data based on access frequency. High-accessibility and low-accessibility zones ensure that the most frequently watched videos are readily available, while less popular content is stored on cheaper, slower-access storage mediums. Additionally, implementing video compression algorithms and deduplication strategies minimizes the actual space required per video, significantly saving costs associated with data storage. Importantly, this must all be balanced against the maxim to never compromise the user's experience — streamlining storage but ensuring content is always delivered rapidly upon request.

### Can You Explain YouTube's Approach to Fault Tolerance and Disaster Recovery?

YouTube's strategy for fault tolerance involves distributing its services and data across multiple, geographically dispersed data centers, utilizing redundant systems and backup procedures. In the event of a server failure, traffic is rerouted to ensure uninterrupted service. For disaster recovery, YouTube employs real-time data replication and regular backups, enabling full system restoration in the case of catastrophic events. Failover mechanisms are tested regularly, with automatic switch-overs to backup systems in place to maintain service continuity. These strategies collectively provide a high degree of resilience, minimizing the impact of potential failures on the user experience.

### References

https://systemdesignschool.io/problems/youtube/solution