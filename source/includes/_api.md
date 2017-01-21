# api

## Standard API

This Javascript library has implemented all of the core API calls that are made available by the current [IOTA Reference Implementation](https://github.com/iotaledger/iri). For the full documentation of all the Standard API calls, please refer to the official documentation: [official API](https://iota.readme.io/).

You can simply use any of the available options from the `api` object then. For example, if you want to use the `getTips` function, you would simply do it as such:

```
iota.api.getTips(function(error, success) {
    // do stuff here
})
```

---

## getTransactionsObjects

Wrapper function for `getTrytes` and the Utility function `transactionObjects`. This function basically returns the entire transaction objects for a list of transaction hashes.


### Input
```
iota.api.getTransactionsObjects(hashes, callback)
```

1. **`hashes`**: `Array` List of transaction hashes
2. **`callback`**: `Function` callback.

### Return Value

1. **`Array`** - list of all the transaction objects from the corresponding hashes.

---

## findTransactionObjects

Wrapper function for `getTrytes` and the Utility function `transactionObjects`. This function basically returns the entire transaction objects for a list of transaction hashes.


### Input
```
iota.api.findTransactionObjects(hashes, callback)
```

1. **`hashes`**: `Array` List of transaction hashes
2. **`callback`**: `Function` callback.

### Return Value

1. **`Array`** - list of all the transaction objects from the corresponding hashes.

---

## getLatestInclusion

Wrapper function for `getNodeInfo` and `getInclusionStates`. It simply takes the most recent solid milestone as returned by getNodeInfo, and uses it to get the inclusion states of a list of transaction hashes.


### Input
```
iota.api.getLatestInclusion(hashes, callback)
```

1. **`hashes`**: `Array` List of transaction hashes
2. **`callback`**: `Function` callback.

### Return Value

1. **`Array`** - list of all the inclusion states of the transaction hashes

---

## broadcastAndStore

Wrapper function for `broadcastTransactions` and `storeTransactions`.

### Input
```
iota.api.broadcastAndStore(trytes, callback)
```

1. **`trytes`**: `Array` List of transaction trytes to be broadcast and stored. Has to be trytes that were returned from `attachToTangle`
2. **`callback`**: `Function` callback.

### Return Value

**`Object`** - empty object.

---

## getNewAddress

Generates a new address from a seed and returns the address. This is either done deterministically, or by providing the index of the new address to be generated.

### Input
```
iota.api.getNewAddress(seed [, options], callback)
```

1. **`seed`**: `String` tryte-encoded seed. It should be noted that this seed is not transferred
2. **`options`**: `Object` which is optional:
  - **`index`**: `Int` If the index is provided, the generation of the address is not deterministic.
  - **`checksum`**: `Bool` Adds 9-tryte address checksum
  - **`total`**: `Int` Total number of addresses to generate.
  - **`returnAll`**: `Bool` If true, it returns all addresses which were deterministically generated (until findTransactions returns null)
3. **`callback`**: `Function` Optional callback.

### Returns
**`String | Array`** - returns either a string, or an array of strings.

---

## getInputs

Gets all possible inputs of a seed and returns them with the total balance. This is either done deterministically (by genearating all addresses until `findTransactions` returns null for a corresponding address), or by providing a key range to use for searching through.

You can also define the minimum `threshold` that is required. This means that if you provide the `threshold` value, you can specify that the inputs should only be returned if their collective balance is above the threshold value.


### Input
```
iota.api.getInputs(seed, [, options], callback)
```

1. **`seed`**: `String` tryte-encoded seed. It should be noted that this seed is not transferred
2. **`options`**: `Object` which is optional:
  - **`start`**: `int` Starting key index  
  - **`end`**: `int` Ending key index
  - **`threshold`**: `int` Minimum threshold of accumulated balances from the inputs that is requested
4. **`callback`**: `Function` Optional callback.

### Return Value

1. **`Object`** - an object with the following keys:
    - **`inputs`** `Array` - list of inputs objects consisting of `address`, `balance` and `keyIndex`
    - **`totalBalance`** `int` - aggregated balance of all inputs


---

## prepareTransfers

Main purpose of this function is to get an array of transfer objects as input, and then prepare the transfer by **generating the correct bundle**, as well as **choosing and signing the inputs** if necessary (if it's a value transfer). The output of this function is an array of the raw transaction data (trytes).

You can provide multiple transfer objects, which means that your prepared bundle will have multiple outputs to the same, or different recipients. As single transfer object takes the values of: `address`, `value`, `message`, `tag`. The message and tag values are required to be tryte-encoded.

For the options, you can provide a list of `inputs`, that will be used for signing the transfer's inputs. It should be noted that these inputs (an array of objects) should have the provided `keyIndex` and `address` values:
```
var inputs = [{
    'keyIndex': //VALUE,
    'address': //VALUE
}]
```

The library validates these inputs then and ensures that you have sufficient balance. The `address` parameter can be used to define the address to which a remainder balance (if that is the case), will be sent to. So if all your inputs have a combined balance of 2000, and your spending 1800 of them, 200 of your tokens will be sent to that remainder address. If you do not supply the `address`, the library will simply generate a new one from your seed.

### Input
```
iota.api.prepareTransfers(seed, transfersArray [, options], callback)
```

1. **`seed`**: `String` tryte-encoded seed. It should be noted that this seed is not transferred
2. **`transfersArray`**: `Array` of transfer objects:
  - **`address`**: `String` 81-tryte encoded address of recipient
  - **`value`**: `Int` value to be transferred.
  - **`message`**: `String` tryte-encoded message to be included in the bundle.
  - **`tag`**: `String` Tryte-encoded tag. Maximum value is 27 trytes.
3. **`options`**: `Object` which is optional:
  - **`inputs`**: `Array` List of inputs used for funding the transfer
  - **`address`**: `String` if defined, this address will be used for sending the remainder value (of the inputs) to.
4. **`callback`**: `Function` Optional callback.

### Return Value

`Array` - an array that contains the trytes of the new bundle.

---

## sendTrytes

Wrapper function that does `attachToTangle` and finally, it broadcasts and stores the transactions.

### Input
```
iota.api.sendTrytes(trytes, depth, minWeightMagnitude, callback)
```

1. **`trytes`** `Array` trytes
2. **`depth`** `Int` depth value that determines how far to go for tip selection
3. **`minWeightMagnitude`** `Int` minWeightMagnitude
4. **`callback`**: `Function` Optional callback.

### Returns
`Array` - returns an array of the transfer (transaction objects).

---

## sendTransfer

Wrapper function that basically does `prepareTransfers`, as well as `attachToTangle` and finally, it broadcasts and stores the transactions locally.

### Input
```
iota.api.sendTransfer(seed, depth, minWeightMagnitude, transfers [, options], callback)
```

1. **`seed`** `String` tryte-encoded seed. If provided, will be used for signing and picking inputs.
2. **`depth`** `Int` depth
3. **`minWeightMagnitude`** `Int` minWeightMagnitude
4. **`transfers`**: `Array` of transfer objects:
  - **`address`**: `String` 81-tryte encoded address of recipient
  - **`value`**: `Int` value to be transferred.
  - **`message`**: `String` tryte-encoded message to be included in the bundle.
  - **`tag`**: `String` 27-tryte encoded tag.
5. **`options`**: `Object` which is optional:
  - **`inputs`**: `Array` List of inputs used for funding the transfer
  - **`address`**: `String` if defined, this address will be used for sending the remainder value (of the inputs) to.
6. **`callback`**: `Function` Optional callback.


### Returns
`Array` - returns an array of the transfer (transaction objects).

---

## replayBundle

Takes a tail transaction hash as input, gets the bundle associated with the transaction and then replays the bundle by attaching it to the tangle.

### Input
```
iota.api.replayBundle(transaction [, callback])
```

1. **`transaction`**: `String` Transaction hash, has to be tail.
2. **`depth`** `Int` depth
3. **`minWeightMagnitude`** `Int` minWeightMagnitude
2. **`callback`**: `Function` Optional callback

---

## broadcastBundle

Takes a tail transaction hash as input, gets the bundle associated with the transaction and then rebroadcasts the entire bundle.

### Input
```
iota.api.broadcastBundle(transaction [, callback])
```

1. **`transaction`**: `String` Transaction hash, has to be tail.
2. **`callback`**: `Function` Optional callback

---

## getBundle

This function returns the bundle which is associated with a transaction. Input has to be a tail transaction (i.e. currentIndex = 0). If there are conflicting bundles (because of a replay for example) it will return multiple bundles. It also does important validation checking (signatures, sum, order) to ensure that the correct bundle is returned.

### Input
```
iota.api.getBundle(transaction, callback)
```

1. **`transaction`**: `String` Transaction hash of a tail transaction.
2. **`callback`**: `Function` Optional callback

#### Returns
`Array` - returns an array of the corresponding bundle of a tail transaction. The bundle itself consists of individual transaction objects.

---


## getTransfers

Returns the transfers which are associated with a seed. The transfers are determined by either calculating deterministically which addresses were already used, or by providing a list of indexes to get the addresses and the associated transfers from. The transfers are sorted by their timestamp. It should be noted that, because timestamps are not enforced in IOTA, that this may lead to incorrectly sorted bundles (meaning that their chronological ordering in the Tangle is different).

If you want to have your transfers split into received / sent, you can use the utility function `categorizeTransfers`

### Input
```
getTransfers(seed [, options], callback)
```

1. **`seed`**: `String` tryte-encoded seed. It should be noted that this seed is not transferred
2. **`options`**: `Object` which is optional:
  - **`start`**: `Int` Starting key index for search
  - **`end`**: `Int` Ending key index for search
  - **`inclusionStates`**: `Bool` If True, it gets the inclusion states of the transfers.
3. **`callback`**: `Function` Optional callback.

### Returns
`Array` - returns an array of transfers. Each array is a bundle for the entire transfer.

---

## getAccountData

Similar to `getTransfers`, just a bit more comprehensive in the sense that it also returns the `balance` and `addresses` that are associated with your account (seed). This function is useful in getting all the relevant information of your account. If you want to have your transfers split into received / sent, you can use the utility function `categorizeTransfers`

### Input
```
getAccountData(seed [, options], callback)
```

1. **`seed`**: `String` tryte-encoded seed. It should be noted that this seed is not transferred
2. **`options`**: `Object` which is optional:
  - **`start`**: `Int` Starting key index for search
  - **`end`**: `Int` Ending key index for search
3. **`callback`**: `Function` Optional callback.

#### Returns
`Object` - returns an object of your account data in the following format:
```
{
    'addresses': [],
    'transfers': [],
    'balance': 0
}
```
