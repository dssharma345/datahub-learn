# Add a Validator

## Introduction

The [Primary Network](https://avalanche.gitbook.io/avalanche/build/tutorials/platform/add-a-validator#introduction) is incorporated to the Avalanche platform and legitimize Avalanche’s [built-in blockchains](https://avalanche.gitbook.io/avalanche/learn/platform-overview). In this tutorial, we’ll create a [subnet](https://avalanche.gitbook.io/avalanche/learn/platform-overview#subnets) on Avalanche and a a node to the Primary Network. 


On Avalanche, the P-Chain regulates metadata.  This involves keeping track of which nodes belong to which subnets, which blockchains are active, and which subnets validate which blockchains. We’ll issue [transactions](http://support.avalabs.org/en/articles/4587384-what-is-a-transaction) to the P-Chain in order to add a validator. 

{% hint style="danger" %}
It's worth noting that there is no way to modify the parameters once you've issued the transaction to add a node as a validator. **You cannot amend the stake amount, node ID, or reward address, or withdraw your stake early.** Please double-check that the values in the API calls below are correct. If you’re not sure, browse the [Developer FAQ's](http://support.avalabs.org/en/collections/2618154-developer-faq) or ask for help on [Discord.](https://chat.avalabs.org/)
{% endhint %}

## Requirements

You've accomplished [Run an Avalanche Node](../../get-started.md) and are acquainted with [Avalanche's architecture](../../../learn/platform-overview/). In this tutorial, we'll utilize [Avalanche’s Postman collection](https://github.com/ava-labs/avalanche-postman-collection) to assist us make API calls.

Make sure your node can receive and send TCP traffic on the staking port \(`9651` by default\) and that you initiated the node with the command line parameter `--public-ip=[YOUR NODE'S PUBLIC IP HERE]` to ensure your node is well-connected. If you fail to do either of these, your staking reward could be jeopardized.

## Create a Avalanche Wallet
 Make a [Avalanche Wallet] by clicking on this link (https://wallet.avax.network).
 ![Create a Avalanche Wallet](../../../.gitbook/assets/create-new-wallet.png).
 
## Home page of a Avalanche Wallet    
![Home page](../../../.gitbook/assets/wallet-ava-home.png).

## Add a validator with Avalanche Wallet

First, We've demostrated how to add your node as a validator by using [Avalanche Wallet](https://wallet.avax.network).

Get your node’s ID by calling [`info.getNodeID`](https://avalanche.gitbook.io/avalanche/build/apis/info-api#info-getnodeid):

![getNodeID postman](../../../.gitbook/assets/getNodeID-postman.png)

```cpp
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.getNodeID"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

The response would've your node’s ID:

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "nodeID": "NodeID-5mb46qkSBj81k9g9e4VFjGGSbaaSLFRzD"
    },
    "id": 1
}
```

Open [the Avalanche wallet](https://wallet.avax.network/), and go to the `Earn` tab. Choose `Add Validator`.
![Web wallet earn tab](../../../.gitbook/assets/ava-earn-wallet.png).

Select this network option to enable Validator.
![wallet.ava-change-network-to-fuji](../../../.gitbook/assets/wallet.ava-change-network-to-fuji.png).

Select the Fuji option instead of Mainnet.
![select-fuji-network](../../../.gitbook/assets/select-fuji-network.png).

To  `Add Validator` you need to add coins in the Wallet.

For adding the coin in the Wallet, Click on this Link (https://faucet.avax-test.network).
![avax-request-fund](../../../.gitbook/assets/avax-request-fund.png).
Add your wallet Address, and verify recapcha and hit on Request AVAX.

Now you have to convert your coin `X-Chain` to `P-Chain`, by clicking on the Cross Chain Tab.
![convert-x2p chain-ava-wallet](../../../.gitbook/assets/convert-x2p chain-ava-wallet.png).
Put the Amount and hit 'confirm'.
 

Complete the staking parameters. We've discussed in greater depth further down. After you've double-checked all of the staking parameters, click `Confirm` Make sure your staking term is at least two weeks long, the delegation fee rate is at least 2%, and you're staking at least 2,000 AVAX.

{% page-ref page="../../../learn/platform-overview/staking.md" %}

![Earn validate](../../../.gitbook/assets/earn-validate.png)

This success notice should appear, along with an update to your balance.

![Your validation transaction is sent](../../../.gitbook/assets/your-validation-transaction-is-sent.png)

Calling [`platform.getPendingValidators`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-getpendingvalidators) confirms that your transaction was successfully accepted.

![getPendingValidators postman](../../../.gitbook/assets/getPendingValidators-postman.png)

Go back to the `Earn` tab, and click `Estimated Rewards`.

![Earn, validate, delegate](../../../.gitbook/assets/earn-validate-delegate.png)

Once its start time has passed, you'll see the rewards your validator may have earned, as well as its start time, end time, and the percentage of its validation period that has elapsed.

![Estimated rewards](../../../.gitbook/assets/estimated-rewards.png)

That’s it!

## Add a validator with API calls

By making API calls to our node, we can also add a node to the validator set. To add a node the Primary Network, we’ll call [`platform.addValidator`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-addvalidator).

This method’s signature is:

```cpp
platform.addValidator(
    {
        nodeID: string,
        startTime: int,
        endTime: int,
        stakeAmount: int,
        rewardAddress: string,
        changeAddr: string, (optional)
        delegationFeeRate: float,
        username: string,
        password: string
    }
) -> {txID: string}
```

Let’s go through and examine these arguments.

`nodeID`

This is the node ID of the validator being added. To obtain your node’s ID, call [`info.getNodeID`](https://avalanche.gitbook.io/avalanche/build/apis/info-api#info-getnodeid):

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "info.getNodeID",
    "params":{},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

The response has your node’s ID:

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "nodeID": "NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji"
    },
    "id": 1
}
```

`startTime` and `endTime`

When someone initiates a transaction to join the Primary Network, they must mention the time they want to enter. \(start validating\) and leave \(stop validating.\) The Primary Network can be validated for a minimum of 24 hours and a maximum of one year. After leaving the Primary Network, one can rejoin however, the maximum _continuous_ period is one year. The Unix times when your validator will start and stop validating the Primary Network are `startTime` and `endTime` respectively. In relation to the time the transaction is issued,`startTime` must be in the future.

`stakeAmount`

Staking AVAX is required to validate the Primary Network. The quantity of AVAX staked is defined by this parameter.
`rewardAddress`

When a validator stops validating the Primary Network, they will receive a reward if they are sufficiently responsive and correct while they validated the Primary Network. These tokens are sent to `rewardAddress`. The original stake will be returned to a 'username' controlled address.

The stake of a validator is never slashed, regardless of their actions; they always get their stake back when they're finished verifying.
`changeAddr`

This address will receive any changes resulting from this transaction. You can leave this field blank; if you do, money will be sent to one of your user's addresses.

`delegationFeeRate`

Stake delegation is possible with Avalanche. When others delegate stake to this validator, this parameter is the percentage fee they charge. For example, if the `delegationFeeRate` is `1.2345` and someone delegates to this validator, then 1.2345 percent of the reward goes to the validator and the remainder goes to the delegator when the delegation period ends.

`username` and `password`

These are the username and password of the user who pays the transaction fee, provides the staked AVAX, and gets the staked AVAX returned.

Now it's time to issue the transaction. For the values of `startTime` and `endTime` we utilise the shell command `date` to compute the Unix time 10 minutes and 30 days in the future, respectively. \(Note: If you’re on a Mac, replace `$(date` with `$(gdate`. If you don’t have `gdate` installed, do `brew install coreutils`.\) In this example we stake 2,000 AVAX \(2 x 1012 nAVAX\).

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.addValidator",
    "params": {
        "nodeID":"NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
        "startTime":'$(date --date="10 minutes" +%s)',
        "endTime":'$(date --date="30 days" +%s)',
        "stakeAmount":2000000000000,
        "rewardAddress":"P-avax1d4wfwrfgu4dkkyq7dlhx0lt69y2hjkjeejnhca",
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u",
        "delegationFeeRate":10,
        "username":"USERNAME",
        "password":"PASSWORD"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The transaction ID is included in the response, as well as the address where the change went to. 

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "txID": "6pb3mthunogehapzqmubmx6n38ii3lzytvdrxumovwkqftzls",
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u"
    },
    "id": 1
}
```

We can verify the status of the transaction by calling [`platform.getTxStatus`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-gettxstatus):

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getTxStatus",
    "params": {
        "txID":"6pb3mthunogehapzqmubmx6n38ii3lzytvdrxumovwkqftzls"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The status should be `Committed`, which indicates that the transaction was completed successfully. We can call [`platform.getPendingValidators`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-getpendingvalidators) and see that the node is now in the pending validator set for the Primary Network:

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getPendingValidators",
    "params": {},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response should include the node we just added:

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "validators": [
            {
                "nodeID": "NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
                "startTime": "1584021450",
                "endtime": "1584121156",
                "stakeAmount": "2000000000000",
            }
        ] 
    },
    "id": 1
}
```

This node will commence validating the Primary Network when the time reaches `1584021450`. This node will stop validating the Primary Network when it reaches `1584121156`. The staked AVAX will be returned to an address controlled by `username` and any rewards will be sent to `rewardAddress` if there are any.

## Adding a Subnet Validator

### Issuing a Subnet Validator Transaction

Now let’s add the same node to a subnet. The following will make more sense if you’ve already done this [tutorial on creating a Subnet](https://avalanche.gitbook.io/avalanche/build/tutorials/platform/create-a-subnet). Right now you can only add validators to subnets with API calls, not with Avalanche Wallet.

Suppose that the Subnet has ID `nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr`, threshold 2, and that `username` holds at least 2 control keys.

To add the validator, we’ll call API method [`platform.addSubnetValidator`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-addsubnetvalidator). Its signature is:

```cpp
platform.addSubnetValidator(
    {
        nodeID: string,
        subnetID: string,
        startTime: int,
        endTime: int,
        weight: int,
        changeAddr: string, (optional)
        username: string,
        password: string
    }
) -> {txID: string}
```

Let’s examine the parameters:

`nodeID`

This is the node ID of the validator being added to the subnet. **This validator must validate the Primary Network for the entire duration that it validates this Subnet.**

`subnetID`

This is the ID of the subnet we’re adding a validator to.

`startTime` and `endTime`

Similar to above, these are the Unix times that the validator will start and stop validating the subnet. `startTime` must be at or after the time that the validator starts validating the Primary Network, and `endTime` must be at or before the time that the validator stops validating the Primary Network.

`weight`

This is the validator’s sampling weight for consensus. If the validator’s weight is 1 and the cumulative weight of all validators in the subnet is 100, then this validator will be included in about 1 in every 100 samples during consensus.

`changeAddr`

Any change resulting from this transaction will be sent to this address. You can leave this field empty; if you do, change will be sent to one of the addresses your user controls.

`username` and `password`

These parameters are the username and password of the user that pays the transaction fee. This user must hold a sufficient number of this Subnet’s control keys in order to add a validator to this Subnet.

We use the shell command `date` to compute the Unix time 10 minutes and 30 days in the future to use as the values of `startTime` and `endTime`, respectively. \(Note: If you’re on a Mac, replace `$(date` with `$(gdate`. If you don’t have `gdate` installed, do `brew install coreutils`.\)

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.addSubnetValidator",
    "params": {
        "nodeID":"NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
        "subnetID":"nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr",
        "startTime":'$(date --date="10 minutes" +%s)',
        "endTime":'$(date --date="30 days" +%s)',
        "weight":1,
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u",
        "username":"USERNAME",
        "password":"PASSWORD"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response has the transaction ID, as well as the address the change went to.

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "txID": "2exafyvRNSE5ehwjhafBVt6CTntot7DFjsZNcZ54GSxBbVLcCm",
        "changeAddr": "P-avax103y30cxeulkjfe3kwfnpt432ylmnxux8r73r8u"
    },
    "id": 1
}
```

We can check the transaction’s status by calling [`platform.getTxStatus`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-gettxstatus):

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getTxStatus",
    "params": {
        "txID":"2exafyvRNSE5ehwjhafBVt6CTntot7DFjsZNcZ54GSxBbVLcCm"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The status should be `Committed`, meaning the transaction was successful. We can call [`platform.getPendingValidators`](https://avalanche.gitbook.io/avalanche/build/apis/platform-chain-p-chain-api#platform-getpendingvalidators) and see that the node is now in the pending validator set for the Primary Network. This time, we specify the subnet ID:

```cpp
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getPendingValidators",
    "params": {"subnetID":"nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr"},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response should include the node we just added:

```cpp
{
    "jsonrpc": "2.0",
    "result": {
        "validators": [
            {
                "nodeID": "NodeID-LMUue2dBBRWdDbPL4Yx47Ps31noeewJji",
                "startTime":1584042912,
                "endTime":1584121156,
                "weight": "1"
            }
        ]
    },
    "id": 1
}
```

When the time reaches `1584042912`, this node will start validating this Subnet. When it reaches `1584121156`, this node will stop validating this Subnet.

### Whitelisting the Subnet

Now that the node has been added as a validator of the subnet, let’s add it to the whitelist of subnets. The whitelist prevents the node from validating a subnet unintentionally.

To whitelist the subnet, restart the node and add the parameter `--whitelisted-subnets` with a comma separated list of subnets to whitelist.

The full command is:

`./build/avalanchego --whitelisted-subnets=nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr`

