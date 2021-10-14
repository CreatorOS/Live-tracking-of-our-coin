# Live tracking of our coin
In the previous quest we launched our own coin. In this quest we’ll track the live updates on who is sending coins to whom. All the data on the blockchain is open data. Whenever a transfer happens, the logs are stored on the computers that are processing the transaction - aka nodes. But for you to keep track of the transfers, you’ll have to run a node yourself which needs you to keep your computer on every time. And, it also requires you to go through the logs and process raw information. This is not only cumbersome, but might also result in non-realtime-ness.

In this quest we’ll create a landing page for our coin that will show the latest transactions in real time. Afterall, the most important information about a coin is how frequently it is being traded or changing hands.
## Making external tracking possible
First up, we need to include in the smart contract what are the things we want to track in real time. We’ll be using the contract that we had written in the quest on Launching your own Coin on [https://questb.uk](https://questb.uk)

Here’s the contract : 

[https://remix.ethereum.org/\#version=soljson-v0.8.4\+commit.c7e474f2.js&optimize=false&runs=200&gist=b6dffd2035f7cc0adea39d3bb300ab63&evmVersion=null](https://remix.ethereum.org/#version=soljson-v0.8.4+commit.c7e474f2.js&optimize=false&runs=200&gist=b6dffd2035f7cc0adea39d3bb300ab63&evmVersion=null)

The first thing we need to do is send out events as and when they occur from our smart contract. The way to do this is to `emit ` an event.

First up we’ll define the event that we want to emit.![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/34957c74-28e8-4f5f-83af-220abca64615.jpg)

This tells Solidity as to what the signature is for the event. The event consists of some data associated with it. In this case, we’re creating an event that tells the listener from whom this transfer was initiated, to whom and what was the value of that transfer. 

Now that we’ve defined the event, we can start emitting them in our contract. We will emit wherever a transaction is getting completed and a transfer is made.

Figure out what are the places you should emit this event.
## what are events
First check if you’ve added the events at the right place. There are two places where a transfer happens. `transfer()` and `transferFrom()`. The emit needs to be added to these functions. But remember they should be emitted only upon a successful transaction. I.e. when the transaction completes. 

Check the solution here: [https://remix.ethereum.org/\#version=soljson-v0.8.4\+commit.c7e474f2.js&optimize=false&runs=200&gist=a577a320809b2b21b6665c1afe7de821&evmVersion=null](https://remix.ethereum.org/#version=soljson-v0.8.4+commit.c7e474f2.js&optimize=false&runs=200&gist=a577a320809b2b21b6665c1afe7de821&evmVersion=null)

Once you have added these events let’s see them in action!

To do so, compile it and deploy it to ropsten.

After deploying, first check if your account has balance. If you’ve not modified the above code, you should have balance as the deployer of the contract ;)

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/741310cc-76ac-4ac2-b93e-038f7cabba8c.jpg)

Now, let’s initiate a transfer. Here’s my address. Why don’t you transfer some coins to me? `0x0D5138BC001aFaEf7Fc679DE66463E3108dc7C26`

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/f0377840-da4c-4138-a26c-5d96f83721e7.jpg)

Once you have hit transact and signed it using the Metamask popup, you’ll see the successful transaction in the logs in a few seconds. 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/236bad33-3fa7-45a9-aa60-b23fff54bde9.jpg)

This means that the transaction has completed. The output has a field called transaction hash. This is what is used to uniquely identify your transaction. 

Let’s check what this transaction looks like on etherscan.

Copy the transaction hash and open

`[https://ropsten.etherscan.io/tx/PASTE_TRANSACTION_HASH_HERE](https://ropsten.etherscan.io/tx/PASTE_TRANSACTION_HASH_HERE)`

Head over to the tab called logs.

You’ll see the following : ![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/1801135c-1bda-4e6f-a72f-bdff0ca4c940.jpg)

Address is the contract’s address from which the event was emitted.

Topics \[0\] is an identifier for your event “Transfer”

Then the next two lines are the from and the to fields. Etherscan knows that Transfer is a common event and has the parsing setup for it already.

Lastly the data is the hex representation of our last parameter in the event, the value - number of coins.

