EOS.IO Software-based blockchain boot orchestration tool
--------------------------------------------------------

[点击查看中文](./README-cn.md)

`eos-bios` is a command-line tool for people who want to kickstart a
blockchain using EOS.IO Software.

It implements the following:
* Multi-staged launch of the mainnet
* Booting local development environments
* Booting testnets
* Booting consortium or private networks

The first you need to know is the **discovery protocol**. See an introduction here:

[![The Disco Dance](https://i.ytimg.com/vi/8aNZ_ZnKS-A/hqdefault.jpg?sqp=-oaymwEZCNACELwBSFXyq4qpAwsIARUAAIhCGAFwAQ==&rs=AOn4CLCZeGSGv9Qix8mHX77R4-d0rzDkgA)](https://youtu.be/8aNZ_ZnKS-A)

https://youtu.be/8aNZ_ZnKS-A

Jump directly to the [sample configurations](./sample_config) if you
know what you're doing.

Other **videos** explaining the different concepts of `eos-bios`:

| Description | Link |
| ----------- | ----:|
| Details of the Discovery file  | https://youtu.be/uE5J7LCqcUc |
| Network meshing algorithm  | https://youtu.be/I-PrnlmLNnQ |
| The boot sequence | https://youtu.be/SbVzINnqWAE |
| Code review of the core flow  | https://youtu.be/p1B6jvx25O0 |
| The hooks to integrate with your infrastructure  | https://youtu.be/oZNV6fUoyqM |

Some Q&A related to `eos-bios`:

| Description | Link |
| ----------- | ----:|
| Are accounts carried over from stage to stage?  | https://youtu.be/amyMm5gVpLg |
| How block producers agree on the content of the chain? | https://youtu.be/WIR7nab40qk |



Local development environment
-----------------------------

[Download `eos-bios` from the releases section here on GitHub](https://github.com/eoscanada/eos-bios/releases),
clone this repository and copy the `sample_config` to a directory of
your choice.

Modify the `my_discovery_file.yaml` to point to a local address:

```
target_http_address: http://localhost:8888
```

Then run:

    ./eos-bios boot --single

This gives you a fully fledged development environment, a chain loaded
with all system contracts, very similar to what you will get on the
main network once launched.

The sample configuration sets up a single node, as it doesn't point to
other block producer candidates (skips the `peers` discovery).



Orchestrate a community launch
------------------------------

When the time comes to orchestrate a launch, *everyone* will run:

    ./eos-bios orchestrate

According to an algorithm, and using the network discovery data, each
team will be assigned a role deterministically:

1. The _BIOS Boot node_, which will, alone, execute the equivalent of `eos-bios boot`.
2. An _Appointed Block Producer_, which executes the equivalent of `eos-bios join --validate`
3. An _other participant_, which executes the equivalent of `eos-bios join`

The orchestrate for all will wait the agreed upon
`seed_network_launch_block`, and then do the dance.


### Practicing to join


Ask anyone on the [latest seed network](#seed-networks) to invite you. They will run:

    ./eos-bios invite [youraccount] [your pubkey]

This will create your account on the seed network. Tweak your `my_discovery_file.yaml`:

* `seed_network_account_name`, make it match what you provided for an invite.
* `seed_network_http_address`, this should be the address of the seed network you want to orchestrate from.

Also add your seed account private key to `privkey.keys`.  This will allow you to run:

    ./eos-bios publish

You will want to tweak those values in your `my_discovery_file.yaml` also:

* `target_http_address` is the address to reach the node you're booting for the next stage.
* `target_p2p_address` is a publicly reachable address advertised to mesh the network
* `target_appointed_block_producer_signing_key` and `target_initial_authority`: these will be injected on the newly created blockchain as `target_account_name`.
* `seed_network_peers` this one warrants [its own section](#network-peers).

Other notable fields:
* `seed_network_launch_block` is the target block on the seed network which will unleash an `orchestrate`d launch.
* `target_contents` are all the pieces of content we need to agree on that will make it into the chain, like system contracts, ERC-20 snapshots, etc..



### Practicing to boot

Review your `my_discovery_file.yaml` (see previous section).  Then run:

    ./eos-bios boot

This will test your `hook_boot_*.sh` hooks.


Discovering the network
-----------------------

When you do point to a seed network (in your config), you can:

    ./eos-bios discover

and this will print the peers, their relative weights, etc..


Network peers
-------------

The `seed_network_peers` section of your discovery file looks like this:

```
seed_network_peers:
- account: eosexample
  comment: "They are good"
  weight: 10  # Weights are between 0 and 100 (INT value)
- account: eosmore
  comment: "They are better"
  weight: 20
```

This means you are comfortable launching the network with both
`eosexample` (at 10% vote weight), and `eosmore` (at 20%). `eos-bios`
will compute a graph of the network based on that peering information.

These are all account names on the seed network used to boot a new
network.


How to weight your peers
------------------------

1. Are they competent and capable persons? Can they fix the network
   if it goes awry?

2. Are they providing a solid and tested infrastructure? Can be stay
   on schedule and produce blocks until we reach the 15% threshold ?

3. Do they fully understand the boot sequence ? Do they understand all
   actions that need to be processed in order to have a chain that
   qualifies as mainnet. (they decide on boot_sequence.yaml)

4. Can they compile system contracts and compare their source code,
   making sure that the proposed contracts are legit, do not contain
   rogue code, etc.. (they decide on target_contents)

5. Do they understand how to make sure the snapshot.csv is valid,
   up-to-date and reflect the last Ethereum snapshot ? (they decide on
   snapshot.csv)

6. Can they properly boot the network and have they practiced being
   the BIOS Boot node.

7. Can they properly boot a node and mesh into the network, have they
   practiced `join` ?


The reason for those is because of the design of `eos-bios` .. votes
determine who gets which role, and based on the role you have, you
have critical decisions to make and the community relies on you for
the critical things, in the order above.


Seed networks
-------------

We keep an updated list of the different stages launched with `eos-bios` here:

https://stages.eoscanada.com



Install / Download
------------------

You can download the latest release here:
https://github.com/eoscanada/eos-bios/releases .. it is a single
binary that you can download on all major platforms. Simply make
executable and run. It has zero dependencies.

Alternatively, you can build from source with:

    go get -v github.com/eoscanada/eos-bios/eos-bios

This will install the binary in `~/go/bin` provided you have the Go
tool installed (quick install at https://golang.org/dl)

Add `-u` to `go get` to pull updates.



Join the discussion
-------------------

On Telegram through this invite link:
https://t.me/joinchat/GSUv1UaI5QIuifHZs8k_eA (EOSIO BIOS Boot channel)



Previous proposition
--------------------

See the previous proposition in this repo in README.v0.md



Readiness checklist
-------------------

* Did I update my `target_p2p_address` to reflect the IP of the NEW network we're booting ?
* Did I update my `target_http_address` to point to my node, reachable from `eos-bios`'s machine ?


Troubleshooting
---------------

* Do your `PRIVKEY` and `PUBKEY` in `hook_join_network.sh` match what
  you published in your discovery file under
  `target_initial_authority` and `target_initial_block_signing_key` ?

* Forked ? Did someone point to an old p2p address ? If so, remove
  them from the network.
