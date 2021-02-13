# Optimizer

The **Optimizer** is the algorithm that attempts to find the best allocation plan for the existing network among storm nodes. The key factors are:

* Traffic Pattern (Amount Transacted on a Daily Basis)
* Existing Channels
* Geographic Location
* Response Time (ms Ping)
* Availability (% Uptime)
* Available funds for channel creation (Unspent coins)

When a new node joins the collective, the collective grants a small amount of inbound and outbound liquidity to the new node. This amount is increased over time based on how frequently funds are moving through the channel.

## Traffic Pattern Factor

Each channel has a traffic IN and traffic OUT factor (Tin, Tout) between 0 and 1 where 0 is no traffic and 1 is a high amount of traffic relative to the rest of the network. The value is calculated as D/T where D is the number of satoshis transferred in the last 24 hours and T is the total transfers on the network. If a Tin value is high, the network will allocate more inbound liquidity to be allocated towards that node. If a Tout value is high, the network will allocate more outbound liquidity from that node.

## Existing Channels Factor

