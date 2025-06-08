
A **Merkle Tree** is a type of **binary tree** used primarily in **cryptography** and **distributed systems** to efficiently and securely verify the integrity of data. It's used in applications like **blockchains**, **version control**, and **distributed file systems**, where the goal is to check if data has been tampered with, without needing to verify the entire dataset.

In a Merkle Tree, each **leaf node** represents a **hash** of a piece of data, and each **non-leaf node** (parent) represents the hash of the concatenation of its children. The **root hash** of the tree summarizes the entire dataset, providing a compact and efficient means to verify data integrity.

### **Structure of a Merkle Tree:**

- **Leaf nodes**: These nodes store the hashes of the individual data blocks (e.g., transactions, file blocks).
- **Internal nodes**: These nodes store hashes derived from their child nodes.
- **Root node**: The top node of the tree that provides a **hash** representing the entire dataset.

### **Merkle Tree Hashing Process:**

1. **Data Partitioning**:
    - Divide the data into smaller chunks (blocks) to hash.
2. **Leaf Nodes**:
    - Each data block is hashed using a cryptographic hash function (like SHA-256).
3. **Internal Nodes**:
    - The hash of each parent node is calculated by combining (typically concatenating) the hashes of its child nodes and applying the hash function again.
4. **Root Node**:
    - The root node’s hash represents the overall dataset's integrity.

### **Example:**

Here's a basic implementation of a **Merkle Tree** in Java, using **SHA-256** for hashing:

```
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.List;

public class MerkleTree {

    // Function to calculate SHA-256 hash
    public static String sha256(String input) throws NoSuchAlgorithmException {
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        byte[] hash = digest.digest(input.getBytes());
        StringBuilder hexString = new StringBuilder();
        for (byte b : hash) {
            hexString.append(String.format("%02x", b));
        }
        return hexString.toString();
    }

    // Function to build a Merkle Tree
    public static String buildMerkleTree(List<String> dataBlocks) throws NoSuchAlgorithmException {
        List<String> currentLevel = new ArrayList<>();

        // Step 1: Create leaf nodes (hash of data blocks)
        for (String data : dataBlocks) {
            currentLevel.add(sha256(data));
        }

        // Step 2: Build internal nodes until we reach the root
        while (currentLevel.size() > 1) {
            List<String> nextLevel = new ArrayList<>();
            for (int i = 0; i < currentLevel.size(); i += 2) {
                // If odd number of elements, the last one is carried over as is
                if (i + 1 < currentLevel.size()) {
                    String combined = currentLevel.get(i) + currentLevel.get(i + 1);
                    nextLevel.add(sha256(combined));
                } else {
                    nextLevel.add(currentLevel.get(i));
                }
            }
            currentLevel = nextLevel;
        }

        // The final remaining element is the root hash
        return currentLevel.get(0);
    }

    public static void main(String[] args) throws NoSuchAlgorithmException {
        // Example data blocks
        List<String> dataBlocks = List.of("Block1", "Block2", "Block3", "Block4");

        // Build Merkle Tree and get root hash
        String rootHash = buildMerkleTree(dataBlocks);
        System.out.println("Merkle Root Hash: " + rootHash);
    }
}

```

### **Explanation of the Code:**

1. **`sha256` Method**:
    
    - This method takes an input string and returns its **SHA-256** hash. It uses the `MessageDigest` class from Java's standard library.

1. **`buildMerkleTree` Method**:
    
    - First, we generate the leaf nodes by hashing the data blocks (strings in this case).
    - Then, we repeatedly combine pairs of hashes (concatenate them) and hash them again to generate the parent nodes. This continues until only one node remains — the **root**.
    - If the number of nodes at any level is odd, the last node is carried over to the next level.

3. **`main` Method**:
    - Example data blocks (strings) are provided, and the Merkle root hash is computed and displayed.

### **How It Works:**

Given four data blocks (`Block1`, `Block2`, `Block3`, `Block4`), the Merkle tree is constructed as follows:

- **Step 1**: Hash each data block:  
    `H(Block1)`, `H(Block2)`, `H(Block3)`, `H(Block4)` (these are leaf nodes).
    
- **Step 2**: Hash pairs of hashes:  
    `H(H(Block1) + H(Block2))`, `H(H(Block3) + H(Block4))`.
    
- **Step 3**: The final root hash will be the hash of the concatenation of these two internal nodes:  
    `H(H(H(Block1) + H(Block2)) + H(H(Block3) + H(Block4)))`.
    

The final **root hash** represents the entire dataset, and if even a single block of data is changed, the root hash will change, indicating a tampering of the data.

---

### **Benefits of Merkle Trees:**

1. **Efficient Data Verification**:
    
    - Instead of comparing entire datasets, you only need to compare the root hash or a subset of hashes (in the case of verifying specific data) to detect discrepancies.
2. **Cryptographic Security**:
    
    - The cryptographic hash functions used in Merkle trees (e.g., SHA-256) ensure that even a small change in the data will result in a completely different hash, providing high security for data integrity.
3. **Scalability**:
    
    - Merkle trees are highly efficient for large datasets, as the verification process only involves the hashes, rather than comparing entire data blocks.
4. **Fault Tolerance**:
    
    - In distributed systems, Merkle trees allow for efficient comparison of data blocks across different nodes, which can detect discrepancies without having to transfer large amounts of data.

---

### **Use Cases of Merkle Trees:**

- **Blockchain**: Used to ensure that all transactions in a block are valid. The root hash represents the entire block, and verifying the integrity of the blockchain is efficient by only comparing the root hashes of blocks.
- **Distributed Systems**: In systems like **Git** or **HDFS**, Merkle trees are used to compare versions of files or data blocks efficiently.
- **Cryptographic Protocols**: Merkle trees play a key role in cryptographic protocols like **Merkle-Damgård**construction, **Merkle signatures**, and **zero-knowledge proofs**.

---

In conclusion, Merkle trees provide a highly efficient and secure way of verifying large datasets or blocks of data in distributed and cryptographic systems. By utilizing cryptographic hash functions, Merkle trees offer strong guarantees about data integrity and are a fundamental building block in modern systems like blockchain and distributed file storage.

Merkle Trees are used in popular software applications like Git, Amazon Dynamo DB and BlockChain. A merkle tree is a metadata-structure, which stores the hash of the combined children in each parent. Any change in a child requires a change in the parent node.

This property of identifying state changes is useful when validating the correctness of data, as calculating the inverse of a hash is computationally infeasible. The merkle tree's properties are suitable for applications where a malicious user must be stopped from changing data. 

##### Git file creation details:
Git registers any change in a file as an entirely new file. It stores the content of the file with a filename of SHA1(fileContents). Let's say this SHA1(fileContents) = x. If x already exists in the git file system, it doesn't go ahead and create it. This avoids unnecessary recreations when metadata about a file is changed. 

When a file is modified in Git, git creates a new (changed)file, and all it's changed parents. The root subsequently changes. It's like a branch of a Persistent Data Structure. If you want to go back a commit, you use the root of the previous commit, which still points to the old file. 

In this way, Git takes snapshots of every point of change in the project. Perfect for a version control system. 

It is also useful to find the points at which the data has changed, using a Merkle Tree. Amazon Dynamo DB uses merkle trees to reduce entropy (Anti-entropy technique) when a new node is added to the Dynamo DB cluster.
### References

https://www.youtube.com/watch?v=3AcQyTs_Es4

https://www.youtube.com/watch?v=qHMLy5JjbjQ