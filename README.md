# Ka'ana like

A collaborative, fair, p2p backup system.

# How it works                                                                  
                                                                                
Each node has an encrypted volume (say, 1TB) where files are stored and shared on the network. It also has extra space for storing data from other nodes, with the size depending on the replication factor (2TB for 2 replicas, eg). The node will announce how much space it wants to share, and for how long:

- Node A: "I have 1 TB of data, and I'm willing to hold data for 7 days."
- Node B: "I accept your offer."
                                                                                
This means that nodes A and B will share a 1TB contract: node A will store 1TB of data from node B, and vice-versa. They exchange the data, reading directly from the device holding the encrypted volume, so all exchanged data is encrypted transparently:

| Node | Unencrypted data | Encrypted data | Extra storage |
| ---- | ---------------- | -------------- | ------------- |
|  A   | [1, 2]           | [a, b]         | [c, d, -, -]  |
|  B   | [3, 4]           | [c, d]         | [a, b, -, -]  |

## Verifying that the data is being stored

Node A needs to verify from time to time that node B is keeping its promise, and storing the data from node A. In order to do that, node A can issue a challenge to node B at any time:

- Node A: "What is the second byte of my data?"
- Node B: "b"
- Node A, "Cool, thanks!"

If the challenge fails, after the contract time (7 days, in this example) data from Node B will be deleted and the space will be available for other contracts.

## Verifying that the data is being kept for the expiration

Imagine Node A goes offline, and Node B issues a challenge:

- Node B: "What is the first byte of my data?"
- Node A: "..."

Node B has agreed to hold Node A's data for 7 days after a failed challenge. This allows Node A time to recover and get back in the system. But Node B could be greedly and immediately delete Node A's data, trying to free the space in order to establish a new contract with a different node. In order to prevent this, nodes will fail the challenge on purpose:

- Node B: "What is the first byte of my data?"
- Node A *lies*: "e"
- Node B: "Wrong! I'll delete your data in 7 days!"

After 6 days, Node A verifies that Node B still has its data:

- Node A: "What is the first byte of my data?"
- Node B: "a"
- Node A: "Cool, thanks! Challenge me again!"
- Node B: "What is the first byte of my data?"
- Node A: "c"
- Node B: "Cool, thanks! I'll refresh the expiration of your data now."
