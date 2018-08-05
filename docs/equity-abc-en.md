# The Equity Language

## 目录

<!-- TOC -->

- [The Equity Language](#the-equity-language)
    - [目录](#目录)
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

<!-- /TOC -->

## Introduction

Equity 是一种高级语言，专门用来编写运行在 [Bytom](https://github.com/Bytom/bytom) 上的合约程序。通过编写、部署 Equity 智能合约，可以对 Bytom 上的各类资产进行操作。

Equity is a high-level language designed for expressing contracts, programs that protect value on Bytom blockchain. By writing and deploying smart contracts, you can manage the various assets on Bytom.

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

To unlock the value in a contract, a clause sometimes requires the presence of some other value, such as when dollars are traded for euros, dollars are required for unlocking euros.

In this case, the clause must use the `requires` syntax to give a name to the required value and to specify its amount and asset type:

- `clause` _ClauseName_ `(` _parameters_ `)` `requires` _name_ `:` _amount_ `of` _asset_

Here, _name_ is an identifier, a name for the value supplied by the transaction unlocking the contract; _amount_ is an [expression](#expressions) of type `Amount`; and _asset_ is an expression of type `Asset`.

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

Equity 编译器支持以下数据类型：

* **integer**

  * **Amount**: The amount of assets (more precisely, the amount of assets counted in the smallest unit), ranges in 0 ~ 2^63. The variable of type `Amount` represents the asset locked in the contract

  * **Integer**: An integer between 2^-63 ~ 2^63

* **boolean**

  * **Boolean**: Value is `true` or `false`

* **string**

  * **String**: Byte string

  * **Hash**: Hashed byte string

  * **Asset**: Asset ID, or type of asset

  * **PublicKey**: Public key

  * **Signature**: Message signed by private key

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

- `(` _expr_ `)` : 依然表示 _expr_ 本身
- _expr_ `(` _arguments_ `)` : 表示函数调用，传入的参数 _arguments_ 是一些用逗号分隔的表达式，具体可查阅下面的 [函数 (functions)](#函数-(functions))
- 单独出现的标识符表示变量本身
- `[` _exprs_ `]` : 表示一个列表(list)，其中 _exprs_ 是一些用逗号分隔的表达式(列表参数目前仅用于`checkTxMultiSig`)

- 以 `-` 开头的一串数字序列属于 `Integer` 类型
- 单引号 `'...'` 之间的字节序列(bytes)表示一个字符串
- 前缀 `0x` 后跟 2n 个十六进制数字，则表示 n 字节长的字符串 (注：每个十六进制数为4位，每个字符为8位)

- `(` _expr_ `)` is _expr_
- _expr_ `(` _arguments_ `)` is a function call, where _arguments_ is a comma-separated list of expressions; see [functions](#functions) below
- a bare identifier is a variable reference
- `[` _exprs_ `]` is a list literal, where _exprs_ is a comma-separated list of expressions (presently used only in `checkTxMultiSig`)
- a sequence of numeric digits optionally preceded by `-` is an integer literal
- a sequence of bytes between single quotes `'...'` is a string literal
- the prefix `0x` followed by 2n hexadecimal digits is also a string literal representing n bytes (Note: Each hexadecimal number is 4 digits and each character is 8 digits)

## Functions

Equity includes several built-in functions for use in `verify` statements and elsewhere.

- `abs(n)` 返回数字 n 的绝对值。
- `min(x, y)` 返回 x 和 y 两数中较小的那个数的值。
- `max(x, y)` 返回 x 和 y 两数中较大的那个数的值。
- `size(s)` 传入一个任意类型的表达式，返回其以字节为单位时的长度，返回值类型是 `Integer`。
- `concat(s1, s2)` 对两个字符串做拼接，从而生成一个新字符串。
- `concatpush(s1, s2)` 传入两个字符串 s1 和 s2，返回的结果是 s1 与一段 Bytom 虚拟机操作码(BVM opcodes) 的拼接，这段操作码的作用是将 s2 送入 Bytom 虚拟机的栈。此函数通常用于嵌套合约中。具体可参考 [BVM 操作码](https://github.com/boy-good/Docs/wiki/BTM-VM-OPS---%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6ops%E6%8C%87%E4%BB%A4%E8%AF%B4%E6%98%8E#%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%A4%84%E7%90%86%E6%93%8D%E4%BD%9C%E7%A0%81)。
- `below(height)` 传入 `Integer` 类型的值，返回 `Boolean` 型的值，判断当前区块高度是否低于参数 height，如果是则返回 true，否则返回 false。
- `above(height)` 传入 `Integer` 类型的值，返回 `Boolean` 型的值，判断当前区块高度是否高于参数 height。
- `sha3(s)` 传入 `string` 类的值，返回其 SHA3-256 的哈希值(返回值类型为`Hash`)。
- `sha256(s)` 传入 `string` 类的值，返回其 SHA-256 的哈希值(返回值类型为`Hash`)。
- `checkTxSig(key, sig)` 传入 `PublicKey` 与 `Signature`，返回 `Boolean` 型的值，以检测签名 `sig` 是否与公钥 `key` 和解锁交易匹配。
- `checkTxMultiSig([key1, key2, ...], [sig1, sig2, ...])` 分别传入列表形式的 `PublicKeys` 与 `Signatures`，返回 `Boolean` 型的值，以检测是否每个 `sig` 都与 `key` 以及解锁交易匹配。关于列表参数顺序的问题：并非所有公钥都需要有签名与其匹配，但所有签名都应有匹配的公钥，并且这些公钥/签名必须在各自的列表中按相同的顺序排列。

## Rules for contracts

只有遵循以下规则的 Equity 合约才是有效的：

- 标识符不能互相冲突。例如，clause 参数的名称不能与 contract 参数的名称相同。(但是，两个不同的 clause 可以使用相同的参数名，这不构成冲突)
- 每个 contract 参数必须至少在一个 clause 中被使用。
- 每个 clause 参数必须在 clause 内被使用。
- 每个 clause 必须使用 `lock` 或 `unlock` 语句处理合约中锁定的 value。
- 每个 clause 还必须用 `lock` 语句处理所有的支付给 clause 的 value(如果有的话)。

## Examples

### Contract locked by single pubKey

下面这个合约叫做 `LockWithPublicKey`，是最简单的合约之一。通过阅读本文档，可以详细了解它是如何工作的。

```
contract LockWithPublicKey(pubKey: PublicKey) locks value {
  clause spend(sig: Signature) {
    verify checkTxSig(pubKey, sig)
    unlock value
  }
}
```

这个合约的名称是 `LockWithPublicKey`。它锁定了一笔资产，被称作 `value`。在锁定 value 的交易(即创建合约的交易)中必须为 `LockWithPublicKey` 指定合约参数，即`pubKey`。

`LockWithPublicKey` 包含一个 clause，这意味着有一种(且只有一种)途径解锁 `value`：把 `Signature` 作为参数，调用 `spend` 条款函数。

`spend` 条款中的 `verify` 检查传入的 `Signature(sig)`是否与 `pubKey` 和尝试解锁 value 的交易匹配。如果检查成功，则 `value` 被解锁。

### Loan on collateral

接下来是一个更具挑战性的例子：叫做 `LoanCollateral`。

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

合约的名称是 `LoanCollateral`。它锁定了一些被称为 `collateral` 的 value。在创建此合约的交易中，必须指定五个合约参数：`assetLoaned`、`amountLoaned`、`repaymentHeight`、`lender` 和 `borrower`。

此合约包含两个 clause，也就是说，有两种途径来解锁 `collateral`：

- `repay()` 不需要传入数据，但需要支付与 `amountLoaned` 等量的资产 `assetLoaned`
- `default()` 既不需要传入数据，也不需要支付资产

合约的含义是：贷方 `lender` 在借方 `borrower` 已出具抵押物 `collateral` 的情况下，将与 `amountLoaned` 等量的资产 `assetLoaned` 借给借方。如果贷款被偿还给贷方，则将抵押物退还给借方。但是，如果还款截止时间已过，则贷方有权为自己索取抵押物。

`repay()` 条款函数通过两个简单的 `lock` 语句将付款发送给贷方，且将抵押物发送给借方。回想一下，向区块链参与者发送 value，其实就是将 value 锁定至允许接收者解锁的 program。

`default()` 条款函数的 `verify` 语句确保在截止区块高度已过时，可以将抵押物 `collateral` 锁定给贷方 `lender`。需要注意的是，这一过程并不是在截止时间到达的时刻自动发生的。贷方(或某个人)必须构造一个调用 `LoanCollateral` 合约的 `default()` 条款的交易，才能解锁抵押物 `collateral` 并将其重新锁定给 `lender`。
