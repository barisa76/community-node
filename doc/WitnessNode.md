To the best of my knowledge, this is the way to set up a TRON witness node. This is a living document and it is updated frequently.

Most of us will be setting up our nodes in the cloud, on a VM slice. There's a video for getting started on [Google Cloud](https://www.youtube.com/watch?v=AN9YwX7PqgY) or you can use an indie host like Rackspace or Linode. Rackspace is rock but be careful about over the limit bandwidth charges. 

**STEP #1 Register an address**

Go to [tronscan.org](https://tronscan.org/#/) and register. Clicking the red bar at the top of the page will create a new test wallet address, password and private key. Copy and save them to a txt file. You can then use the password to login and request 1,000,000 test TRX to be sent to your wallet address. If you want to run the wallet-web app it will now show you the same balance, but you can only request tokens from the tronscan.org server. The test net has a good amount of functionality now, you can use those test TRX to vote for delegates, to create tokens, and to send to other TRX test addresses.

**STEP #2 Install Java 8**

Even though there is a Java 9 and Java 10 available, use JDK 8. You'll need several GB of free space to unpack java, and even more free space so the cache buildup doesn't crash the server every day. 

For a **Windows** install you can follow [these instructions](https://docs.google.com/document/d/1RsRx_NrWsySP6EthUzYtdEGkAWg7j29UTHOxRhFgfVY) to run the INTELLIJ app which will take you up to the "this is just a node" line in this document.

For **Mac O/S**, see if you have java installed. Go to Applications/Utilities/Terminal. You'll be needing it later but for now learn a neat Mac trick. Type:
    
    open /Library/Java/JavaVirtualMachines
In case you didn't notice, it opened that folder in the finder. If you have anything other than a jdk1.8.0_172.jdk folder there, either delete it (them) or move to the desktop until you're done with this project. Get and install the [jdk-8u172-macosx-x64.dmg](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

For **Debian or Ubuntu**, After the initial o/s install, you want to update with any new patches

    apt-get update && apt-get -y upgrade

Add the java repository to the apt-cache
    add-apt-repository ppa:webupd8team/java
    apt-get update

We'll install needed packages & helpful tools

    apt-get -y -V install build-essential git git-core locate curl libcurl4-openssl-dev wget javascript-common libjs-jquery libcap2-bin software-properties-common unzip zip unattended-upgrades libcap2-bin

Then, for the love of all that is holy, set security updates to automatically install.

    echo 'APT::Periodic::Update-Package-Lists "1";' >> /etc/apt/apt.conf.d/20auto-upgrades
    echo 'APT::Periodic::Unattended-Upgrade "1";' >> /etc/apt/apt.conf.d/20auto-upgrades

Add the java repository to the apt-cache

    add-apt-repository ppa:webupd8team/java
    apt-get update
    apt-get -y -V install oracle-java8-installer npm --allow-unauthenticated

The node.js that comes with the standard debian package is insufficient. Remove it, and then download the full package.

    apt-get remove nodejs
    apt-get remove npm	
    cd /tmp
    curl -sL https://deb.nodesource.com/setup_6.x | sudo bash -
    apt-get install -y nodejs
    apt-get install -y npm

Install the grpc because I think it's needed for protobuf

    npm install grpc


**THE REST OF THIS DOC IS THE SAME FOR ALL O/S** Although it's written for linux users. If you're on a Mac you ought to be able to figure out how to copy, edit and delete files.

**STEP #3 Install Java-Tron**

From this point on, you should not need to run any commands as 'sudo'.

Clone the tron repository to the local machine, then change into the tron directory and build the machine

    git clone https://github.com/tronprotocol/java-tron.git
    cd java-tron
    ./gradlew build
This will create a folder named 'build'. There are 3 ways to launch the java-tron machine. The first and second are for all intents and purposes identical. The third uses a GUI program. I like #1 best:

    ./gradlew run

This is just a node. The next step is to set up a witness node using the information you got when you registered at tronscan in step 1. You can choose to copy the original config.conf file in java-tron/src/main/resources to the root of the application and edit that. This way you don't end up changing any of the repository files and you don't have to do any git trickery to do another pull.

    cp src/main/resources/config.conf .
    nano config.conf    
There is only one key piece of information to enter here - the private key in the localwitnesses block. However if you don't specify your IP address it will have to probe for it on startup and that takes a few seconds, so I like to hardcode the IP information. Google cloud users will have two IP addresses. It may work best by configuring separate IP's like this. Use actual IP addresses. I have both of those configured for my one public IP even though I also have an internal subnet IP. 

**config.conf changes**

    node.discovery = {
      enable = true
      persist = true
      bind.ip = "INTERNAL.IP.ADDRESS"
      external.ip = "EXTERNAL.IP.ADDRESS"
    }

--------------------------

	localwitness = [
	ABCDEF1234567890ABCDEF1234567890ABCDEF1234567890ABCDEF1234567890
	]




O.K. now when the tron machine starts up it's not just running a node, it's running *your* node.

If you've made any changes when you restart ~/java-tron, you'll need to throw away the database directory. While you're at it, start the logs fresh.

    cd ~/java-tron
    rm -Rf output-directory/database
    rm -Rf logs/*
    ./gradlew run -Pwitness=true

The TRON Developers recommend calling the jar file directly. This cuts down the system processes from 3 to 2.

    cd ~/java-tron/build/libs
    java -jar java-tron.jar




They say don't put your private key in the config file, call it as a parameter when you're starting the application, like this:

    cd build/libs
    java -jar java-tron.jar -p private_key --witness -c config_path

**Example:**

    java -jar java-tron.jar -p 650950B193DDDDB35B6E48912DD28F7AB0E7140C1BFDEFD493348F02295BD812 --witness -c /data/java-tron/config.conf

    java -jar java-tron.jar -p --witness 

**LOG FILE EXAMPLES**

The first lines of logs/tron.log should be 

    Full node running.
    Refreshing org.springframework.context.annotation.AnnotationConfigApplicationCote [Thu Apr 26 18:47:17 EDT 2018]; root of context hierarchy
    Transaction create succeeded！

    Unknown channel option 'SO_KEEPALIVE' for channel '[id: 0x9ec67546]' 
    Reading Node statistics from PeersStore: 0 nodes. 
    Add new node: NodeHandler[state: Discovered, node:  
    Change node: old NodeHandler[state: Discovered, node:  

Then it will scroll through adding nodes from the network

    Add new node: NodeHandler[state: Discovered, node: 93.35.144.199:3484, id=609cd88e], size=19
    Add new node: NodeHandler[state: Discovered, node: 109.27.36.86:50196, id=61c5b1bc], size=20

Finally it looks like you've got something, only to be disappointed in the details:

    Peer stats:
    Active peers
    Other connected peers
    
    LocalNode stats:
    MyHeadBlockNum: 0
    advToSpreadNum: 0
    advObjectToFetchNum: 0
    advObjWeRequestedNum: 0
    unSyncNum: 0
    blockWaitToProcess: 0
    blockJustReceived: 0
    syncBlockIdWeRequested: 0
    badAdvObj: 0

Eventually you'll connect to 47.91.246.252:18888 which seems to be the most trusted peer.
    
    Other connected peers
    Peer 47.91.246.252:18888: [           aed3688f, ping    215 ms]
    connect time: 1969-12-31 19:00:00.0
    last know block num: 0
    needSyncFromPeer:true
    needSyncFromUs:false
    syncToFetchSize:0
    syncToFetchSizePeekNum:-1
    syncBlockRequestedSize:0
    unFetchSynNum:0
    syncChainRequested:2018-04-26 18:49:10.483
    blockInPorc:0
    NodeStat[reput: 390(0), discover: 1/1 0/0 5/5 3/3 229ms, p2p: 1/1/1 , tron: 3/2   
    
    Handle Message: INVENTORY: First hash: 3d0f8c2e68d89082f2d53222c6debdeb3254eba7dd5f6ab96946e60bdb39c588 End hash: 3d0f8c2e68d89082f2d53222c6debdeb3254eba7dd5f6ab96946e60bdb39c588 from 
    Peer: aed3688f52718c895d3181eb8223f6556f0689f6515862fb08e70200b5970aae7f6c97fc304946630db595c3f9d75a5e056496045e536dc55a1a143ccc49925d | /47.91.246.252:18888
    channel active, /47.91.246.252:47240
    Finish handshake with /47.91.246.252:47240.
    rcv BLOCK_CHAIN_INVENTORY from /47.91.246.252:18888
    Handle Message: BLOCK_CHAIN_INVENTORY : first blockId: Num:0,ID:00000000000000006aaae11afeca9b84f031ef5e45c311ac302eacd4c6cbb73b end blockId: Num:500,ID:00000000000001f40ae2b2aae1b96655e4d614fdd3e927cee636968b06651300 size: 501 from 
    Peer: aed3688f52718c895d3181eb8223f6556f0689f6515862fb08e70200b5970aae7f6c97fc304946630db595c3f9d75a5e056496045e536dc55a1a143ccc49925d | /47.91.246.252:18888
    update peer 47.91.246.252 block both we have, Num:0,ID:00000000000000006aaae11afeca9b84f031ef5e45c311ac30
    2eacd4c6cbb73b
    headNumber:0
    syncBeginNumber:0
    solidBlockNumber:0
    headNumber:0
    syncBeginNumber:0
    solidBlockNumber:0
    Send Message:SYNC_BLOCK_CHAIN: 
    Num:0,ID:00000000000000006aaae11afeca9b84f031ef5e45c311ac302eacd4c6cbb73b
    Num:251,ID:00000000000000fbd7da930cb3b20ba7235cda24f51b48d73cb27534ec8300ee
    Num:376,ID:0000000000000178201326b872ec527841e9a2b9d64837800eb2179a77be691b
    Num:439,ID:00000000000001b7ccda2628b8f2f7841be1251be901d8b7465be28ff21ef861
    Num:470,ID:00000000000001d6dca3061341dbd59f6a84074691dc9d88e452a55e4254dba4
    Num:486,ID:00000000000001e62adc9c52ad1281dbc862fa54e1770589fb7d49d227f03e1d
    Num:494,ID:00000000000001eee197065f8628548a675e80ac7762118eb1a506af21f336c4
    Num:498,ID:00000000000001f2182f8ff99eddf21df25968a11ca2c4f77bf3986e38ffd612
    Num:500,ID:00000000000001f40ae2b2aae1b96655e4d614fdd3e927cee636968b06651300 to
    aed3688f52718c895d3181eb8223f6556f0689f6515862fb08e70200b5970aae7f6c97fc304946630db595c3f9d75a5e056496045e536dc55a1a143ccc49925d | /47.91.246.252:18888
    18:49:10.963 INFO  [MessageQueueTimer-1] [MessageQueue](MessageQueue.java:186) send SYNC_BLOCK_CHAIN to /47.91.246.252:18888

After some kind of handshake the trusted node goes from 'Other connected peers' to 'Active peers'

    Active peers
    Peer 47.91.246.252:18888: [           aed3688f, ping    271 ms]
    connect time: 2018-04-26 18:49:20.459
    last know block num: 0
    needSyncFromPeer:true
    needSyncFromUs:false
    syncToFetchSize:0
    syncToFetchSizePeekNum:-1
    syncBlockRequestedSize:0
    unFetchSynNum:0
    syncChainRequested:2018-04-26 18:49:20.462
    blockInPorc:0
    NodeStat[reput: 180(0), discover: 1/1 0/0 5/5 3/3 229ms, p2p: 2/2/2 , tron: 8/4 X 1<=DUPLICATE_PEER 

Finally we start to sync blocks

    There are 31922 blocks we need to sync.
    
    handle Block number is 1
    Handle Message: [Message Type: BLOCK, Message Hash:0000000000000002ecb773a55f1f0311372e34e47749af06ed973e37299b1159] from Peer: aed3688f52718c895d3181eb8223f6556f0689f6515862fb08e70200b5970aae7f6c97fc304946630db595c3f9d75a5e056496045e536dc55a1a143ccc49925d | /47.91.246.252:18888

Once you have caught up to the blockchain you'll mostly just see transactions:

    1:TransactionCapsule 
    [ hash=51d9db7a72aa66886b8886d2352f600c67acf14c8a39fa9639491342d268b802
    contract list:{ [0] type: TransferContract
    from address=[B@d6a90e1
    to address=[B@62622a52
    transfer amount=8000000
    sign=G16nepaM8mBg//Yp06AZocYlYxSz8XIc6+d9hsn12r3GIVTWiP7mOPGhz8VziZ/4+107JQCZTY6N29ICH6IIj+4=
    }
    ]


Finally you'll receive notice of your place in the queue

    save block num:33716
    update peer 47.91.246.252 block both we have Num:33716,ID:00000000000083b46d4a63d6cbda07f0fd5ce620230545c629b
    scheduledWitness:a00a9309758508413039e4bc5a3d113f3ecc55031d, currentSlot:304959039

Here's a command to extract the display of discovered nodes.

    more ~/java-tron/logs/tron.log | grep nodes.
	
> Write Node statistics to PeersStore: 2375 nodes.



Next article - Adding a Solidity Node





03:14:24.478 INFO  [nioEventLoopGroup-4-2] [o.t.c.n.n.NodeImpl](NodeImpl.java:1077) update peer 104.237.3.78 block both we have Num:215999,ID:0000000000034bbf8af84f11623bd1cbc6f8eb43caafbc08d35e05e59fb60c7f
03:14:27.269 INFO  [nioEventLoopGroup-4-2] [o.t.c.d.DynamicPropertiesStore](DynamicPropertiesStore.java:552) update latest block header id = 0000000000034bc061fcd76d5350d4992dc44f38513ae19800525ca9a8e80abc
03:14:27.269 INFO  [nioEventLoopGroup-4-2] [o.t.c.d.DynamicPropertiesStore](DynamicPropertiesStore.java:544) update latest block header number = 216000
03:14:27.269 INFO  [nioEventLoopGroup-4-2] [o.t.c.d.DynamicPropertiesStore](DynamicPropertiesStore.java:536) update latest block header timestamp = 1526627667000
03:14:27.271 INFO  [nioEventLoopGroup-4-2] [o.t.c.d.Manager](Manager.java:1033) update solid block, num = 215992
03:14:27.274 INFO  [nioEventLoopGroup-4-2] [o.t.c.d.Manager](Manager.java:714) save block: BlockCapsule 
03:14:27.276 INFO  [nioEventLoopGroup-4-2] [o.t.c.n.n.NodeImpl](NodeImpl.java:1077) update peer 54.219.41.56 block both we have Num:216000,ID:0000000000034bc061fcd76d5350d4992dc44f38513ae19800525ca9a8e80abc
03:14:27.276 INFO  [nioEventLoopGroup-4-2] [o.t.c.n.n.NodeImpl](NodeImpl.java:259) Ready to broadcast a block, Its hash is 0000000000034bc061fcd76d5350d4992dc44f38513ae19800525ca9a8e80abc
last know block num: 216000
blockInPorc:0
03:14:30.541 INFO  [TronJClientWorker-1] [o.t.c.d.DynamicPropertiesStore](DynamicPropertiesStore.java:552) update latest block header id = 0000000000034bc10c79b059d95152c5a65b0b3415dcafc7685418c9db9c7b00
03:14:30.541 INFO  [TronJClientWorker-1] [o.t.c.d.DynamicPropertiesStore](DynamicPropertiesStore.java:544) update latest block header number = 216001
03:14:30.541 INFO  [TronJClientWorker-1] [o.t.c.d.DynamicPropertiesStore](DynamicPropertiesStore.java:536) update latest block header timestamp = 1526627670000
03:14:30.550 INFO  [TronJClientWorker-1] [o.t.c.n.n.NodeImpl](NodeImpl.java:259) Ready to broadcast a block, Its hash is 0000000000034bc10c79b059d95152c5a65b0b3415dcafc7685418c9db9c7b00
blockWaitToProc: 0
blockJustReceived: 0
03:14:36.604 INFO  [nioEventLoopGroup-4-2] [MessageQueue](MessageQueue.java:94) rcv FETCH_INV_DATA from /104.237.3.78:34706
03:14:36.604 INFO  [nioEventLoopGroup-4-2] [o.t.c.n.n.NodeImpl](NodeImpl.java:205) Handle Message: FETCH_INV_DATA:BLOCK, size=1, First hash:0000000000034bc3731c20f373c0e3949ffa262607c1a9f9849f7e0a7c5c778b from 
Peer: /104.237.3.78:34706 | 5b3be0c2f7a89af0bbb62a25570630824f7dfef5d0925cf0c546b355c6740f16494bd35762781a95c901beb9af70cdec1f9dba041a11786dd7e181f31f5bbef6
03:14:36.605 INFO  [nioEventLoopGroup-4-2] [MessageQueue](MessageQueue.java:86) send BLOCK to /104.237.3.78:34706
03:14:36.615 INFO  [nioEventLoopGroup-4-1] [MessageQueue](MessageQueue.java:94) rcv BLOCK from /18.188.227.130:57404
03:14:36.616 INFO  [nioEventLoopGroup-4-1] [o.t.c.n.n.NodeImpl](NodeImpl.java:205) Handle Message: [Message Type: BLOCK, Message Hash: 0000000000034bc3731c20f373c0e3949ffa262607c1a9f9849f7e0a7c5c778b] from 
Peer: /18.188.227.130:57404 | 55622281e7aac3be8470999e5a52d80fcdf26e4cabce872b39bfb4933973f06204acdb276136096aae5ac6c03e8c12124c1cab497fc1d286ae9b2a6b8f57805e
03:14:36.617 INFO  [nioEventLoopGroup-4-1] [o.t.c.n.n.NodeImpl](NodeImpl.java:617) handle Block number is 216003
03:14:36.634 INFO  [nioEventLoopGroup-4-1] [MessageQueue](MessageQueue.java:94) rcv P2P_PING from /139.99.173.27:55810
03:14:36.634 INFO  [nioEventLoopGroup-4-1] [MessageQueue](MessageQueue.java:86) send P2P_PONG to /139.99.173.27:55810
03:14:36.639 INFO  [nioEventLoopGroup-4-1] [MessageQueue](MessageQueue.java:94) rcv P2P_PONG from /139.99.173.27:55810
03:14:36.685 INFO  [TronJClientWorker-1] [MessageQueue](MessageQueue.java:94) rcv INVENTORY from /13.125.95.134:18888
03:14:36.685 INFO  [TronJClientWorker-1] [o.t.c.n.n.NodeImpl](NodeImpl.java:205) Handle Message: INVENTORY:BLOCK, size=1, First hash:0000000000034bc3731c20f373c0e3949ffa262607c1a9f9849f7e0a7c5c778b from 
Peer: /13.125.95.134:18888 | 9c1ce86bbad3e1e40cdc848d0cbbe1aace56aff67520afdd5e9745ee559f4360f72bc0f374c6fba58e23aa48f67b1fcb9ac5909edc015191c9593cfbcc144e09
03:14:37.854 INFO  [pool-8-thread-1] [SyncPool](SyncPool.java:118) address: 195.201.32.0:18888, ID:ed31ce4c NodeStat[reput: 351(0), discover: 27/27 0/0 4963/4963 2453/2453 117ms, p2p: 20/20/20 , tron: 20/0 X 0  
03:14:37.857 INFO  [pool-8-thread-1] [SyncPool](SyncPool.java:124) -------- active channel 20, node in user size 20
03:14:37.857 INFO  [pool-8-thread-1] [SyncPool](SyncPool.java:127) /47.254.16.55:18888 | e89ec4a92ab927175caf3e42241375289ceb63e9a14b23edb6a04085b247fdc90a0b883214bf8af157a51109740cefb9e4157f420e19b1e9211992cc715130db
03:14:37.863 INFO  [pool-8-thread-1] [SyncPool](SyncPool.java:146) Peer stats:
Active peers
============
Peer 47.98.58.42:18888: [           bf620d79, ping    250 ms]-----------
connect time: 2018-05-18 02:58:17.068
last know block num: 216001
needSyncFromPeer:false
needSyncFromUs:false
syncToFetchSize:0
syncToFetchSizePeekNum:-1
syncBlockRequestedSize:0
unFetchSynNum:0
syncChainRequested:NULL
blockInPorc:0
NodeStat[reput: 251(0), discover: 1/1 1/1 0/0 4858/4858 212ms, p2p: 10/10/12 , tron: 20039/17839 X 0  

Other connected peers
============
03:14:38.174 INFO  [TronJClientWorker-1] [MessageQueue](MessageQueue.java:94) rcv INVENTORY from /47.254.16.55:18888
03:14:38.174 INFO  [TronJClientWorker-1] [o.t.c.n.n.NodeImpl](NodeImpl.java:205) Handle Message: INVENTORY:TRX, size=1, First hash:957eb5ca18f233a655bf3e51bfb3bef8497f86a0af54a2ec46d552799a99a976 from 
Peer: /47.254.16.55:18888 | e89ec4a92ab927175caf3e42241375289ceb63e9a14b23edb6a04085b247fdc90a0b883214bf8af157a51109740cefb9e4157f420e19b1e9211992cc715130db
03:14:38.179 INFO  [broad-msg-] [MessageQueue](MessageQueue.java:86) send FETCH_INV_DATA to /35.230.28.129:58016
03:14:38.247 INFO  [nioEventLoopGroup-4-1] [MessageQueue](MessageQueue.java:94) rcv TRXS from /35.230.28.129:58016
03:14:38.247 INFO  [nioEventLoopGroup-4-1] [o.t.c.n.n.NodeImpl](NodeImpl.java:205) Handle Message: trx_size:1 from 
Peer: /35.230.28.129:58016 | 7a3ca9de7ab064f047eeb5a7ab262094785cb4d78efde426251f7e7bba635ae096d2e48c9bde3a5b6bfc35749733660ad3be471dbde4220dc176adbb4814587b
03:14:38.247 INFO  [nioEventLoopGroup-4-1] [o.t.c.n.n.NodeImpl](NodeImpl.java:770) onHandleTransactionsMessage, size = 1, peer 35.230.28.129
03:14:38.247 INFO  [nioEventLoopGroup-4-1] [o.t.c.n.n.NodeDelegateImpl](NodeDelegateImpl.java:81) handle transaction
03:14:38.247 INFO  [nioEventLoopGroup-4-1] [o.t.c.d.Manager](Manager.java:427) push transaction

