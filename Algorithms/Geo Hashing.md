
**Geo Hashing** is a method used to represent geographic coordinates (latitude and longitude) in a compact, short string. It is often used in systems that need to store, index, or query geographic data efficiently, particularly in the context of spatial databases, geo-distribution, and geospatial load balancing.

Geo hashing converts the latitude and longitude of a location into a string of characters that encode geographic areas at different levels of precision, with each character representing a progressively finer grid resolution.

### **How Geo Hashing Works**

1. **Base-32 Encoding**: Geo hashing uses a **base-32** encoding scheme, which means it uses a set of 32 different symbols (0-9, a-z) to represent the encoded geographical region.
    
2. **Interleaving**:
    
    - The algorithm works by interleaving binary representations of the latitude and longitude of a point.
    - The geographic area is divided into a grid of cells, where each cell corresponds to a specific geo hash value.
    - The precision of the hash increases with each additional character added to the geo hash string.
3. **Recursive Division**:
    
    - The process begins with dividing the world into two equal parts based on latitude (North and South) and longitude (East and West).
    - The algorithm further subdivides the resulting regions recursively. Each subdivision reduces the size of the area represented by the hash, with more digits providing higher resolution.
    - For example, a 5-character geo hash represents a square of approximately 4.9 km × 4.9 km in size, whereas a 10-character geo hash corresponds to an area as small as 1 meter by 1 meter.

### **Example of Geo Hashing**

Consider a point with latitude 40.6895 and longitude -74.0445, which is the location of the Statue of Liberty.

1. **Initial Step**: Divide the world into two halves: North/South for latitude, and East/West for longitude.
2. **Subsequent Steps**: Continue splitting the regions into smaller regions using binary encoding.
3. **Final Hash**: After enough subdivisions, the result is a string, such as **"dr5ru"**, which represents a specific geographic area.

### **Precision of Geo Hashing**

The length of the geo hash string determines the precision:

- **Shorter geo hashes** (e.g., 5 characters) represent a large area, while **longer geo hashes** (e.g., 12 characters) represent smaller, more specific areas.
    - 5-character geo hash: Represents a region of about **4.9 km × 4.9 km**.
    - 6-character geo hash: Represents a region of about **1.2 km × 1.2 km**.
    - 12-character geo hash: Represents a region as small as **1 meter × 1 meter**.

### **Use Cases for Geo Hashing**

Geo Hashing is widely used in scenarios where geographic proximity is important, and efficient querying of spatial data is needed. Some key use cases include:

#### 1. **Geospatial Indexing & Search**

- **Spatial databases**: Geo hashing is commonly used to index geographic data in databases, making it easier to perform location-based queries. For example, if you want to find all points within a certain radius of a location, geo hashing allows you to quickly find nearby points.
- **Efficient queries**: Since geo hashes represent a continuous space, querying for nearby locations can be done by comparing the geo hashes of different points.

#### 2. **Geospatial Load Balancing**

- **Geo-distribution of resources**: For applications like Content Delivery Networks (CDNs), geo hashing can help route requests to the closest server or data center based on geographic location. By grouping requests based on their geo hash prefixes, systems can route traffic efficiently to geographically appropriate servers.
- **Proximity-based services**: Services that match users to nearby resources (e.g., ride-sharing apps, food delivery, etc.) can use geo hashing to map users to service providers in the same or neighboring geo hash cells.

#### 3. **Geospatial Clustering**

- **Grouping nearby points**: Geo hashing can help cluster nearby points of interest (POIs) in maps or geospatial applications. For example, businesses could be grouped in a specific geo hash area for more targeted recommendations.
- **Analyzing geographic density**: Geo hashes can help in analyzing density patterns, such as identifying areas with high concentrations of events, users, or resources.

#### 4. **Geofencing**

- **Location-based triggers**: Geo hashing is used in geofencing applications to trigger events when a user enters or exits a specific geographic region. By mapping an area to a geo hash, you can check whether a user’s current location falls within the specified region.

