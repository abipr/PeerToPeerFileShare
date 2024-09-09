# P2PFileShare Project

Erik Hartker (erik.hartker@ufl.edu)

Abigail Prechter (prechterabigail@ufl.edu)

Samitha Kosanam (sa.foster@ufl.edu)


## How to run the project
Navigate to the directory that the project is in on the CLI. Then, run the following commands to compile and run the program for a single peer:

```
javac peerProcess.java
java peerProcess [peerID]
```
Repeat this process to run multiple peers. Run the program with multiple peerIDs on multiple different machines, or multiple tabs of a CLI. Fill in Common.cfg and PeerInfo.cfg with the appropriate fields based on the machines your are running the program on, and the file you are transferring.

## peerProcess.java

Main driver for the program.

| Method | Description |
|----------|----------|
| main() | The driver for the program, reads in all info from .cfg files and begins connecting with other peers. Spawns all threads which handle connections and choking. Downloads file to machine once it is completely downloaded. | 
| String[] readPeerInfo(String line) | Reads in PeerInfo.cfg line by line and assigns its values to a String[] | 
| String[] readCommon() | Reads in Common.cfg and assigns its values to a String[] | 

Attributes:
- fileByte: 2D array that holds all the bytes of the file to be downloaded
- peerThreads: a static list of PeerThreads that can be accessed across the program
- chokingManager: a ChokingManager object to handle choking and unchoking
- peers: a representation of peers and their attributes from the PeerInfo.cfg file

## PeerThread.java

Represents an individual connection between the local peer and a connected peer. Handles all communication between the two.

| Method | Description |
|----------|----------|
| run() | Handles handshaking and sending of bitfield, as well as verification of the peer it is connecting to. Calls listenForRequests() to handle the rest of the message passing. | 
| void listenForRequests() | Handles all incoming messages, and responds appropriately to them. Handles updating bitfield, requestedBitfield, and peerBitfield. Shuts down the program when all peer bitfields and the local bitfield are completed. | 
| void sendHandshake() | Sends a handshake message to the connected peer | 
| byte[] receiveHandshake() | Returns a byte array of the handshake message received | 
| byte[] getRequestPiece() | Determines which piece to send based on which pieces are in the peer bitfield and have not already been requested. Randomly selects one piece from all such pieces. | 
| byte[] receiveMessage() | Returns a byte array of the message received | 
| void sendMessage() | Sends a message of form byte[] | 
| byte[] composeMessage() | Formats a message number and optional payload byte[] to create a properly formatted message | 
| void compareBitfields() | Compares a peer bitfield and the local bitfield to determine interest in the peer. Sends an 'interested' or 'not interested' message accordingly | 

Attributes:
- socket: the socket for the connection
- bitfield: a representation of the local peer's bitfield
- requestedBitfield: a representation of the local peer's bitfield including requested pieces
- peerBitfield: a representation of the connected peer's bitfield
- choked: indicates whether the local peer has choked the connected peer
- peerChoked: indicates whether the connected peer has choked the local peer
- isInterested: indicates the interest of the local peer in the connected peer's pieces
- peerInterested: indicates the interest of the connected peer in the local peer's pieces
- peerID: represents the local peer's ID
- connectedPeerID: represents the connected peer's ID
- exit: indicates whether the thread should be exited or not in the case of a complete file for all peers
- downloadRate: represents the number of pieces downloaded by the local peer over the choking/unchoking interval

## ChokingManager.java

Handles the choking/unchoking and optimistic unchoking for each PeerThread

| Method | Description |
|----------|----------|
| void updateDownloadRates() | Handles the choking and unchoking of peers/the selection of preferred neighbors. Handles cases where the local peer has already downloaded the file and when it has not. | 
| void optimisticallyUnchoke() | Handles optimistic unchoking of a peer. | 
| void start() | Begins two ScheduledExecutorService objects that handle incremental choking/unchoking and optimistic choking/unchoking | 

Attributes:
- unchokedPeers: represents the number of preferred neighbors for a peer
- optimisticInterval: represents the interval (in seconds) for optimistic unchoking
- chokeInterval: represents the interval (in seconds) for choking/unchoking

## BitField.java

Representation of the downloaded/not downloaded pieces for each peer via a BitSet.

| Method | Description |
|----------|----------|
| void setBitfield(byte[] byteArray) | sets the BitSet bitfield based on the bit values of a byte array  | 
| List<Integer> getAvailableList() | Returns the indices of not downloaded pieces in the bitfield | 
| byte[] convertToByte() | Converts the bitfield BitSet into its byte representation | 

Attributes:
- bitfield: a BitSet which represents the downloaded/undownloaded pieces of the file for a peer
- numPieces: the number of pieces in the bitfield
- fileSize: the size of the file in bytes
- pieceSize: the size of each piece in bytes

## LogWriter.java

Writes logs to log files for each peer.

| Method | Description |
|----------|----------|
| void writeConnection() | logs message for TCP connection/handshake in which the local peer is the one making the connection | 
| void writeIncomingConnection() | logs message for TCP connection/handshake in which the local peer is the one receiving the connection | 
| void writeChangeNeighbors() | logs message for change of preferred neighbors (choking/unchoking) | 
| void writeOptimisticallyUnchoked() | logs message for change of optimistically unchoked neighbor | 
| void writeUnchokedByPeer() | logs message for being choked by the connected peer | 
| void writeChokedByPeer() | logs message for being unchoked by the connected peer | 
| void writeHaveMessage() | logs message when a peer receives a 'have' message for a piece from another peer | 
| void writeInterested() | logs message when a peer is 'interested' in another peer's pieces | 
| void writeNotInterested() | logs message when a peer is 'not interested' in another peer's pieces | 
| void writeDownloadPiece() | logs message when a peer has received and downloaded a piece from another peer | 
| void writeCompleteDownload() | logs message when a peer has finished downloading a file | 
| void writeReceivedBitfield() | logs message when a peer has received another peer's bitfield | 


## FileParser.java

Reads in a file

| Method | Description |
|----------|----------|
| byte[][] parse() | parses a file into its byte[][] representation  | 

## FileWriter.java

Writes a byte[][] to a file 

| Method | Description |
|----------|----------|
| void writeToFile(int peerID, byte[][] fileBytes, String fileName) | writes a byte[][] to the corresponding peer's directory | 

## ServerThread.java

Listens for incoming connections

| Method | Description |
|----------|----------|
| run() | handles incoming connections and creates PeerThreads for them. Adds them to the peerThreads list | 

## Peer.java

Represents a peer and its attributes

| Method | Description |
|----------|----------|
| getPeerIDs() | returns all peerIDs within the program | 
| addPeerIDs() | returns all peerIDs within the program | 

## Handshake.java
