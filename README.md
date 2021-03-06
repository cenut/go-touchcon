## GO TOUCHCON

Official golang implementation of the Ethereum-based TouchCon protocol.

## Building the source 

Building gtc requires both a Go (version 1.10.1 or later) and a C compiler.

Clone the repository to a directory of your choosing:

    git clone https://github.com/touchconDev/go-touchcon.git

Building gtc requires package to be installed:

    ubuntu :
    sudo apt-get install -y build-essential

    centos :
    sudo yum install -y epel-release
    

Finally, build the gtc program using the following command.
 
    cd go-touchcon
    make gtc



## Executables

The go-touchcon project comes with several wrappers/executables found in the `cmd` directory.

| Command    | Description |
|:----------:|-------------|
| **`gtc`** | Our main TouchCon CLI client. It is the entry point into the TouchCon network (main-, test- or private net), capable of running as a full node (default) archive node (retaining all historical state) or a light node (retrieving data live). It can be used by other processes as a gateway into the TouchCon network via JSON RPC endpoints exposed on top of HTTP, WebSocket and/or IPC transports. `gtc --help` and the [CLI Wiki page](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options) for command line options. |
| `abigen` | Source code generator to convert TouchCon contract definitions into easy to use, compile-time type-safe Go packages. It operates on plain [TouchCon contract ABIs](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI) with expanded functionality if the contract bytecode is also available. However it also accepts Solidity source files, making development much more streamlined. Please see our [Native DApps](https://github.com/ethereum/go-ethereum/wiki/Native-DApps:-Go-bindings-to-Ethereum-contracts) wiki page for details. |
| `bootnode` | Stripped down version of our TouchCon client implementation that only takes part in the network node discovery protocol, but does not run any of the higher level application protocols. It can be used as a lightweight bootstrap node to aid in finding peers in private networks. |
| `evm` | Developer utility version of the EVM (TouchCon Virtual Machine) that is capable of running bytecode snippets within a configurable environment and execution mode. Its purpose is to allow isolated, fine-grained debugging of EVM opcodes (e.g. `evm --code 60ff60ff --debug`). |
| `gtcrpctest` | Developer utility tool to support our [ethereum/rpc-test](https://github.com/ethereum/rpc-tests) test suite which validates baseline conformity to the [Ethereum JSON RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC) specs. Please see the [test suite's readme](https://github.com/ethereum/rpc-tests/blob/master/README.md) for details. |
| `rlpdump` | Developer utility tool to convert binary RLP ([Recursive Length Prefix](https://github.com/ethereum/wiki/wiki/RLP)) dumps (data encoding used by the Ethereum protocol both network as well as consensus wise) to user friendlier hierarchical representation (e.g. `rlpdump --hex CE0183FFFFFFC4C304050583616263`). |
| `swarm`    | swarm daemon and tools. This is the entrypoint for the swarm network. `swarm --help` for command line options and subcommands. See https://swarm-guide.readthedocs.io for swarm documentation. |

## Running gtc

Going through all the possible command line flags is out of scope here (please consult the compatible Go-Ethereum
[CLI Wiki page](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options)), but we've
enumerated a few common parameter combos to get you up to speed quickly on how you can run your
own GTC instance.

### Full node on the main TouchCon network

By far the most common scenario is people wanting to simply interact with the TouchCon network:
create accounts; transfer funds; deploy and interact with contracts. For this particular use-case
the user doesn't care about years-old historical data, so we can fast-sync quickly to the current
state of the network. To do so:

```
$ gtc --fast --cache=512 console
```

This command will:

 * Start GTC in fast sync mode (`--fast`), causing it to download more data in exchange for avoiding
   processing the entire history of the Musicoin network, which is very CPU intensive.
 * Bump the memory allowance of the database to 512MB (`--cache=512`), which can help significantly in
   sync times especially for HDD users. This flag is optional and you can set it as high or as low as
   you'd like, though we'd recommend the 512MB - 2GB range.
 * Start up GTC's built-in interactive [JavaScript console](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console),
   (via the trailing `console` subcommand) through which you can invoke all official [`web3` methods](https://github.com/ethereum/wiki/wiki/JavaScript-API)
   as well as GTC's own [management APIs](https://github.com/ethereum/go-ethereum/wiki/Management-APIs).
   This too is optional and if you leave it out you can always attach to an already running GTC instance
   with `gtc attach`.


### Programatically interfacing GTC nodes

As a developer, sooner rather than later you'll want to start interacting with GTC and the TouchCon
network via your own programs and not manually through the console. To aid this, GTC has built-in
support for a JSON-RPC based APIs ([standard APIs](https://github.com/ethereum/wiki/wiki/JSON-RPC) and
[gtc specific APIs](https://github.com/ethereum/go-ethereum/wiki/Management-APIs)). These can be
exposed via HTTP, WebSockets and IPC (unix sockets on unix based platforms, and named pipes on Windows).

The IPC interface is enabled by default and exposes all the APIs supported by GTC, whereas the HTTP
and WS interfaces need to manually be enabled and only expose a subset of APIs due to security reasons.
These can be turned on/off and configured as you'd expect.

HTTP based JSON-RPC API options:

  * `--rpc` Enable the HTTP-RPC server
  * `--rpcaddr` HTTP-RPC server listening interface (default: "localhost")
  * `--rpcport` HTTP-RPC server listening port (default: `9646)
  * `--rpcapi` API's offered over the HTTP-RPC interface (default: "eth,net,web3")
  * `--rpccorsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
  * `--ws` Enable the WS-RPC server
  * `--wsaddr` WS-RPC server listening interface (default: "localhost")
  * `--wsport` WS-RPC server listening port (default: 8546)
  * `--wsapi` API's offered over the WS-RPC interface (default: "eth,net,web3")
  * `--wsorigins` Origins from which to accept websockets requests
  * `--ipcdisable` Disable the IPC-RPC server
  * `--ipcapi` API's offered over the IPC-RPC interface (default: "admin,debug,eth,miner,net,personal,shh,txpool,web3")
  * `--ipcpath` Filename for IPC socket/pipe within the datadir (explicit paths escape it)

You'll need to use your own programming environments' capabilities (libraries, tools, etc) to connect
via HTTP, WS or IPC to a GTC node configured with the above flags and you'll need to speak [JSON-RPC](http://www.jsonrpc.org/specification)
on all transports. You can reuse the same connection for multiple requests!

**Note: Please understand the security implications of opening up an HTTP/WS based transport before
doing so! Hackers on the internet are actively trying to subvert TouchCon nodes with exposed APIs!
Further, all browser tabs can access locally running webservers, so malicious webpages could try to
subvert locally available APIs!**

#### Starting up your member nodes

With the bootnode operational and externally reachable (you can try `telnet <ip> <port>` to ensure
it's indeed reachable), start every subsequent GTC node pointed to the bootnode for peer discovery
via the `--bootnodes` flag. It will probably also be desirable to keep the data directory of your
private network separated, so do also specify a custom `--datadir` flag.

```
$ gtc --datadir=path/to/custom/data/folder --bootnodes=<bootnode-enode-url-from-above>
```

*Note: Since your network will be completely cut off from the main and test networks, you'll also
need to configure a miner to process transactions and create new blocks for you.*

## License

The go-ethereum library (i.e. all code outside of the `cmd` directory) is licensed under the
[GNU Lesser General Public License v3.0](https://www.gnu.org/licenses/lgpl-3.0.en.html), also
included in our repository in the `COPYING.LESSER` file.

The go-ethereum binaries (i.e. all code inside of the `cmd` directory) is licensed under the
[GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html), also included
in our repository in the `COPYING` file.

