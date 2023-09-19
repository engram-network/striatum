<h1 align="center"><strong>Under Developments</strong></h1>

<h1 align="center"><strong>Striatum</strong></h1>
Official Golang execution layer implementation of the Engram protocol. 

Automated builds are available for stable releases and the unstable master branch. Binary
archives are published at https://str.engram.tech/downloads/.

## Building the source

Building `striatum` requires both a Go (version 1.19 or later) and a C compiler. You can install
them using your favourite package manager. Once the dependencies are installed, run

```shell
make striatum
```

or, to build the full suite of utilities:

```shell
make all
```

## Executables

The Striatum project comes with several wrappers/executables found in the `cmd`
directory.

|  Command   | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| :--------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`striatum`** | Our main Engram CLI client. It is the entry point into the Engram network (main-, test- or private net), capable of running as a full node (default), archive node (retaining all historical state) or a light node (retrieving data live). It can be used by other processes as a gateway into the Engram network via JSON RPC endpoints exposed on top of HTTP, WebSocket and/or IPC transports. `striatum --help` and the [CLI page]() for command line options.

## Running `Striatum`

Going through all the possible command line flags is out of scope here,
but we've enumerated a few common parameter combos to get you up to speed quickly
on how you can run your own `striatum` instance.

### Hardware Requirements

Minimum:

* CPU with 2+ cores
* 4GB RAM
* 1TB free storage space to sync the Mainnet
* 8 MBit/sec download Internet service

Recommended:

* Fast CPU with 4+ cores
* 16GB+ RAM
* High-performance SSD with at least 1TB of free space
* 25+ MBit/sec download Internet service

### Full node on the main Engram network

By far the most common scenario is people wanting to simply interact with the Engram
network: create accounts; transfer funds; deploy and interact with contracts. For this
particular use case, the user doesn't care about years-old historical data, so we can
sync quickly to the current state of the network. To do so:

```shell
$ striatum console
```

This command will:
 * Start `striatum` in snap sync mode (default, can be changed with the `--syncmode` flag),
   causing it to download more data in exchange for avoiding processing the entire history
   of the Ethereum network, which is very CPU intensive.
 * Start the built-in interactive [JavaScript console](),
   (via the trailing `console` subcommand) through which you can interact using [`web3` methods](https://github.com/ChainSafe/web3.js/blob/0.20.7/DOCUMENTATION.md) 
   (note: the `web3` version bundled within `striatum` is very old, and not up to date with official docs),
   as well as `striatum`'s own [management APIs](https://geth.ethereum.org/docs/interacting-with-geth/rpc).
   This tool is optional and if you leave it out you can always attach it to an already running
   `striatum` instance with `striatum attach`.

### A Full node on the Tokio test network

Transitioning towards developers, if you'd like to play around with creating Ethereum
contracts, you almost certainly would like to do that without any real money involved until
you get the hang of the entire system. In other words, instead of attaching to the main
network, you want to join the **test** network with your node, which is fully equivalent to
the main network, but with play-Ether only.

```shell
$ striatum --engram console
```

The `console` subcommand has the same meaning as above and is equally
useful on the testnet too.

Specifying the `--engram` flag, however, will reconfigure your `striatum` instance a bit:

 * Instead of connecting to the main Engram network, the client will connect to the Görli
   test network, which uses different P2P bootnodes, different network IDs and genesis
   states.
 * Instead of using the default data directory (`~/.striatum` on Linux for example), `striatum`
   will nest itself one level deeper into a `tokio` subfolder (`~/.striatum/goerli` on
   Linux). Note, on OSX and Linux this also means that attaching to a running testnet node
   requires the use of a custom endpoint since `striatum attach` will try to attach to a
   production node endpoint by default, e.g.,
   `striatum attach <datadir>/tokio/geth.ipc`. Windows users are not affected by
   this.

*Note: Although some internal protective measures prevent transactions from
crossing over between the main network and test network, you should always
use separate accounts for play and real money. Unless you manually move
accounts, `striatum` will by default correctly separate the two networks and will not make any
accounts available between them.*

### Programmatically interfacing `striatum` nodes

As a developer, sooner rather than later you'll want to start interacting with `striatum` and the
Ethereum network via your own programs and not manually through the console. To aid
this, `striatum` has built-in support for a JSON-RPC based APIs.
These can be exposed via HTTP, WebSockets and IPC (UNIX sockets on UNIX based
platforms, and named pipes on Windows).

The IPC interface is enabled by default and exposes all the APIs supported by `striatum`,
whereas the HTTP and WS interfaces need to manually be enabled and only expose a
subset of APIs due to security reasons. These can be turned on/off and configured as
you'd expect.

HTTP based JSON-RPC API options:

  * `--http` Enable the HTTP-RPC server
  * `--http.addr` HTTP-RPC server listening interface (default: `localhost`)
  * `--http.port` HTTP-RPC server listening port (default: `8545`)
  * `--http.api` API's offered over the HTTP-RPC interface (default: `eth,net,web3`)
  * `--http.corsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
  * `--ws` Enable the WS-RPC server
  * `--ws.addr` WS-RPC server listening interface (default: `localhost`)
  * `--ws.port` WS-RPC server listening port (default: `8546`)
  * `--ws.api` API's offered over the WS-RPC interface (default: `eth,net,web3`)
  * `--ws.origins` Origins from which to accept WebSocket requests
  * `--ipcdisable` Disable the IPC-RPC server
  * `--ipcapi` API's offered over the IPC-RPC interface (default: `admin,debug,eth,miner,net,personal,txpool,web3`)
  * `--ipcpath` Filename for IPC socket/pipe within the datadir (explicit paths escape it)

You'll need to use your own programming environments' capabilities (libraries, tools, etc) to
connect via HTTP, WS or IPC to a `striatum` node configured with the above flags and you'll
need to speak [JSON-RPC](https://www.jsonrpc.org/specification) on all transports. You
can reuse the same connection for multiple requests!

**Note: Please understand the security implications of opening up an HTTP/WS based
transport before doing so! Hackers on the internet are actively trying to subvert
Ethereum nodes with exposed APIs! Further, all browser tabs can access locally
running web servers, so malicious web pages could try to subvert locally available
APIs!**

## License

The Striatum library (i.e. all code outside of the `cmd` directory) is licensed under the
[GNU Lesser General Public License v3.0](https://www.gnu.org/licenses/lgpl-3.0.en.html),
also included in our repository in the `COPYING.LESSER` file.

The Striatum binaries (i.e. all code inside of the `cmd` directory) are licensed under the
[GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html), also
included in our repository in the `COPYING` file.