Now that these events are correctly being emitted, we can move on to writing the next step of creating a listener.
## Setting up a listener
To listen to events that are being emitted in real time, you need a computer to be always on to listen to these events and trigger actions. 

But instead of spinning up a node ourselves, we’ll use a useful tool that setsup a computer node for us on the cloud and lets us connect to it over the internet.

Head over to [https://infura.io](https://infura.io) 

Go to ethereum section after signing up/logging in.

And create a new project.

Select the new project you’ve selected and head over to the settings tab. Under the keys section, change the network to ropsten and copy the `projectid`

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/0ae8bec1-1032-4ad7-a1d2-3831c99ff71c.jpg)

Infura will be listening to all the events through out ethereum and will communicate that to our service that we’ll write shortly over wss (web sockets, secure).
## Creating a service
To show real time updates, let’s create a node file called index.js.

We’ll import the a library called ethers.js. This is a handy javascript library for both browser and nodejs. The documentation can be found here : [https://docs.ethers.io](https://docs.ethers.io) 

From your terminal (on mac and linux) and wsl on windows, lets quickly setup our project:

```

mkdir quest-events

cd quest-events

npm init

npm install ethers

```

We now need to setup ethers.js. 

Create a file called index.js in the directory `quest-events`

```

const { ethers } = require("ethers");

    const provider = new ethers.providers.InfuraProvider( network = "ropsten" , "paste infura project id here" )

```

The provider for us is what we just setup on infura. Copy the wss link and paste it here.

We’ll come back to how to make this more secure, but this is ok for this demo right now.

Let’s see if this is working!

```

provider.getBlockNumber().then(data => {

  console.log(“connected to ethereum. current block : ” \+data);

});

```

And then run the file using 

```

node index.js

```

You’ll see that the current block is being printed to console. That means, your ethers.js based service is able to talk to the infura provider.
## Defining the contract
We’ve seen earlier (like, on [https://ethcontract.app](https://ethcontract.app)) that to define a contract you need two things. First is the contract address and the second is the contract abi.

Go to remix and copy the contract address

```

const contractAddress = “”

```

Then copy the ABI from the compile tab in remix

```

const contractAbi = \[

	{

		"inputs": \[\],

		"stateMutability": "nonpayable",

		"type": "constructor"

	},

            ...

```

Let’s initialize the contract

```

const contract = new ethers.Contract(contractAddress, contractAbi, provider);

```

 To check if the contract is initialized correctly, lets call a function from this contract. If you remember, the contract has a function called name() that returns the name of our crypto currency. Since all of functions we write will have to talk to the infura node over a network call, it’s better we write our code in an async function

  
```

async function main() {

  const name = await contract.name();

  console.log(name);

}

main();

```

Again run the index.js again

```

node index.js

```

Now let’s listen to our events.
## Listening to events
Now that our contract is initialized and working, we can start listening to the events.

```

contract.on(“Transfer”, function(from, to, value) {

  console.log(“New Transfer recorded from “ \+ from \+ “ to “ \+ to \+ “ for “ \+ value \+ “ coins “);

});

```

This time around when you run the node file, you’ll see that the program doesn’t terminate it will keep listening to the events called “transfer”.

The function which is the second parameter takes as parameters the same data as present in the event that we emit from the contract. 

Now that the function is listening to the events, go back to remix and create a transfer again. I don’t mind if you send a few more coins to me at `0x0D5138BC001aFaEf7Fc679DE66463E3108dc7C26`. Make sure you don’t deploy the contract again. If you do, also remember to update the new contractAddress in your index.js

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/8569a996-15f1-479b-b017-2282c898bd5a.jpg)

When the transaction completes, you’ll see the logs in Remix and also on your console where index.js is running 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/72807cae-93cb-43c4-b837-f327420493bf.jpg)

So you’ve now successfully started listening to events.

[https://gist.github.com/madhavanmalolan/a11dcab72d6017614dd1ed712980ce09](https://gist.github.com/madhavanmalolan/a11dcab72d6017614dd1ed712980ce09)
## Next steps
As a next step, create a service that’ll always be running on your computer. Store the transactions in a database (say mongo) as and when they occur. Then create a webpage using expressjs that shows the latest transactions in real time!