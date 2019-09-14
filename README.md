Running a private Neo network for testing is vital for development of smart contracts and dApps. TestNet GAS is scarce and the TestNet is often under load by other people's stress tests, so it makes sense to do most of your initial testing in a local setting, where you own all the NEO and GAS and there are no other transactions besides your own.

Previously your choices for setting up a privatenet were to run 4 different neo-cli consensus nodes on separate hardware, or using Docker to run it all inside of containers - which introduces its own learning curve and challenges. This tradeoff can be worth it in certain environments, but in others it may be introducing more problems than it solves.

This tutorial describes how to set up a Neo "singlenet", which is what I call a privatenet that consists of only a single consensus node. This offers the lowest-possible CPU and memory requirements for running on a single machine for test purposes.

### Prequisites:

* [neo-cli binary package for your operating system](https://github.com/neo-project/neo-cli/releases)
* [.Net Core 2.2 runtime (or SDK if you want to rebuild from source)](https://dotnet.microsoft.com/download)
* [Singlenet protocol.json file](https://raw.githubusercontent.com/hal0x2328/neo-singlenet-tutorial/master/protocol.singlenet.json)
* [Singlenet config.json file](https://raw.githubusercontent.com/hal0x2328/neo-singlenet-tutorial/master/config.singlenet.json)
* [Singlenet consensus/owner wallet file](https://raw.githubusercontent.com/hal0x2328/neo-singlenet-tutorial/master/wallet-singlenet.json)

### Steps

1. Install the .NET runtime/SDK according to instructions from Microsoft on
your platform
2. Unzip the neo-cli package to a folder of your choice
3. Open a terminal prompt and cd to that folder
4. Copy the protocol.singlenet.json file above to protocol.json
5. Copy the config.singlenet.json file above to config.json
6. Copy the wallet JSON file into the same folder

### Run the node 

On Windows or Linux:
```
dotnet ./neo-cli.dll /rpc
```
On OSX:
```
DYLD_INSERT_LIBRARIES=/usr/local/lib/libtcmalloc.dylib dotnet ./neo-cli.dll /rpc
```

You should see the consensus node start and display its version. Note that you won't actually see the usual `neo>` prompt prefix when in consensus mode, but the node will still accept typed commands. 

Use the command `show state` to watch the block height increase. The first of the three numbers at the top of the window is the wallet height. The second number is the database height, and the third number is the blockchain height. Since this is the only node on the network, and the wallet is open by default, all three numbers should be in close sync with each other.

Now you can view the wallet balance running the command ```list asset``` in the
prompt:

```
list asset
       id:0xc56f33fc6ecfcd0c225c4ab356fee59390af8560be0e930faebe74a6daff7c9b
     name:NEO
  balance:100000000
confirmed:100000000
```

Here you can see that you own 100,000,000 NEO, but no GAS yet. After a couple of minutes you will have aquired enough GAS to deploy a smart contract, but you will have to issue a claim transaction for it, which is actually a two-step process - first you send all your NEO to yourself, then you make a GAS claim. These are the steps:
```
send neo ASingLeneTRyh8UaPSAQTmNgBYTxY6ZM8N all
```
Enter the password `one`, wait for the next block, then run:

```
claim gas all
```

If everything worked you should see `Transaction Succeeded`, and running 
`list asset` again will show that you now have enough GAS to deploy a smart 
contract. It runs out quickly however, so you'll want to do the `send neo/claim gas` steps again later after the node height has passed a few thousand blocks. Repeat the process again whenever you need more GAS.

You may want to edit `protocol.json` and change the `SecondsPerBlock` parameter - initially it is set to 2 seconds per block, in order to allow you to accumulate GAS more quickly. However, if you intend to keep your singlenet online 24x7, having a fast block rate will use much more hard drive space and will take longer to sync new wallets or clients with the network. 

The default block interval for the Neo MainNet and TestNet is 15 seconds, but that can be a little slow when you are testing a smart contract and waiting for transactions to persist to the chain. A value between 5 and 10 seconds for the `SecondsPerBlock` parameter is a good tradeoff for speed vs storage/time needed to sync.

Now your singlenet is ready for testing contracts and dApps. You can send and
receive information from the node using the JSON-RPC protocol on TCP port 30332,
or, by running a neo-python node which has more advanced capability for deploying and testing smart contracts.

### Connecting neo-python to your singlenet

To install neo-python, follow the instructions on the neo-python Github repo, then, download these two files: 

* [np-protocol.singlenet.json](https://raw.githubusercontent.com/hal0x2328/neo-singlenet-tutorial/master/np-protocol.singlenet.json)
* [np-wallet-singlenet.db3](https://github.com/hal0x2328/neo-singlenet-tutorial/raw/master/np-wallet-singlenet.db3)

If you are running neo-python on the same computer as the neo-cli singlenet node, you can simply copy these files to your neo-python folder, and run neo-python using the command:
```
np-prompt -c np-singlenet.protocol.json
```
Otherwise you will need to edit np-singlenet.protocol.json to replace 127.0.0.1 with the IP address or hostname of the computer where the singlenet node is running, making sure that no firewalls are blocking your connection.

Once you are up and running with neo-python, you can open the wallet using the command:
```
wallet open np-singlenet.wallet.db3
```
Note that the order of the command words is different in neo-python than neo-cli, i.e. `wallet open` vs `open wallet`. However the password for both wallets in this tutorial is the same, and that password is: *one*

Refer to the [neo-python documentation and tutorials](https://neo-python.readthedocs.io/en/latest/) for command-line usage and how to get started developing smart contracts and dApps on Neo. 
