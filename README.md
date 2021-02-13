# Storm - Node Collective Management Daemon on the Bitcoin Lightning Network

Storm is an open-source node coordination protocol and communication system for the Bitcoin Lightning Network using a directives-based proof-of-stake command-and-control system. Nodes in a particular Storm follow the Storm Protocol which has specific node participation requirements and constraints for the purposes of the smooth operation of the network. These constraints are designed to ensure:

* Efficiently balanced liquidity throughout the network
* Efficiently balanced connectivity throughout the network
* High uptimes and reliable connections
* Effective onboarding of new nodes/liquidity
* Effective re-routing over planned downtime or withdrawals

A trade-off for achieving these features is that in a given Storm, there is a loss of some of the privacy[a] afforded in an independent Lightning Node configuration. In order for proper coordination to occur, some data such as payment flow and channel balances are reported upstream to Storm management ("manager").

A Storm can be public or private. 

A private Storm could be used by a single corporation that needs to deploy many nodes. The privacy concerns between these nodes are not relevant since the company has full access to all the nodes anyways. A private Storm could also be a group of "trusted friends" who simply wish to help each other. Again, these friends might otherwise be willing to share private information about payments, liquidity, etc. even if Storm didn't exist in the first place. 

A public Storm is one that permits anyone to connect and become a member of the Storm (given they follow that Storm's specific rules).

In ad-hoc networks, both liquidity and connectivity can form in unorganized clusters. This can result in poorly distributed channels (e.g. "waste") where the liquidity is not utilized effectively, or there is not enough connectivity leading to constant network splits or unavailability.


A very common problem is that new lightning nodes have no ability to transfer funds. They have zero incoming and outgoing liquidity. With Storm, the collective helps to lift the new node by routing liquidity to that node automatically. Through the use of channel creation and channel rebalancing, the new node can begin participating in both sending/receiving reliably even without any funds to contribute.

In order to avoid these issues, Storm uses a messaging protocol that is used for collective coordination and excludes or permits a node into the Storm. 

## The Supercell Storm Instance

Anyone can create their own public or private storms and each storm has a storm name. Supercell is the default public group that uses the Storm protocol with the following privileges and rule set:

As a member node, you are guaranteed:

* Reliable connectivity and liquidity
* Shared fee rate based on your liquidity contribution to the network (proof-of-stake)
   * Unbalanced fees can cause strange bottlenecks in ad-hoc networks, so in Storm, all channel fees are set to the same base rate and fee rate dictated by the Storm manager. This is based on a quorum of voting for the fee you wish to have, and the # of votes you have is based on the % of liquidity you are adding to the network.
* Fee rate voting rights (proof-of-stake as your % contribution to total liquidity)

As a member node, you must follow the specified protocol messages which will dictate the following:

* When adding liquidity, you will query the Storm and be given back a "directive" that dictates channel creation (how, when, how much). The algorithm will query the participants channel liquidity status and payment flow to determine the best channels to be placed given the geographic location of the node, the amount of liquidity being added and network conditions.[b]
* Creating channels outside of the protocol directives is not permitted. For the benefit of the collective, new liquidity must always be added through directives.
* A "directive" may tell you to create 4 channels to A, B, C, and D with X amount of liquidity in each.
* If you need to remove liquidity, you can do this gracefully which will prevent your node from getting banned. In a graceful reduction, you will query the Storm and be told when you may close a channel. During this timeout phase, the network will be prioritizing new liquidity to route around your planned removal of liquidity.
* If a bad member node is being removed from the Storm based on incorrect activity, you must follow any directives to close channels with that node to effectively ban the node.
* All incoming/outgoing channel liquidity operations are controlled by the Storm.
* A "directive" may tell you to send a specific circular payment for rebalancing at any time.
* If your uptime is bad, you may get banned.
* If you need downtime to upgrade your server, you can query the Storm. The Storm will begin to route new liquidity around your node, and tell you when it's safe to disconnect.

The querying the network and performing all liquidity actions through it for ensured membership, a centralized algorithm can be used to determine the proper balance avoiding network bottlenecks and partitions which can cause payment delays or failures. Furthermore, by effectively using all the liquidity, measuring the payment flows real-time as well as using predictive modeling, the network can expand and grow in a way that is much more coordinated and effective.

In the long-term, there may be many competing Storm groups, but market forces might choose the best ones based on the privacy preferences and compensation they wish to have. Various dashboards can be made that compare the different Storm groups and even tell the potential users how much they can expect to earn for adding liquidity to a particular group.

Sample Groups Real-Time Comparison Page

Supercell (base fee: 2250 fee rate: 0.006932) Flow rate: 2,938 tps Expected earnings per 1m sat liquidity added: 3,870 sats/day

DrBitcoin's FunTime Group (base fee: 3038 fee rate: 0.000032) Flow rate: 33092 tps Expected earnings per 1m sat liquidity added: 4,022 sats/day
	
## Software

Running the stormd server performs the actions necessary to participate in the Storm protocol. It also includes a stormcli client to perform requests such as receiving liquidity directives.

A sample stormd.conf file:

```
network=mainnet
node=lnd
nodeMacaroon=/home/user/.lnd/admin.macaroon
nodeRpcPort=3600
group=Supercell         # Specify which group to connect to
voteBaseFee=1000        # Your proposed base fee (needs consensus)
voteFeeRate=0.000001    # Your proposed fee rate (needs consensus)
logLevel=debug
logFile=/home/user/.stormd/debug.log
```	

The stormd server communicates with lnd[c] over RPC to manage the operation of the node.

### Example stormcli commands:

```
$ stormcli addfunds 2000000
Your request to add 2,000,000 sats has been submitted. Be sure your lnd wallet has sufficient funds. The stormd server will automatically create the channels necessary to utilize those funds. To cancel this request, run `stormcli cancel jX78nVt`. To see a list of all requests, run `stormcli getinfo`.


$ stormcli disconnect
Your request to disconnect from the group Supercell has been submitted. All funds will be de-allocated. Please wait while liquidity is re-routed around your node. If you disconnect prematurely, you may become banned from the group.


$ stormcli connect Supercell
Your request to connect to the group has been submitted. To see the status of your request, run `stormcli getinfo`.


$ stormcli removefunds 2000000
Your request to de-allocate 2,000,000 sats has been submitted. The stormd server will automatically shut down channels and re-route accordingly. To check the status of your de-allocation, run `stormcli getinfo`.


$ stormcli getinfo
{
  "version": "Storm release v0.1.0",
  "group": "Supercell",
  "members": 3082,
  "connected": true,
  "uptime": "3d 22h 11m 4s",
  "consensus_fee_base": 2500,
  "consensus_fee_rate": 0.002250,
  "rating": 100,
  "total_allocations": 12300000,
  "pending_allocations": [
    {
      "id": "jX78nVt",
      "amount": 2000000,
    }
  ],
  "pending_deallocations": [
    {
    }
  ],
  "pending_directives": [
    {
      "action": "do_not_disconnect"
    }
  ]
}


$ stormcli help
```

There may be other commands not listed here.

## Pitfalls and Issues

TBD

* [a]Side note: privacy level can be configured, e.g. ignore flow data
* [b]This is the key feature of Storm
* [c]No reason why the protocol software couldn't support many implementations actually.