#### 5. **Efficient Data Storage**

- **Storing geospatial data**: Since geo hashes are compact strings, they are efficient for storing in a database or using in search indexes.
- **Spatial data compression**: Geo hashes can reduce the data size when storing location-based data, making it easier to perform comparisons and range queries.

### **Advantages of Geo Hashing**

- **Efficient Range Queries**: Geo hashes allow for fast querying of data points within a given radius by comparing hash prefixes.
- **Compact Representation**: Geo hashes provide a short and efficient way to represent geographic coordinates, reducing storage overhead.
- **Geospatial Proximity**: Locations that are close to each other are likely to have similar geo hash prefixes, making them easy to identify and group together.

### **Challenges of Geo Hashing**

- **Resolution Issues**: Geo hashing can lead to precision issues at the boundaries of geohash cells. Locations near the edges of a geo hash cell may be quite far apart, even though their geo hashes are similar.
- **Uneven Grid Size**: The grid of geo hashes is not perfectly square, and the resolution of the geographic areas varies as you move from the equator to the poles. This is due to the curvature of the Earth.
- **Edge Cases**: As geo hashes divide the Earth's surface into regions, areas near the boundaries of geo hash cells can cause issues, especially for proximity-based applications.

### **Geohash Hierarchy Example**

Let’s look at a specific example:

- **Location**: Latitude 40.6895, Longitude -74.0445 (Statue of Liberty, New York)
    - **Geo hash (5 characters)**: `dr5ru` — covers an area of approximately 4.9 km × 4.9 km.
    - **Geo hash (6 characters)**: `dr5ru1` — covers an area of approximately 1.2 km × 1.2 km.
    - **Geo hash (12 characters)**: `dr5ru1v5p3t0` — covers an area of about 1 meter × 1 meter.



## More on GeoHash

