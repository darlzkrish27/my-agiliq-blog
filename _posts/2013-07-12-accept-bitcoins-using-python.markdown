---
layout: post
title:  "Accept bitcoins using python"
date:   2013-07-12 18:32:44+05:30
tags: python
author: Javed
---
In this blog post we'll talk about bitcoins and look at an overview of how to
accept them from your python app.

Bitcoin:
--------

Bitcoin is crypto-currency whose protocol allows fast, anonymous and secure
digital transactions.

To know more about bitcoin and to install it, go to the ["Developer
Guide"](http://bitcoin.org/en/bitcoin-for-developers) guide.

Getting started:
----------------

To get started, install bitcoin (assuming Ubuntu):

    $ sudo apt-get install bitcoind

Now edit `~/.bitcoin/bitcoin.conf` and add:

    testnet=1

This will allow us to run bitcoin in test mode.

Now start the `bitcoind` daemon using:

    $ bitcoind -daemon

To check that it's working, do:

    $ bitcoind getinfo

If you see a JSON output, you should be good to go.

Using JSON RPC:
---------------

Bitcoin includes a JSON RPC server as a part of the daemon.

Using a client library like
["bitcoin-python"](https://github.com/laanwj/bitcoin-python), we can invoke
bitcoin calls from python like so:

    >>> import bitcoinrpc
    >>> bitcoin = bitcoinrpc.connect_to_local()
    >>> bitcoin.getinfo()

Payment Flow:
-------------

In a standard payment flow, the steps involved would be:

* User checks out a shopping cart. Transaction starts.
* Generate a new bitcoin addres and ask the user to pay.
* Verify that payment is made, wait for confirmations if required.
* Generate invoice. Transaction ends.

Let's see how to do each of this step using the python client:

Generate a new bitcoin address:

    >>> add = bitcoin.getnewaddress()

Verify payment is made:

    >>> bitcoin.getreceivedbyaddress(add)

Get transcation info:

    >>> bitcoin.listtransactions(address=add)

Note: By default the RPC client waits for 1 confirmation, you can increase
this using the `minconf` argument to most API calls.

You can use a testnet faucet for testing:

[http://tpfaucet.appspot.com/](http://tpfaucet.appspot.com/)

[http://testnet.mojocoin.com/](http://testnet.mojocoin.com/)

When you're ready, you can switch the testnet mode in your bitcoinf.conf.

Security:
---------

You need to take precautions to make sure that your bitcoin wallet is secured
to prevent wallet theft, this is out of the scope of this tutorial, so please
refer to:

[https://en.bitcoin.it/wiki/Securing_your_wallet](https://en.bitcoin.it/wiki/Securing_your_wallet)

References:
-----------

[https://en.bitcoin.it/wiki/Bitcoin-python](https://en.bitcoin.it/wiki/Bitcoin-python)

[https://en.bitcoin.it/wiki/Merchant_Howto](https://en.bitcoin.it/wiki/Merchant_Howto)



