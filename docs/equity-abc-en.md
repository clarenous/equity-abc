# The Equity Language

## Index

  - [Introduction](#introduction)
  - [Contract](#contract)
  - [Clause](#clause)
  - [Contract and clause parameters](#contract-and-clause-parameters)
  - [Required payments](#required-payments)
  - [Statements](#statements)
      - [Verify statements](#verify-statements)
      - [Unlock statements](#unlock-statements)
      - [Lock statements](#lock-statements)
  - [Types of data](#types-of-data)
  - [Expressions](#expressions)
  - [Functions](#functions)
  - [Rules for contracts](#rules-for-contracts)
  - [Examples](#examples)
      - [Contract locked by single pubKey](#contract-locked-by-single-pubkey)
      - [Loan on collateral](#loan-on-collateral)

## Introduction

Equity is a high-level language designed for expressing contract programs that protect value on [Bytom](https://github.com/Bytom/bytom) blockchain. By writing and deploying smart contracts, you can manage the various assets on Bytom.

Here provides a brief introduction to the assets on Bytom, before we dive into the details of Equity:

* Bytom adopts the model of BUTXO, which maintains a public ledger with different types of UTXOs on blockchain.

* Each UTXO has two important attributes: `asset_id` and `amount`, which represents the `type` and `amount` of an asset. Generally, the `amount` here is called `value`, while sometimes UTXO is also abstractly referred to as a `value`.

* All `value(UTXO)` is locked by its corresponding contract program `program`. Only the input that satisfies the conditions defined in `program` can unlock `value` locked by `program`.

Therefore, writing an Equity smart contract is aimed at "describing with which smart contracts that assets are locked, and defining the conditions to unlocked specified assets."

## Contract

An Equity program consists of a contract, defined with the `contract` keyword. A contract definition has the form:

- `contract` _ContractName_ `(` _parameters_ `)` `locks` _value_ `{` _clauses_ `}`

To be specific:

* _ContractName_ is an identifier, a name for the contract, defined by yourself

* _parameters_ is the list of contract parameters, and the type of these parameters should be within the range of [Types of data](#types-of-data)

* _value_ is an identifier, a name for the value locked by the contract, defined by yourself

* _clauses_ is a list of one or more clauses

## Clause

Each clause describes one way to unlock the value in the contract, together with any data and/or payments required. A clause definition has the form:

- `clause` _ClauseName_ `(` _parameters_ `)` `{` _statements_ `}`

or like this:

- `clause` _ClauseName_ `(` _parameters_ `)` `requires` _payments_ `{` _statements_ `}`

To be specific:

* _ClauseName_  is an identifier, a name for the clause, defined by yourself

* _parameters_ is the list of clause parameters, and the type of these parameters should be within the range of [Types of data](#types-of-data)

* _payments_ is a list of [required payments](#required-payments)

* _statements_ is a list of one or more [statements](#statements). Each statement in a clause is either a `verify`, a `lock`, or an `unlock`. 

## Contract and clause parameters

Contract and clause parameters have **names** and **types**. A parameter is written as:

- _name_ `:` _TypeName_

and a list of parameters is:

- _name1_ `:` _TypeName1_ `,` _name2_ `:` _TypeName2_ `,` ...

Adjacent parameters sharing the same type may be coalesced like so for brevity:

- _name1_ `,` _name2_ `,` ... `:` _TypeName_

So that these two contract declarations are equivalent:

- `contract LockWithMultiSig(key1: PublicKey, key2: PublicKey, key3: PublicKey)`
- `contract LockWithMultiSig(key1, key2, key3: PublicKey)`

Available types are:

- `Integer` `Amount` `Boolean` `String` `Hash` `Asset` `PublicKey` `Signature` `Program`

These types are described in [Types of data](#types-of-data) below.

## Required payments

In some cases， the payment of some other value may be required to unlock the value in a contract, such as when dollars are traded for euros, dollars are required for unlocking euros.

Here the clause must use the `requires` syntax to give a name to the required value and to specify its amount and asset type:

- `clause` _ClauseName_ `(` _parameters_ `)` `requires` _name_ `:` _amount_ `of` _asset_

To be specified:

* _name_ is an identifier, a name for the value supplied by the transaction unlocking the contract.

* _amount_ is an [expression](#expressions) of type `Amount`.

* _asset_ is an expression of type `Asset`.

Some clauses require two or more payments in order to unlock the contract. Multiple required payments can be specified after `requires` like so:

- ... `requires` _name1_ `:` _amount1_ `of` _asset1_ `,` _name2_ `:` _amount2_ `of` _asset2_ `,` ...

## Statements

The body of a clause contains one or more statements:

* `verify` statements test the (bool) value of contract and clause arguments

* `unlock` statements can be used to unlock contract value

* `lock` statements can be used to lock contract and clause value with new programs

### Verify statements

A `verify` statement has the form:

- `verify` _expression_

The expression must have `Boolean` type. Every `verify` in a clause must evaluate as true in order for the clause to succeed.

Examples:

- `verify above(blockNumber)` tests that the current block height is above of `blockNumber`.
- `verify checkTxSig(key, sig)` tests that a given signature matches a given public key _and_ the transaction unlocking this contract.
- `verify newBid > currentBid` tests that one amount is strictly greater than another.

### Unlock statements

Unlock statements only ever have the form:

- `unlock` _value_

where _value_ is the name given to the contract value after the `locks` keyword in the `contract` declaration. This statement releases the contract value for any use; i.e., without specifying the new contract that must lock it.

### Lock statements

Lock statements have the form:

- `lock` _value_ `with` _program_

This locks _value_ (the name of the contract value, or any of the clause’s required payments) with _program_, which is an expression that must have the type `Program`.

## Types of data

The following data types are supported in Equity compiler:

* **integer**

  * **Amount**: The amount of assets (more precisely, the amount of assets counted in the smallest unit), ranges in 0 ~ 2^63. Variable of type `Amount` represents the asset locked in the contract

  * **Integer**: An integer between 2^-63 ~ 2^63

* **boolean**

  * **Boolean**: Value is `true` or `false`

* **string**

  * **String**: Byte string

  * **Hash**: Hashed byte string

  * **Asset**: Asset ID, or type of asset

  * **PublicKey**: Public key

  * **Signature**: Message signed by a private key

  * **Program**: Byte code expressed in bytes

## Expressions

Equity supports a variety of expressions for use in `verify` and `lock` statements as well as the `requires` section of a clause declaration.

Each _expr_ represents an _expression_. For example, _expr1_ `+` _expr2_ can be regarded as `20 + 15` or some more complicated expressions.

- `-` _expr_ : Negates a numeric expression
- `~` _expr_ : Inverts the bits in a byte string

Each of the following requires numeric operands (`Integer` or `Amount`) and produces a `Boolean` result:

- _expr1_ `>` _expr2_ : Tests whether _expr1_ is greater than _expr2_
- _expr1_ `<` _expr2_ : Tests whether _expr1_ is less than _expr2_
- _expr1_ `>=` _expr2_ : Tests whether _expr1_ is greater than or equal to _expr2_
- _expr1_ `<=` _expr2_ : Tests whether _expr1_ is less than or equal to _expr2_
- _expr1_ `==` _expr2_ : Tests whether _expr1_ is equal to _expr2_
- _expr1_ `!=` _expr2_ : Tests whether _expr1_ is not equal _expr2_

These operate on byte strings and produce byte string results:

- _expr1_ `^` _expr2_ : Produces the bitwise XOR of its operands
- _expr1_ `|` _expr2_ : Produces the bitwise OR of its operands
- _expr1_ `&` _expr2_ : Produces the bitwise AND of its operands

These operate on numeric operands (`Integer` or `Amount`) and produce a numeric result:

- _expr1_ `+` _expr2_ : Adds its operands
- _expr1_ `-` _expr2_ : Subtracts _expr2_ from _expr1_
- _expr1_ `*` _expr2_ : Multiplies its operands
- _expr1_ `/` _expr2_ : Divides _expr1_ by _expr2_
- _expr1_ `%` _expr2_ : Produces _expr1_ modulo _expr2_
- _expr1_ `<<` _expr2_ : Performs a bitwise left shift on _expr1_ by _expr2_ bits
- _expr1_ `>>` _expr2_ : Performs a bitwise right shift on _expr1_ by _expr2_ bits

Other expression types:

- `(` _expr_ `)` is _expr_
- _expr_ `(` _arguments_ `)` is a function call, where _arguments_ is a comma-separated list of expressions; see [functions](#functions) below
- a bare identifier is a variable reference
- `[` _exprs_ `]` is a list literal, where _exprs_ is a comma-separated list of expressions (presently used only in `checkTxMultiSig`)
- a sequence of numeric digits optionally preceded by `-` is an integer literal
- a sequence of bytes between single quotes `'...'` is a string literal
- the prefix `0x` followed by 2n hexadecimal digits is also a string literal representing n bytes (Note: Each hexadecimal number is 4 digits and each character is 8 digits)

## Functions

Equity includes several built-in functions for use in `verify` statements and elsewhere.

- `abs(n)` : Takes a number `n` and produces its absolute value.
- `min(x, y)` : Takes two numbers `x, y` and produces the smaller one.
- `max(x, y)` : Takes two numbers `x, y` and produces the larger one.
- `size(s)` : Takes an expression of any type and produces its `Integer` size in bytes.
- `concat(s1, s2)` : Takes two strings `s1, s2` and concatenates them to produce a new string.
- `concatpush(s1, s2)` : Takes two strings `s1, s2` and produces the concatenation of `s1` followed by the Bytom VM opcodes needed to push `s2` onto the Bytom VM stack. It is typically used to construct new BVM programs out of pieces of other ones. See [the BVM specification](https://github.com/boy-good/Docs/wiki/BTM-VM-OPS---%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6ops%E6%8C%87%E4%BB%A4%E8%AF%B4%E6%98%8E#%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%A4%84%E7%90%86%E6%93%8D%E4%BD%9C%E7%A0%81).
- `below(height)` : Takes an `Integer` and returns a `Boolean` telling whether the current block height is below of `height`.
- `above(height)` : Takes an `Integer` and returns a `Boolean` telling whether the current block height is above of `height`.
- `sha3(s)` : Takes a byte string `s` and produces its SHA3-256 hash (with type `Hash`).
- `sha256(s)` : Takes a byte string `s` and produces its SHA-256 hash (with type `Hash`).
- `checkTxSig(key, sig)` : Takes a `PublicKey` and a `Signature` and returns a `Boolean` telling whether `sig` matches both `key` _and_ the unlocking transaction.
- `checkTxMultiSig([key1, key2, ...], [sig1, sig2, ...])` : Takes one list-literal of `PublicKeys` and another of `Signatures` and returns a `Boolean` that is true only when every `sig` matches both a `key` _and_ the unlocking transaction. Ordering matters: not every key needs a matching signature, but every signature needs a matching key, and those must be in the same order in their respective lists.

## Rules for contracts

An Equity contract is correct only if it obeys all of the following rules:

- Identifiers must not collide. For example, a clause parameter must not have the same name as a contract parameter. (However, two different clauses may reuse the same parameter name; that’s not a collision.)
- Every contract parameter must be used in at least one clause.
- Every clause parameter must be used in its clause.
- Every clause must dispose of the contract value with a `lock` or an `unlock` statement.
- Every clause must also dispose of all clause values with a `lock` statement for each. (If required payments exist)

## Examples

### Contract locked by single pubKey

Here is a contract `LockWithPublicKey`, one of the simplest possible contracts. By reading this document, you are able to learn more details about how it works.

```
contract LockWithPublicKey(pubKey: PublicKey) locks value {
  clause spend(sig: Signature) {
    verify checkTxSig(pubKey, sig)
    unlock value
  }
}
```

The name of this contract is `LockWithPublicKey`. It locks some asset, called `value`. The argument `pubKey` must be specified as the one parameter for `LockWithPublicKey` in the transaction that locking the value (the transaction that create this contract).

`LockWithPublicKey` has one clause, which means one way to unlock `value`: try `spend` with a `Signature` as an argument.

The `verify` in `spend` checks that `Signature(sig)` matches both `publicKey` and the new transaction trying to unlock `value`. If that succeeds, then `value` is unlocked.

### Loan on collateral

Here is a more challenging example: called `LoanCollateral`.

```
contract LoanCollateral(assetLoaned: Asset,
                        amountLoaned: Amount,
                        repaymentHeight: Integer,
                        lender: Program,
                            borrower: Program) locks collateral {
  clause repay() requires payment: amountLoaned of assetLoaned {
    lock payment with lender
    lock collateral with borrower
  }
  clause default() {
    verify above(repaymentHeight)
    lock collateral with lender
  }
}
```

The name of this contract is `LoanCollateral`. It locks some value called `collateral`. There are five arguments must be specified as parameters for `LoanCollateral`: `assetLoaned`, `amountLoaned`, `repaymentDue`, `lender`, and `borrower`.

The contract has two clauses, which means two ways to unlock `collateral`:

- `repay()` requires no data but does require payment of `amountLoaned` units of `assetLoaned`
- `default()` requires no payment or data

The intent of this contract is: `lender` loans `collateral` to `borrower` in the forms of atomic swap. The required payment is paid to `lender`, as long as the collateral is transfered to `borrower`. But if the payment deadline passes, `lender` is entitled to claim `collateral` for him or herself.

The statements in `repay()` send the payment to the lender, and the collateral to the borrower with a simple pair of `lock` statements. Recall that "sending" value "to" a blockchain participant actually means locking the payment with a program that allows the recipient to unlock it.

The `verify` in `default()` ensures that `collateral` could be locked with `lender` by `lock` statement if deadline has passed. Note that this does not happen automatically when the deadline passes. The lender (or someone) must explicitly unlock `collateral` by constructing a new transaction that invokes the `default()` clause of `LoanCollateral`.