Geohashing (or geohash) is a [geocoding](https://www.pubnub.com/learn/glossary/what-is-geocoding-and-reverse-geocoding/) method used to encode geographic coordinates (latitude and longitude) into a short string of digits and letters delineating an area on a map, which is called a cell, with varying resolutions (sizes). The more characters in the string, the more precise the location (smaller cell). 

### What is an example of a geohash?

Geohash is a public domain of encoding coordinates. An example of a geohash is the coordinate pair 28.6132,77.2291 being converted into a geohash of ttnfv2u. 

### What is the maximum length of a geohash?

The **maximum length of a geohash is 12 characters**. The length of a Geohash string depends on the desired level of precision or granularity required for representing a geographic location. 

Geohash length sizes and corresponding precision levels examples:

- A Geohash of length 1 represents a large area, such as a continent.
    
- A Geohash of length 2 represents a smaller area, such as a country.
    
- A Geohash of length 3 represents a region or large city.
    
- A Geohash of length 4 represents a smaller city or town.
    
- Geohashes of length 5 or higher represent progressively smaller areas, down to street level and even specific locations.
    

For example, a Geohash of length 6 might represent a neighborhood or a specific landmark, while a Geohash of length 12 could represent a location accurate to within a few meters.

## How Does Geohashing Work?

### Geohash algorithm and calculation

Geohashes use Base-32 alphabet encoding (characters can be 0 to 9 and A to Z, excl "A", "I", "L" and "O”). Imagine the world is divided into a grid with 32 cells. The first character in a geohash identifies the initial location as one of the 32 cells. This cell will also contain 32 cells, and each one of these will contain 32 cells (and so on repeatedly). Adding characters to the geohash sub-divides a cell, effectively zooming in to a more detailed area.

![](https://www.pubnub.com/cdn/3prze68gbwl1/assetglossary-17su9wok1ui0z7r/55420735eef7b0469e22092fdc0683f4/geohashing-large-scale-example.jpeg?w=700&h=550&fit=pad "geohashing large scale example")

Precision factor determines the size of the cell. A precision factor of one (geohash 1) creates a cell 5,000km high and 5,000km wide. Precision factor of six creates a cell 0.61km high and 1.22km wide, and a precision factor of nine creates a cell 4.77m high and 4.77m wide (cells are not always square).

![](https://www.pubnub.com/cdn/3prze68gbwl1/assetglossary-17su9wok1ui0z7s/54cff1f40a461b1019b02cf1c35b542a/geohashing-examples-san-francisco.png?w=700&h=550&fit=pad "geohashing examples san francisco")

## Geohashing Examples and Use Cases

Geohashing was originally developed as a URL-shortening service but it is now commonly used for spatial indexing (or spatial binning), location searching, mashups and creating unique place identifiers.

### Geohash benefits 

A geohash is shorter than a regular address, or latitude and longitude coordinates, and therefore easier to share, remember and store.

- **Social Networking:** Chat with people near you within a particular cell, and to [create chat apps](https://www.pubnub.com/learn/glossary/what-is-in-app-chat/).
    
- **Proximity Searches:** Find nearby locations using [API mapping](https://www.pubnub.com/learn/glossary/what-is-a-map-api/) or [geolocation APIs](https://www.pubnub.com/learn/glossary/what-is-a-geolocation-api/) and identify places of interest, restaurants, shops and accommodation establishments in an area.
    
- **Digital Travels:** Geohashers go on global expeditions to meet people and explore new places. The twist: the destination is a computer-generated geohash and participants in this turnkey travel experience have to write up and post their story on the internet.
    
- **Custom Interactive Apps:** Geohashing can be used to create [real-time interactive](https://www.pubnub.com/products/pubnub-platform/) apps.
    

You can try it out [here](https://www.movable-type.co.uk/scripts/geohash.html). Use the coordinates 40.748440989 and -73.985663981 for a view of the area around the Empire State Building in New York. Change the precision factor to increase or decrease the resolution (size of the cell), or use the geohash dr5ru6j2c5fqt for a 13-digit precision factor to zoom right in.

### What are the drawbacks to using a geohash?

Disadvantages to geohashing include:

1. Using a grid based geohashing algorithm does need meet high-precision requirements
    
2. Geohashing deviation changes as latitude increases because the Earth is an irregular ellipse.
    

Ready to incorporate real-time functionality in your app? [Create a free PubNub account today](https://admin.pubnub.com/#/register).

## Geohash alternatives

There are several options for representing geographic coordinates and handling spatial data. Some of most popular solutions include:

1. **Quadtree**: A quadtree is a hierarchical data structure used for spatial partitioning of two-dimensional space. It recursively subdivides space into quadrants, allowing efficient storage and retrieval of spatial data. Quadtree is commonly used in geographic information systems (GIS) and computer graphics.
    
2. **S2 Geometry**: S2 geometry is a spherical geometry library developed by Google for representing and manipulating geographic shapes on the Earth's surface. It divides the Earth into regions of varying sizes using a hierarchical grid system similar to Geohash but with some differences in encoding and precision.
    
3. **H3**: H3 is a hierarchical hexagonal grid system developed by Uber for spatial indexing and analysis. It divides the Earth into hexagonal cells of varying sizes, providing a uniform and efficient way to represent geographic areas at different resolutions.
    
4. **R-tree**: An R-tree is a spatial index structure used for indexing multidimensional information, particularly in databases and GIS. It organizes spatial objects into a tree hierarchy, allowing efficient queries for nearest neighbors, range searches, and spatial joins.
    
5. **GeoJSON**: GeoJSON is a format for encoding geographic data structures using JavaScript Object Notation (JSON). It provides a lightweight and human-readable way to represent points, lines, polygons, and other geometric shapes, making it suitable for storing and exchanging spatial data in web applications and geographic databases.


### References

https://www.youtube.com/watch?v=6uhSpLjGLgo

https://www.movable-type.co.uk/scripts/geohash.html

[[Quad Tree]]

