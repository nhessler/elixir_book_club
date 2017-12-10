# 12 Building A Distributed System

* Erlang's distributed nature is not RPC
  * Erlang gets distibuted primitives early on
  * processes and messages on one machine can be considered distributed already


## 12.1

* `iex --sname node1@localhost` names the node you're about to start w/ iex
* `Kernel.node/0` gets you the name of the current node you're on
* `Node.connect/1` takes the symbol name of the node you'd like to connect to.
* `Node.list/0` shows all the nodes currently connected to your node.
* connecting to another node will also then try to create connections to other connected nodes
* regular `tick` messages are sent to connected nodes
* if any node fails 4 times to accept tick messages then they are considered disconnected
* `Node.monitor/1` to get info when a node disconnects
* you can watch all nodes with `:net_kernel.monitor_nodes/1,2`
* `Node.spawn/2` spins up a process on a specified node
* All standard I/O calls are forwarded to the *group leader*
* *location transparency* You can send a message to a process regardless of it's location
* when messages cross nodes erlang uses :erlang.term_to_binary/1 and :erlang.binary_to_term/1 to encode and decode them
* It's generally a bad idea to send lambdas across nodes.
* `send` can take a tuple of `{:alias, :node}` for sending to a registered process on another node.
* `:global` module can be used to cluster wide registration
* Process IDs account for nodes as all
  * when the X of the group <X.Y.Z> is 0 it's a local process
  * when the X of the group <X.Y.Z> is not 0 it's a remote process
* many distributed services are built in pure erlang.
  * global
  * pg2
  * rpc

## 12.2

* remember to add timeout constraints when talking across nodes
* Have a strategy for partial success or you'll be out of sync. inconsistent
* Network partions, netsplits, or "split brain" can be difficult to deal with
* from a node's perspective it doesn't matter why another node is disco'd and is not possible to discern.
* CAP Theorem you can have a CP system or an AP system
  * Consistency
  * Availability
  * Partition Tolerance


## 12.3
* `iex --name node1@127.0.0.1` allows for long names.
* `Node.get_cookie` and `Node.set_cookie` for managing cookies.
* cookies must match to get nodes connected.
* you can connect hidden and other nodes won't see you
* use EPMD (Erlang Port Mapper Daemon) to find other nodes
* nodes are started on random ports
  * `inet_dist_listen_min`
  * `inet_dist_listen_max`
* use the above to set port range.
* they can be the same effectively allowing only one node on machine
* erlang expects a trusted environment.
* limit user power appropriately
