## Solidity Source File structure

The source file can contain any number of contract definitions, import instructions, and miscellaneous instructions.

### Version miscellaneous note

To avoid being compiled by compilers that may introduce incompatible changes in the future, the source file can (and should) be annotated by so-called version annotations. We try to make such changes as small as possible. In particular, we need to introduce changes in a way that syntax must be modified synchronously when modifying semantics. Of course, this is sometimes difficult to do. Therefore, it is always a good way to read through the change log for versions with major changes at least. The versions of these versions are always `0.x.0` Or `x.0.0` The form.

Version miscellaneous notes are used as follows::

```sh
pragma solidity ^0.4.0;
```

In this way, the source file does not allow compiler compilation earlier than Version `0.4.0` or higher than (including) `0.5.0` Compiler compilation of the version (the second condition is due to the use ^ Added). The consideration of this approach is that the compiler will not have major changes before version `0.5.0`, so it can ensure that the source code is always compiled as expected. In the preceding example, the specific version number of the compiler is not fixed, so the compiler patch version can also be used.

You can use more complex rules to specify the compiler version and the expression follows [npm](https://docs.npmjs.com/misc/semver) version semantics.


>  Pragma 是 pragmatic information 的简称，微软 Visual C++ [文档](https://msdn.microsoft.com/zh-cn/library/d9x1s805.aspx) 中译为杂注。Solidity 中沿用 C ，C++ 等中的编译指令概念，用于告知编译器 **如何** 编译。  ——译者注

### Import other source files

#### Grammar and semantics

Although Solidity does not know what "default export" is, the syntax of import statements supported by Solidity is very similar to JavaScript (starting from ES6).

> ES6 即 ECMAScript 6.0，ES6是 JavaScript 语言的下一代标准，已经在 2015 年 6 月正式发布。 ——译者注

At the global level, you can use import statements in the following format：

```sh
  import "filename";
```

This statement imports all global symbols from "filename" into the current global scope (unlike ES6,Solidity is backward compatible).

```sh
  import * as symbolName from "filename";
```
Create a new global symbol `symbolName` , whose members are all from `filename` Global symbol in.

```sh
  import {symbol1 as alias, symbol2} from "filename";
```
Create a new global symbol ``alias`` And ``symbol2``，respectively from ``"filename"`` reference ``symbol1`` and ``symbol2`` 。

Another syntax does not belong to ES6, but may be simpler:

```sh
  import "filename" as symbolName;
```

This statement is equivalent `import * as symbolName from "filename";`。

#### Path

The filename mentioned above is always processed by path `/` as a directory separator, `.` mark the current directory ``..`` Indicates the parent directory. 
When `.` or `..` the following characters are `/` they can be treated as the current directory or parent directory.
Only the path to the current directory `.` or parent directory `..` at the beginning, it can be regarded as a relative path.


For `import "./x" as x` statement to import files under the same directory as the current source file `x` 。
If used `import "x" as x;` instead, different files may be introduced (in the global `include directory` Medium).

The final file to be imported depends on how the compiler (see below) parses the path.Generally, directory levels do not need to be strictly mapped to local file systems,It can also be mapped to resources that can be discovered through ipfs,http, or git.

#### Use in actual compilers


When running the compiler, it can specify not only how to discover the first element of the path, but also the path prefix|remapping|。
For example，``github.com/ethereum/dapp-bin/library`` will be replayed to shoot ``/usr/local/dapp-bin/library`` ，
The compiler reads the file from the remapping location. If multiple paths are remapped, the longest path is remapped first.
This allows ``""`` be mapped ``"/usr/local/include/solidity"`` to perform "rollback and remapping.
At the same time, these remaps depend on the context, allowing you to configure the packages to be imported, such as different versions of the same library.

**solc**:

For solc（command line compiler），these remaps are based on `context:prefix=target` parameters in the form are provided.
among them，``context:`` and  ``=target`` parts are optional（the target is prefix by default）。
All remapping values are compiled regular files (including their dependencies), and this mechanism is completely backward compatible（as long as the file name does not contain `=` or `:`），Therefore, this is not a destructive modification.

in `content` source code files in the directory or its subdirectories, all import statements `prefix` Import files starting with will be used `target` Replacement `prefix` To redirect.

For example, if you have cloned ``github.com/ethereum/dapp-bin/`` to local ``/usr/local/dapp-bin`` ，
used int the source file:

```sh
  import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;
```

Then run the compiler:

```sh
  solc github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ source.sol
```

For a more complex example, suppose you rely on some modules that use a very old version of dapp-bin.
The old version of dapp-bin has been checked out `/usr/local/dapp-bin_old` , at this time you can use:

```sh
  solc module1:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ \
  module2:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin_old/ \
  source.sol
```

In this way, `module2` all imports in point to the old version, and `module1` import in to obtain the new version.

Note that solc only allows files from a specific directory: they must be located in a explicitly specified source file directory (or subdirectory), or in a remapped target Directory (or subdirectory).

If you want to include files directly with an absolute path, just add a remapping `=/` .

If multiple remappings point to a valid file, the remapping with the longest common prefix is selected.

**Remix**:

[Remix](https://remix.ethereum.org/) Provides an automatic remapping for the github source code platform, which automatically obtains files through the network: for example, you can use `import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;` import a map iterator.

In the future, Remix may support other source code platforms.

### Annotation

you can use a single of comments（``//``）and multiple lines of comments（``/*...*/``）

```sh

  // 这是一个单行注释。

  /*
  这是一个
  多行注释。
  */
```

In addition, there is another annotation called natspec annotation, and its documentation has not yet been compiled.
They use three backslashes（``///``）or a block beginning with a double star（``/** ... */``）writing, they should be used directly on function declarations or statements.
Can be used in comments [Doxygen](https://en.wikipedia.org/wiki/Doxygen) style labels to document functions,
The condition for passing the label form verification, and provide a condition that is displayed to the user when the user attempts to call a function **Confirmation text**.

In the following example, we record the contract title, two input parameters, and two return values:

```sh

  pragma solidity ^0.4.0;

  /** @title 形状计算器。 */
  contract shapeCalculator {
      /** @dev 求矩形表明面积与周长。
      * @param w 矩形宽度。
      * @param h 矩形高度。
      * @return s 求得表面积。
      * @return p 求得周长。
      */
      function rectangle(uint w, uint h) returns (uint s, uint p) {
          s = w * h;
          p = 2 * (w + h);
      }
  }
```

## Contract structure

In Solidity, contracts are similar to classes in object-oriented programming languages.
Each contract can contain `状态变量`，`函数`，`函数修饰器`，`事件`，`结构类型` and `枚举类型` the contract can be inherited from other contracts.

### State variable

State variables are values permanently stored in contract storage.

```sh
    pragma solidity ^0.4.0;

    contract SimpleStorage {
        uint storedData; // 状态变量
        // ...
    }
```

Valid state variable types see `type` chapters, Possible options for state variable visibility see `可见性和getter函数` 。

### Function

A function is the executable unit of the code in a contract.

```sh
    pragma solidity ^0.4.0;

    contract SimpleAuction {
        function bid() public payable { // 函数
            // ...
        }
    }
```

`函数调用` can occur inside or outside the contract, and the function has different degrees of visibility to other contracts（ `可见性和getter函数`）。 


### Function modifier

Function modifiers can be used to improve function semantics in a declarative manner (see contract section `function` ).


```sh
    pragma solidity ^0.4.22;

    contract Purchase {
        address public seller;

        modifier onlySeller() { // 修饰器
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }
        
        function abort() public onlySeller { // Modifier usage
            // ...
        }
    }
```

### Event

Events are interfaces that can easily call the log function of Simplechain virtual machines.

```sh

    pragma solidity ^0.4.21;
    contract SimpleAuction {
        event HighestBidIncreased(address bidder, uint amount); // 事件

        function bid() public payable {
            // ...
            emit HighestBidIncreased(msg.sender, msg.value); // 触发事件
        }
    }
```

For information on how to declare events and how to use events in dapp see contracts `event`。

### Structure type

Structure is a custom type that can Group several variables (see type section `structure`）。

```sh

    pragma solidity ^0.4.0;

    contract Ballot {
        struct Voter { // 结构
            uint weight;
            bool voted;
            address delegate;
            uint vote;
        }
    }
```

### Enumeration type

Enumeration can be used to create a custom type consisting of a certain number of constant values（see type chapter `Enumeration type`）。 

```sh

    pragma solidity ^0.4.0;

    contract Purchase {
        enum State { Created, Locked, Inactive } // 枚举
    }
```

## Type

Solidity is a static type language, which means each variable (state variable and local variable) you need to specify the type of the variable at compile time (or at least you can deduce the variable type).
Solidity provides several basic types that can be used to combine complex types.

In addition, types can interact in expressions containing operator symbols.

### Value type

The following types are also called value types because variables of these types are always passed by value.
That is, when these variables are used as function parameters or in assignment statements, values are always copied.

### Boolean type

``bool`` ：the possible value is a literal constant value ``true`` and ``false`` 。

Operator:

*  ``!`` （logical no）
*  ``&&`` （logic and， "and" ）
*  ``||`` （logic or， "or" ）
*  ``==`` （equal to）
*  ``!=`` （not equal to）

Operator ``||`` and ``&&`` all follow the same short-counterfeiting rule. that means in the expression ``f(x) || g(y)`` middle，
If ``f(x)`` Value of  ``true`` ，then ``g(y)`` will not be executed even if there are some side effects.

### Integer

``int`` / ``uint`` ：integer variables representing different numbers of signed and unsigned digits respectively.
Supported keywords ``uint8`` to ``uint256`` （unsigned, from 8 bits to 256 bits）and ``int8`` to ``int256``，``8`` bit is the step increment.
``uint`` and ``int`` respectively ``uint256`` and ``int256`` the alias.

Operator:

* Comparison operator: ``<=`` ， ``<`` ， ``==`` ， ``!=`` ， ``>=`` ， ``>`` （return a boolean value）
* bit operator： ``&`` ， ``|`` ， ``^`` （different or）， ``~`` （reverse bit）
* Arithmetic operator:： ``+`` ， ``-`` ， unary operation ``-`` ， unary operation ``+`` ， ``*`` ， ``/`` ， ``%`` （Surplus） ， ``**`` （Power）， ``<<`` （Left Shift） ， ``>>`` （Right Shift）

Division is always truncated（only compiled as in EVM `DIV`` Operation Code），
But if all the operands are `字面常数（literals)`(Or literal constant expression), it will not be truncated.

Dividing by zero or modulo zero will cause runtime exceptions.

The result of the shift operation depends on the type to the left of the operator.
expression ``x << y`` and ``x * 2**y`` is equivalent,
``x >> y`` 与 ``x / 2**y`` Is equivalent. This means that shifting a negative number will cause its symbol to disappear.

.. warning::
   The result generated by the right shift of the negative value of the signed integer type is different from that produced in other languages.
In Solidity, the right shift and division are equivalent, so the right shift of a negative number will result in the rounded (truncated) to 0.
In other languages, moving the negative number to the right is similar to Rounding (to the negative infinity).

### Fixed-length floating point type

> Solidity does not fully support the fixed-length floating point type. You can declare floating-point variables of fixed length, but you cannot assign them or assign them to other variables.

``fixed`` / ``ufixed``：indicates signed and unsigned fixed-length floating point types of various sizes.
In keywords ``ufixedMxN`` and ``fixedMxN`` Middle，``M`` Indicates the number of digits occupied by this type,``N`` Indicates the number of available decimal places.
``M`` Must be able to divide exactly 8, that is, 8 to 256 bits.
``N`` It can be any number from 0 to 80.
``ufixed`` and ``fixed`` fixed ``ufixed128x19`` and ``fixed128x19`` alias。

Operator：

* Comparison operator：``<=``， ``<``， ``==``， ``!=``， ``>=``， ``>`` （return value is boolean）
* Arithmetic operator：``+``， ``-``， unary operation ``-``， unary operation ``+``， ``*``， ``/``， ``%`` （take the remainder）

.. note::
   Floating point type (in many languages `float` And `double` The biggest difference between type, more precisely IEEE 754 type) and fixed-length floating point type is, In the former, the number of digits required for the integral part and the decimal part (the part after the decimal point) is flexible and variable, while the length of these two parts in the latter is strictly regulated. Generally speaking, in the floating point type, almost the whole space is used to represent numbers, but only a few bits represent the position of decimal point.

### Address type

``address``：The address type stores a 20-byte value (the size of the Simplechain address).
The address type also has member variables and serves as the basis for all contracts.

Operator：

* ``<=``， ``<``， ``==``， ``!=``， ``>=`` and ``>``

>  Starting from version 0.5.0, the contract does not derive from the address type, but can still be explicitly converted to the address type.

Address type member variable

* ``balance`` 和 ``transfer``

can used ``balance`` property to query the balance of an address,
can also used ``transfer`` function sends | ether | (in wei) to an address:

```bash
address x = 0x123;
address myAddress = this;
if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
```

> If ``x`` is a contract address, its code (more specifically, its fallback function, if any) and ``transfer`` Function calls are executed together (this is a feature of EVM and cannot be prevented).
If gas is used up in the execution process or the execution fails for any reason, the | ether | Transaction will be called back, and the current contract will also throw an exception at the same time of termination.

* ``send``

`send` is `transfer` the low-level version. If execution fails, the current contract will not be terminated due to exceptions, `send` 会返回 `false`.

> In use `send` there are some risks：if the call stack depth is 1024, it will lead to sending failure (which can always be forced by the caller), and if the receiver uses up gas, it will also lead to sending failure. Therefore, in order to ensure the security of | ether | Sending, you must check ``send`` the return value of, using ``transfer`` or a better way: Use a mode in which the receiver can retrieve funds.

* ``call``， ``callcode`` and ``delegatecall``

In addition, in order to interact with contracts that do not conform to | ABI |, there are contracts that can accept any type and any number of parameters call Function. These parameters are packaged into a continuous area of 32 bytes. One exception is when the first parameter is encoded into exactly 4 bytes. In this case, this parameter is not followed by subsequent parameter encoding to allow the use of function signatures.

```bash
    address nameReg = 0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2;
    nameReg.call("register", "MyName");
    nameReg.call(bytes4(keccak256("fun(uint256)")), a);
```

``call`` The returned Boolean value indicates that the called function has been executed（``true``）or an EVM exception is thrown（``false``）。
The real data returned cannot be accessed (for this we need to know the encoding and size in advance).

Can be used `.gas()` | modifier | Adjust the quantity of gas provided:

    namReg.call.gas(1000000)("register", "MyName");

Similarly, the value of | ether | Provided can also be controlled:

```bash
   nameReg.call.value(1 ether)("register", "MyName"); 
```

Finally, these | modifier | Can be used together. The order in which each modifier appears is not important:

```bash
   nameReg.call.gas(1000000).value(1 ether)("register", "MyName"); 
```

gas or value cannot be used in overloaded functions at present. One solution is to introduce a special case to gas and values and re-check whether they appear at heavy loads.

Similarly, it can also be used ``delegatecall``：The difference is that only the code of the given address is used, and other attributes (storage, balance,…) All are taken from the current contract. ``delegatecall`` the purpose is to use the library code stored in another contract. You must ensure that the storage structures in both contracts apply to delegatecall. Before the homestead version, there was only one with similar functions but limited functions ``callcode`` the function of is available, but it cannot obtain the entrusting party's ``msg.sender`` and ``msg.value``。

These three functions ``call``， ``delegatecall`` and ``callcode`` they are all very low-level functions and should only be regarded Last move To use, because they undermine the type security of Solidity.

> All contracts inherit the member variable of address, so it can be used ``this.balance`` query the balance of the current contract.

> Use is not encouraged ``callcode``，it will also be removed in the future.

> These three functions are all low-level functions and need to be used with caution. Specifically, any unknown contract can be malicious. When you call a contract, you give control to it. It can call your contract in turn. Therefore, when the call returns, be prepared for changes in your state variables.

### Fixed-length byte array

Keywords: ``bytes1``， ``bytes2``， ``bytes3``， ...， ``bytes32``，``byte`` is ``bytes1`` alias。

Operator：

* Comparison operator: ``<=``， ``<``， ``==``， ``!=``， ``>=``， ``>`` （return boolean）
* Bit operator: ``&``， ``|``， ``^`` （bit or）， ``~`` （reverse by position）， ``<<`` （left shift）， ``>>`` （right shift）
* Index access: if ``x`` is ``bytesI`` type，then ``x[k]`` （Among them ``0 <= k < I``） return  ``k`` bytes（read-only）。

This type can be shifted to any integer type as the right operand (but the returned result type is the same as the left operand type), and the right operand indicates the number of digits to be moved.
A negative displacement operation will cause a runtime exception.

Member variables:

* ``.length`` indicates the length of the byte array (read-only).

> Can put ``byte[]`` it is used as a byte array, but this method wastes a lot of storage space. To be exact, each element wastes 31 bytes when calling in. A better approach is to use ``bytes``。

### Variable length byte array

``bytes``: Variable length byte array，It is not a value type.
``string``: Variable length UTF-8 encoding string type, It is not a value type.

### Address literal constant（Address Literals）

For example `0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF` such hexadecimal literal constants that pass the address checksum test belong ``address`` type. The hexadecimal literal constant, which is 39 to 41 numbers in length and fails to pass the checksum test and generates a warning, is regarded as a normal rational literal constant.

> Mixed case address checksum format is defined in [EIP-55](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md)medium.

### Literal constants of rational numbers and integers

The literal constant of an integer consists of a string of numbers ranging from 0 to 9 and is expressed as decimal. For example, `69` Indicates the number 69. `Solidity` There is no octal in, so the pre -0 is invalid.

Decimal face constant with one ``.``，at least there will be a number on one side. For example:``1.``，``.1``，and ``1.3``。

Scientific symbols are also supported. Although the index must be an integer, the base number can be a decimal. For example:``2e10``， ``-2e10``， ``2e-10``， ``2.5e1``。

Numeric literal constant expressions themselves support arbitrary precision unless they are converted to non-literal constant types (that is, conversion occurs when they appear in non-literal constant expressions). This means that in numerical constant expressions, calculations do not overflow and division does not truncate.

For example， ``(2**800 + 1) - 2**800`` result is a literal constant ``1`` （belong ``uint8`` type），although the intermediate result of the calculation has exceeded the machine word length of |evm| . In addition, `.5 * 8` result is integer `4` （Although there are non-integers involved in the calculation）.

As long as the operands are integers, operators supported by any integer can be applied to literal constant expressions of numerical values.
If either of the two numbers is a decimal, bit operation is not allowed. If the index is a decimal, power operation is not supported (because this may result in an irrational number).

> Solidity has a corresponding literal constant type for each rational number. Integer literal constant and rational number literal constant both belong to the type of numerical literal constant. In addition, all numerical literal constant expressions (expressions containing only numerical literal constants and operators) belong to the type of numerical literal constants.
Therefore, the literal constant expression of the numerical value `1 + 2` And `2 + 1` The result is the same as the literal constant type of the value of rational number three.

> In earlier versions, the division of the literal constant of an integer was also truncated, but in current versions, the result was converted into a rational number. That `5 / 2` Not equal `2` , but equal `2.5` .

> A numeric literal constant expression is converted to a non-literal constant type as long as it is used in a non-literal constant expression. In the following example, although we know `b` The value of is an integer, `2.5 + a` This part of the expression does not perform type check, so compilation cannot pass.

```sh

    uint128 a = 1;
    uint128 b = 2.5 + a + 0.5;

```

### String literal constant

The literal constant of a string refers to a string caused by double quotation marks or single quotation marks（``"foo"`` or ``'bar'``）。
Not like having an ending character in C language;``"foo"`` it is equivalent to 3 bytes instead of 4.
Like integer literal constants, the types of string literal constants can also be changed, but they can be converted ``bytes1``，……，``bytes32``，if appropriate, it can also be converted ``bytes`` and ``string``。

String literal constants support escape characters, such ``\n``，``\xNN`` and ``\uNNNN``。``\xNN`` represents a hexadecimal value, which is finally converted into an appropriate byte, and``\uNNNN`` indicates the Unicode encoded value, which is eventually converted to a sequence of UTF-8.


### Hexadecimal literal constant

Hexadecimal literal constants use keywords``hex`` a string that starts with single or double quotation marks (for example, hex "001122FF" ).
the content of the string must be a hexadecimal string, and their values will be represented in binary.

Hexadecimal literal constants are similar to string literal constants and have the same conversion rules.

### Enumeration type

```sh

    pragma solidity ^0.4.16;

    contract test {
        enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
        ActionChoices choice;
        ActionChoices constant defaultChoice = ActionChoices.GoStraight;

        function setGoStraight() public {
            choice = ActionChoices.GoStraight;
        }

        // 由于枚举类型不属于 |ABI| 的一部分，因此对于所有来自 Solidity 外部的调用，
        // "getChoice" 的签名会自动被改成 "getChoice() returns (uint8)"。
        // 整数类型的大小已经足够存储所有枚举类型的值，随着值的个数增加，
        // 可以逐渐使用 `uint16` 或更大的整数类型。
        function getChoice() public view returns (ActionChoices) {
            return choice;
        }

        function getDefaultChoice() public pure returns (uint) {
            return uint(defaultChoice);
        }
    }
```

### Function type

A function type is a type that represents a function. You can assign a function to a variable of another function type, pass a function as a parameter, and return a function type variable in a function call. There are two types of functions:- internal (internal) Function sum external (external) Function:

Internal functions can only be called within the current contract (more specifically, within the current code block, including internal library functions and inherited functions), because they cannot be executed outside the current contract context. Calling an internal function is implemented by redirecting to its entry label, just like calling a function inside the current contract.

An external function consists of an address and a function signature, which can be passed or returned by calling an external function.

The function type is expressed as follows:

```bash
function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]
```
Contrary to the parameter type, the return type cannot be empty-if the function type does not need to return, you need to delete the entire `returns` Part.

The function type is an internal function by default, so it does not need to be declared ``internal`` keywords.
On the contrary, the function itself in the contract is public by default, and it is an internal function by default only when it is regarded as a type name.

There are two ways to access the function in the current contract: one is to use its name directly``f`` ，other is to use ``this.f`` 。
The former applies to internal functions, while the latter applies to external functions.

An exception is thrown if a function type variable is called before initialization.
If in a function is ``delete`` same situation occurs after calling it.

If external function types are used outside the context environment of Solidity, they are considered `function` Type.
This type encodes the function address along with its function identifier as one `bytes24` Type.

Note that the public function of the current contract can be used either as an internal function or as an external function.
If you want to use a function as an internal function, use `f` Call, if you want to use it as an external function, use `this.f` .

In addition, the public (or external) function also has a special member variable called `selector` , can return`ABI function selector`

```bash
    pragma solidity ^0.4.16;

    contract Selector {
      function f() public view returns (bytes4) {
        return this.f.selector;
      }
    }
```

If use an example of an internal function type:

```bash
    pragma solidity ^0.4.16;

    library ArrayUtils {
      // 内部函数可以在内部库函数中使用，
      // 因为它们会成为同一代码上下文的一部分
      function map(uint[] memory self, function (uint) pure returns (uint) f)
        internal
        pure
        returns (uint[] memory r)
      {
        r = new uint[](self.length);
        for (uint i = 0; i < self.length; i++) {
          r[i] = f(self[i]);
        }
      }
      function reduce(
        uint[] memory self,
        function (uint, uint) pure returns (uint) f
      )
        internal
        pure
        returns (uint r)
      {
        r = self[0];
        for (uint i = 1; i < self.length; i++) {
          r = f(r, self[i]);
        }
      }
      function range(uint length) internal pure returns (uint[] memory r) {
        r = new uint[](length);
        for (uint i = 0; i < r.length; i++) {
          r[i] = i;
        }
      }
    }

    contract Pyramid {
      using ArrayUtils for *;
      function pyramid(uint l) public pure returns (uint) {
        return ArrayUtils.range(l).map(square).reduce(sum);
      }
      function square(uint x) internal pure returns (uint) {
        return x * x;
      }
      function sum(uint x, uint y) internal pure returns (uint) {
        return x + y;
      }
    }
```

Another example of using an external function type:

```bash
    pragma solidity ^0.4.11;

    contract Oracle {
      struct Request {
        bytes data;
        function(bytes memory) external callback;
      }
      Request[] requests;
      event NewRequest(uint);
      function query(bytes data, function(bytes memory) external callback) public {
        requests.push(Request(data, callback));
        NewRequest(requests.length - 1);
      }
      function reply(uint requestID, bytes response) public {
        // 这里要验证 reply 来自可信的源
        requests[requestID].callback(response);
      }
    }

    contract OracleUser {
      Oracle constant oracle = Oracle(0x1234567); // 已知的合约
      function buySomething() {
        oracle.query("USD", this.oracleResponse);
      }
      function oracleResponse(bytes response) public {
        require(msg.sender == address(oracle));
        // 使用数据
      }
    }
```

>  The introduction of Lambda expressions or inline functions is planned, but is not currently supported.

### Reference type

Compared with the value types discussed before, we need to be more cautious when dealing with complex types (that is, types that occupy more than 256 bits of space).
Since the overhead of copying these type variables is considerable, we have to consider its storage location and save them in `memory` (Not permanent storage), Or `storage` Where state variables are saved.

### Data Location

All complex types, namely Array And Structure Type, all have an additional attribute, "data location", indicating that the data is saved in `memory` Medium or storage Medium. Depending on the context, most of the time the data has a default location, but you can also add keywords after the type name. storage Or `memory` Modify. By default, the data location of function parameters (including returned parameters) is `memory` , the default data location of the local variable is `storage` , the data location of the state variable is forced to be `storage` (This is obvious).

There is also a third data location, `calldata` , this is a read-only and not permanently stored location, used to store function parameters. The data location of external function parameters (non-return parameters) is forcibly specified `calldata` , effect and `memory` Almost.

The designation of data locations is very important because they affect assignment behavior:

In `storage` And `memory` Assign values between two pairs, or `storage` Assigning values to state variables (even from other state variables) creates an independent copy. However, when a state variable assigns a value to a local variable, it only passes a reference, and this reference always points to a state variable, so the latter changes while the former changes. On the other hand, from one `memory` The reference type of the storage to another `memory` The reference type assignment of the storage does not create a copy.

```sh

    pragma solidity ^0.4.0;

    contract C {
        uint[] x; // x 的数据存储位置是 storage

        // memoryArray 的数据存储位置是 memory
        function f(uint[] memoryArray) public {
            x = memoryArray; // 将整个数组拷贝到 storage 中，可行
            var y = x;  // 分配一个指针（其中 y 的数据存储位置是 storage），可行
            y[7]; // 返回第 8 个元素，可行
            y.length = 2; // 通过 y 修改 x，可行
            delete x; // 清除数组，同时修改 y，可行
            // 下面的就不可行了；需要在 storage 中创建新的未命名的临时数组， /
            // 但 storage 是“静态”分配的：
            // y = memoryArray;
            // 下面这一行也不可行，因为这会“重置”指针，
            // 但并没有可以让它指向的合适的存储位置。
            // delete y;
            
            g(x); // 调用 g 函数，同时移交对 x 的引用
            h(x); // 调用 h 函数，同时在 memory 中创建一个独立的临时拷贝
        }

        function g(uint[] storage storageArray) internal {}
        function h(uint[] memoryArray) public {}
    }
```

**Summary**

Force the specified data location:
 - Parameters of external functions (excluding return parameters): calldata
 - Status variable: storage

Default data location:
 - Function parameters (including return parameters): memory
 - All other local variables: storage

#### Array

An array can be declared with a length specified or dynamically resized.
for `storage` element type can be arbitrary (that is, the element can also be an array type, mapping type, or structure).
for `memory` The element type cannot be a mapping type. If it is a parameter of the public function, it can only be a ABI type.

An element type is ``T``，fixed length is``k`` the array of can be declared ``T[k]``，while the dynamic array is declared ``T[]``.
For example, a length of 5 and an element type ``uint`` array of the dynamic array of, should be declared ``uint[][5]`` （Note that compared with other languages, the declared position of array length is reversed).
To access the second element of the third dynamic array, you should use x[2][1](the array subscript starts from 0, and the subscript order when accessing the array is opposite to that when declaring, that is, x[2] is reduced by one level from the right).

``bytes`` and ``string`` variables of type are special arrays.
``bytes`` similar  ``byte[]``，but it will be "tightly packaged" in calldata (Translator's note: elements are continuously stored together and will not be stored in a unit per 32 bytes).
``string`` and ``bytes`` Same, but (temporarily) access with length or index is not allowed.

.. note::
    If you want to access a string in bytes ``s``，please use``bytes(s).length`` / ``bytes(s)[7] = 'x';``。
    Note that you are accessing low-level bytes in the form of UTF-8 instead of a single character.

You can identify an array ``public``，so that Solidity can create a `getter`.
After that, you must use the digital subscript as a parameter to access getter.

**Create a memory array**

Available ``new`` keyword creates a variable length array in memory.
and `storage` opposite of the array is that you Can't By modifying member variables `length` change `memory` size of array

```sh

    pragma solidity ^0.4.16;

    contract C {
        function f(uint len) public pure {
            uint[] memory a = new uint[](7);
            bytes memory b = new bytes(len);
            // 这里我们有 a.length == 7 以及 b.length == len
            a[6] = 8;
        }
    }
```

**Array literal constant/inline array**

The literal constant of an array is an array in the form of writing expressions and is not immediately assigned to a variable.

```sh

    pragma solidity ^0.4.16;

    contract C {
        function f() public pure {
            g([uint(1), 2, 3]);
        }
        function g(uint[3] _data) public pure {
            // ...
        }
    }
```

The literal constant of an array is a fixed-length | memory | Array type. Its basic type is determined by the common type of elements in the array. For example，``[1, 2, 3]`` type of is ``uint8[3] memory``，because the type of each literal constant is ``uint8``。
Because of this, it is necessary to convert the first element in the above example ``uint`` type.
At present, it should be noted that the fixed length `memory` An array cannot be assigned to a longer `memory` Array, the following is a counter example:

```sh
    // 这段代码并不能编译。

    pragma solidity ^0.4.0;

    contract C {
        function f() public {
            // 这一行引发了一个类型错误，因为 unint[3] memory
            // 不能转换成 uint[] memory。
            uint[] x = [uint(1), 3, 4];
        }
    }
```
Such restrictions have been planned to be removed in the future, but the current array is ABI The problem of transmission in caused some trouble.

#### Members

**length**:

Array has ``length`` member variable indicates the length of the current array. Dynamic arrays can be in `storage`（not `memory` ）by changing member variables ``.length`` Change the array size. You cannot automatically extend the length of an array by accessing the length of the current array. Once created, `memory` The size of the array is fixed (but dynamic, that is, it depends on runtime parameters).

**push**:

Variable length | storage | Array and bytes Type (not `string` Type) all have one called push The member function of, which is used to attach new elements to the end of the array. This function returns the new array length.


>  Multidimensional arrays are not currently available in external functions.

>  Due to the limitation of | evm |, dynamic content cannot be returned through external function calls. For example, if you call web3.js ``contract C { function f() returns (uint[]) { ... } }`` In ``f`` Function，which returns some content，but cannot be achieved through Solidity.

At present, the only alternative is to use large static arrays.

```bash
    pragma solidity ^0.4.16;

    contract ArrayContract {
        uint[2**20] m_aLotOfIntegers;
        // 注意下面的代码并不是一对动态数组，
        // 而是一个数组元素为一对变量的动态数组（也就是数组元素为长度为 2 的定长数组的动态数组）。
        bool[2][] m_pairsOfFlags;
        // newPairs 存储在 memory 中 —— 函数参数默认的存储位置

        function setAllFlagPairs(bool[2][] newPairs) public {
            // 向一个 storage 的数组赋值会替代整个数组
            m_pairsOfFlags = newPairs;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // 访问一个不存在的数组下标会引发一个异常
            m_pairsOfFlags[index][0] = flagA;
            m_pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
            // 如果 newSize 更小，那么超出的元素会被清除
            m_pairsOfFlags.length = newSize;
        }

        function clear() public {
            // 这些代码会将数组全部清空
            delete m_pairsOfFlags;
            delete m_aLotOfIntegers;
            // 这里也是实现同样的功能
            m_pairsOfFlags.length = 0;
        }

        bytes m_byteData;

        function byteArrays(bytes data) public {
            // 字节的数组（语言意义中的 byte 的复数 ``bytes``）不一样，因为它们不是填充式存储的，
            // 但可以当作和 "uint8[]" 一样对待
            m_byteData = data;
            m_byteData.length += 7;
            m_byteData[3] = byte(8);
            delete m_byteData[2];
        }

        function addFlag(bool[2] flag) public returns (uint) {
            return m_pairsOfFlags.push(flag);
        }

        function createMemoryArray(uint size) public pure returns (bytes) {
            // 使用 `new` 创建动态 memory 数组：
            uint[2][] memory arrayOfPairs = new uint[2][](size);
            // 创建一个动态字节数组：
            bytes memory b = new bytes(200);
            for (uint i = 0; i < b.length; i++)
                b[i] = byte(i);
            return b;
        }
    }
```

### Structure

Solidity supports defining new types by constructing structures. The following is an example of how a structure is used:

```bash
    pragma solidity ^0.4.11;

    contract CrowdFunding {
        // 定义的新类型包含两个属性。
        struct Funder {
            address addr;
            uint amount;
        }

        struct Campaign {
            address beneficiary;
            uint fundingGoal;
            uint numFunders;
            uint amount;
            mapping (uint => Funder) funders;
        }

        uint numCampaigns;
        mapping (uint => Campaign) campaigns;

        function newCampaign(address beneficiary, uint goal) public returns (uint campaignID) {
            campaignID = numCampaigns++; // campaignID 作为一个变量返回
            // 创建新的结构体示例，存储在 storage 中。我们先不关注映射类型。
            campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
        }

        function contribute(uint campaignID) public payable {
            Campaign storage c = campaigns[campaignID];
            // 以给定的值初始化，创建一个新的临时 memory 结构体，
            // 并将其拷贝到 storage 中。
            // 注意你也可以使用 Funder(msg.sender, msg.value) 来初始化。
            c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
            c.amount += msg.value;
        }

        function checkGoalReached(uint campaignID) public returns (bool reached) {
            Campaign storage c = campaigns[campaignID];
            if (c.amount < c.fundingGoal)
                return false;
            uint amount = c.amount;
            c.amount = 0;
            c.beneficiary.transfer(amount);
            return true;
        }
    }
```

The above contract is just a simplified version of crowdfunding contract, but it is enough to let us understand the basic concept of structure.
Structure types can be used as elements in mappings and arrays, and can also contain mappings and arrays as member variables.

Although the structure itself can be a mapping value type member, it cannot contain itself. This restriction is necessary because the size of the structure must be limited.

Note how a structure is assigned to a local variable when a structure is used in a function (the default storage location is storage ). In this process, the structure is not copied, but a reference is saved, so the assignment of local variable members is actually written into the state.

Of course, you can also directly access the members of the structure without assigning it to a local variable, like this, ``campaigns[campaignID].amount = 0``。

#### Mapping

The mapping type is declared in the form ``mapping(_KeyType => _ValueType)``。
Of which ``_KeyType`` it can be almost all types except mappings, variable-length arrays, contracts, enumerations, and structures.
``_ValueType`` it can be any type including the mapping type.

Mapping can be treated [哈希表](https://en.wikipedia.org/wiki/Hash_table)，they create each possible key in the actual initialization process and map it to a value whose byte form is all zero: a type `默认值`。However, the following is where the mapping differs from the hash table:

In the mapping, it does not actually store the key, but stores its ``keccak256`` Hash value, so that it is easy to query the actual value.
Because of this, mapping has no length, nor the concept of a set of keys or values. Only state variables (or references to storage variables in internal functions) can use mapping types.

You can declare the mapping ``public``，and then let Solidity create a `getter `。``_KeyType`` Will become a required parameter for getter, and getter will return ``_ValueType``。``_ValueType`` It can also be a mapping. When getter is used, each  ``_KeyType`` parameters.

```bash
    pragma solidity ^0.4.0;

    contract MappingExample {
        mapping(address => uint) public balances;

        function update(uint newBalance) public {
            balances[msg.sender] = newBalance;
        }
    }

    contract MappingUser {
        function f() public returns (uint) {
            MappingExample m = new MappingExample();
            m.update(100);
            return m.balances(this);
        }
    }
```

>  Mapping does not support iteration, but such a data structure can be implemented on top of it. For example, see: [Iterable mapping](https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol)

#### Operators involving LValues

If ``a`` is an LValue (that is, a variable or other things that can be assigned values), and the following operators can be used in shorthand:

``a += e`` equivalent ``a = a + e``.  Other operators ``-=``， ``*=``， ``/=``， ``%=``， ``|=``， ``&=`` and  ``^=`` they are all defined in this way.
``a++`` and ``a--`` respectively equivalent ``a += 1`` and ``a -= 1``，But the value of the expression itself is equal `a` value before the calculation.
On the contrary, ``--a`` and ``++a`` although eventuall ``a`` result of is the same as that of the previous expression, but the return value of the expression is the value after calculation.

### Delete

``delete a`` result of is ``a`` value of the type during initialization is assigned ``a``. That is for integer variables, equivalent ``a = 0``，
However, delete is also applicable to Arrays. For dynamic arrays, the length of the array is set to 0, while for static arrays, all elements in the array are reset. If the object is a structure, all properties in the structure are reset.

``delete`` It is invalid for the entire mapping because the mapping key can be arbitrary and usually unknown. Therefore, when you delete a structure, the result will reset all non-mapping attributes. This process is recursive unless they are mapped. However, individual keys and their mapped values can be deleted.

Understand``delete a`` effect is like giving ``a`` assignment is very important, in other words, this is equivalent to in `a` new object is stored in.

```bash
    pragma solidity ^0.4.0;

    contract DeleteExample {
        uint data;
        uint[] dataArray;

        function f() public {
            uint x = data;
            delete x; // 将 x 设为 0，并不影响数据
            delete data; // 将 data 设为 0，并不影响 x，因为它仍然有个副本
            uint[] storage y = dataArray;
            delete dataArray; 
            // 将 dataArray.length 设为 0，但由于 uint[] 是一个复杂的对象，y 也将受到影响，
            // 因为它是一个存储位置是 storage 的对象的别名。
            // 另一方面："delete y" 是非法的，引用了 storage 对象的局部变量只能由已有的 storage 对象赋值。
        }
    }
```

### Conversion between Basic types

#### Implicit conversion

If an operator is used between two different types of variables, the compiler will implicitly convert one type to another (so is the assignment between different types). Generally speaking, as long as the conversion between value types is semantic and there is no information loss during the conversion process, implicit conversion is basically achievable:

``uint8``can be converted ``uint16``，``int128`` convert ``int256``，but ``int8`` cannot convert ``uint256``（because ``uint256`` Some values cannot be covered, for example,``-1``）. furthermore, an unsigned integer can be converted to a byte type of the same size or larger as it, but vice versa. Any can be converted uint160 type of can be converted address type.

#### Explicit conversion

In some cases, if the compiler does not support implicit conversion, but you know what you want to do, explicit conversion can be considered. Note that this may have some unexpected consequences, so be sure to test and make sure the results are what you want! the following example is int8 Convert a negative number of the type uint :

```bash
int8 y = -3;
uint x = uint(y);
```

At the end of this code，``x`` value of will be ``0xfffff..fd`` （64 hexadecimal characters），because this is the 256-bit complement form of -3.

If a type is explicitly converted to a smaller type, the corresponding high order will be discarded:

```bash
uint32 a = 0x12345678;
uint16 b = uint16(a); // 此时 b 的值是 0x5678
```

#### Type inference

For convenience, it is not necessary to specify the type of a variable precisely every time. The compiler automatically infers the type of the variable based on the type of the first expression that assigns the variable:

```bash
uint24 x = 0x123;
var y = x;
```
Here `y` The type of will be `uint24` . Cannot be used for function parameters or return parameters var .


> The type can only be inferred from the first assignment, so the loop in the following code is infinite because i The type of IS uint8 , and the maximum value ratio of this type of variable `2000` Small.``for (var i = 0; i < 2000; i++) { ... }``


## Unit and global variables

### SIMPLE unit

The conversion between SIMPLE units is to add after the number ``wei``、 ``finney``、 ``szabo`` or ``ether`` to implement, if there is no unit behind, the default is Wei. For example ``2 ether == 2000 finney`` logical judgment value of IS ``true``。

### Time Unit

Seconds is the default time unit, between time units, followed by numbers ``seconds``、 ``minutes``、 ``hours``、 ``days``、 ``weeks`` and ``years`` Can be converted, the basic conversion relationship is as follows:

 * ``1 == 1 seconds``
 * ``1 minutes == 60 seconds``
 * ``1 hours == 60 minutes``
 * ``1 days == 24 hours``
 * ``1 weeks == 7 days``
 * ``1 years == 365 days``

It is not 365 days a year and not 24 hours a day due to leap seconds [leap seconds](https://en.wikipedia.org/wiki/Leap_second)，so if you want to use these units to calculate the date and time, please pay attention to this problem. Because leap seconds cannot be predicted, an external prediction machine (oracle, an out-of-chain data service, noted by the translator) is required to correct the time of a certain date code base.

>``years`` Suffix is no longer recommended because it will no longer be supported from version 0.5.0.

These suffixes cannot be used directly behind variables. If you want to convert the input variable to time in a unit of time (for example, days), you can do this as follows:

```sh
    function f(uint start, uint daysAfter) public {
        if (now >= start + daysAfter * 1 days) {
            // ...
        }
    }
```

### Special variables and functions

Some special variables and functions already exist (by default) in the global namespace, which are mainly used to provide information about the block chain or some common tool functions.

.. index:: abi, block, coinbase, difficulty, encode, number, block;number, timestamp, block;timestamp, msg, data, gas, sender, value, now, gas price, origin


#### Block and transaction attributes

- ``block.blockhash(uint blockNumber) returns (bytes32)``：The block hash of the specified block-can only be used for the latest 256 blocks, excluding the current block. blocks are not recommended since version 0.4.22 ``blockhash(uint blockNumber)`` replace
- ``block.coinbase`` (``address``): dig out the miner address of the current block
- ``block.difficulty`` (``uint``): current block difficulty
- ``block.gaslimit`` (``uint``): current block gas limit
- ``block.number`` (``uint``): current block number
- ``block.timestamp`` (``uint``): the timestamp in seconds of the current block starting from the unix azone.
- ``gasleft() returns (uint256)``：the remaining gas
- ``msg.data`` (``bytes``): complete calldata
- ``msg.gas`` (``uint``): residual gas-is not recommended since version 0.4.21, `gesleft()` replace
- ``msg.sender`` (``address``): Message sender (current call)
- ``msg.sig`` (``bytes4``): cthe first 4 bytes of calldata (that is, the function identifier)
- ``msg.value`` (``uint``): The number of wei sent with the message
- ``now`` (``uint``): current block timestamp（``block.timestamp``）
- ``tx.gasprice`` (``uint``): The gas price of the transaction
- ``tx.origin`` (``address``): transaction initiator (full call chain)


>  For each `External function` call, including `msg.sender` and `msg.value` all included `msg` The value of the member changes. This includes calls to library functions.

>  Do not rely on ``block.timestamp``、 ``now`` and ``blockhash`` generate random numbers unless you know what you are doing. The timestamp and block Hash may be affected by miners to some extent. For example, malicious miners in the mining community can use a given hash to run the payout function of casino contracts, and if they do not receive money, they can also try again with a different hash. The timestamp of the current block must be strictly greater than that of the last block. However, the only thing that can be ensured here is the value between the timestamps of the two consecutive blocks on the authoritative chain.
    
>  Based on the scalability factor, the block hash is not valid for all blocks. You can only access the hashes of the last 256 blocks, and the remaining hashes are zero.

### ABI coding function

- ``abi.encode(...) returns (bytes)``：`ABI`encode the given parameters
- ``abi.encodePacked(...) returns (bytes)``：execute the given parameter`紧打包编码`
- ``abi.encodeWithSelector(bytes4 selector, ...) returns (bytes)``：`ABI` encode the given parameter and return the 4-byte data starting with the given function selector.
- ``abi.encodeWithSignature(string signature, ...) returns (bytes)``:equivalent ``abi.encodeWithSelector(bytes4(keccak256(signature), ...)``

> These encoding functions can be used to construct function call data without actually calling. In addition, ``keccak256(abi.encodePacked(a, b))`` is a more accurate method to calculate what is not recommended in future versions``keccak256(a, b)``。

For more details, see `ABI` and `紧打包编码`。


#### Error handling

``assert(bool condition)``: If the conditions are not met, the current transaction is ineffective-used to check for internal errors.
    
``require(bool condition)``: Revoke state changes if conditions are not met-used to check for errors caused by input or external components.

``require(bool condition, string message)``: Revoke state changes if conditions are not met-used to check for errors caused by input or external components, an error message can be provided at the same time.

``revert()``: Terminate the operation and cancel the status change.

``revert(string reason)``: Terminating the operation and canceling state changes can provide an explanatory string at the same time.

#### Mathematical and cryptographic functions

``addmod(uint x, uint y, uint k) returns (uint)``:calculation ``(x + y) % k``，addition will be executed at any precision, and even if the result of addition exceeds ``2**256`` it will not be intercepted. Starting from the compiler version 0.5.0 ``k != 0`` verify（assert）。

``mulmod(uint x, uint y, uint k) returns (uint)``:calculation ``(x * y) % k``，multiplication is executed at any precision, and even if the result of multiplication exceeds ``2**256`` it will not be intercepted. Starting from the compiler version 0.5.0 ``k != 0`` verify（assert）。

``keccak256(...) returns (bytes32)``: calculation :ref:`(tightly packed) arguments <abi_packed_mode>`  Ethereum-SHA-3 （Keccak-256）hash。

``sha256(...) returns (bytes32)``:calculation :ref:`(tightly packed) arguments <abi_packed_mode>`  SHA-256 hash。

``sha3(...) returns (bytes32)``:equivalent to keccak256。

``ripemd160(...) returns (bytes20)``:calculation :ref:`(tightly packed) arguments <abi_packed_mode>`  RIPEMD-160 hash。

``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)`` Use the elliptic curve signature to restore the address related to the public key. An error returns a zero value. [example usage](https://ethereum.stackexchange.com/q/1777/222)

Above `tightly packed` does not perform parameter values `padding` processing (that is, the bytecode of all parameter values is stored continuously, translator's note), which means that the following calls are equivalent:

    keccak256("ab", "c")
    keccak256("abc")
    keccak256(0x616263)
    keccak256(6382179)
    keccak256(97, 98, 99)

If padding is required, you can use explicit type conversion: ``keccak256("\x00\x12")`` and ``keccak256(uint16(0x12))`` it's the same.

Note that constant values are packaged using the minimum number of bytes required to store them. For example：``keccak256(0) == keccak256(uint8(0))``，``keccak256(0x12345678) == keccak256(uint32(0x12345678))``。

On a private chain, you are likely to encounter ``sha256``、``ripemd160`` or ``ecrecover`` caused by Out-of-Gas. The reason is that these cryptographic functions exist in the form of "precompiled contracts" in Simplechain virtual machines, and it does not really exist until the first time you receive the message (although the contract code is a hard code already existing in EVM). Therefore, messages sent to non-existent contracts are very expensive, so actual execution will lead to Out-of-Gas errors. Before you actually use your contract, send a little SIMPLE to each contract, such as 1 Wei. This is not a problem on the official network or test network.

#### Address-related

``<address>.balance`` (``uint256``):in Wei :ref:`address` balance

``<address>.transfer(uint256 amount)``:to :ref:`address` if the number of Wei sent is the amount, an exception is thrown when the error occurs. The miner fee for sending 2300 gas cannot be adjusted.

``<address>.send(uint256 amount) returns (bool)``:to :ref:`address` the number of Wei sent is amount. If the number fails, the system returns ``false``，miner's fee for sending 2300 gas cannot be adjusted.

``<address>.call(...) returns (bool)``:issue low-level functions ``CALL``，if it fails, return ``false``，send all available gas, adjustable.

``<address>.callcode(...) returns (bool)``：issue low-level functions ``CALLCODE``，if it fails, return `false` , send all available gas, adjustable.

``<address>.delegatecall(...) returns (bool)``:issue low-level functions ``DELEGATECALL``，if it fails, return ``false``，send all available gas,adjustable.

> There are many dangers when using send: if the call stack depth has reached 1024 (which can always be forcibly specified by the caller), the transfer will fail; And if the receiver uses up the gas, the transfer will also fail. In order to ensure the security of Ethernet currency transfer, always check `send` The return value of, using `transfer` Or the following is a better way: use this mode of receiving money back.

>  If you need to access the variables in the storage when you use the low-level function delegatecall to initiate a call, the variables in the storage of the two contracts must be defined in the same order, so that the called contract code can correctly access the contract's storage variables through the variable name. Of course, this does not refer to the situation like the stored variable pointer passed when an advanced library function is called.

>  Use is not encouraged `callcode` , and it will be removed in the future.

Contract-related


``this`` (current contract's type):current contract, which can be explicitly converted to `address`。

``selfdestruct(address recipient)``:destroy the contract and send the balance to the specified `address`。

``suicide(address recipient)``:equivalent to selfdestruct, but not recommended.

In addition, all functions in the current contract can be called directly, including the current function.


## Expression and control structure

### Input and output parameters

Like Javascript, functions may require parameters as input; Unlike Javascript and C, they may return any number of parameters as output.

#### Input parameters

Input parameters are declared in the same way as variables. However, one exception is that unused parameters can omit parameter names.
For example, if we want the contract to accept external calls to functions with two integer parameters, we will write as follows

```bash
    pragma solidity ^0.4.16;

    contract Simple {
        function taker(uint _a, uint _b) public pure {
            // 用 _a 和 _b 实现相关功能.
        }
    }
```

#### Output parameters

The declaration method of the output parameter is in the keyword returns After that, the declaration method is the same as that of the input parameters.
For example, if we need to return two results: the sum and product of two given integers, we should write

```bash
    pragma solidity ^0.4.16;

    contract Simple {
        function arithmetics(uint _a, uint _b)
            public
            pure
            returns (uint o_sum, uint o_product)
        {
            o_sum = _a + _b;
            o_product = _a * _b;
        }
    }
```

The output parameter name can be omitted. The output value can also be used `return` Statement specifies.
return Statement can also return multiple values, The returned output parameters are initialized to 0; If they are not explicitly assigned, they are always 0.

Input and output parameters can be used as expressions in function bodies. Therefore, they can also be assigned to the left of the equal sign.

### Control Structure

Most control structures in JavaScript are available in Solidity, `switch` And `goto` . Therefore, there are ``if``，``else``，``while``，``do``，``for``，``break``，``continue``，``return``，``? :`` these keywords express the same semantics as in C or JavaScript.

Brackets used to represent conditions No. If it is omitted, the braces on both sides of the single statement body can be omitted. Note that unlike C and JavaScript, non-Boolean values in Solidity cannot be converted to Boolean types, so ``if (1) { ... }`` write in Solidity in Invalid .

### Returns multiple values

When a function has multiple output parameters, ``return (v0, v1, ...,vn)`` you can return multiple values. However, the number of elements must be the same as the number of output parameters.

### Function call

#### Internal function call

The functions in the current contract can be called directly ("from inside") or recursively, just like the ridiculous example below.

```bash

    pragma solidity ^0.4.16;

    contract C {
        function g(uint a) public pure returns (uint ret) { return f(); }
        function f() internal pure returns (uint ret) { return g(7) + f(); }
    }
```

These function calls are interpreted as simple redirection in EVM. The effect of this is that the current memory will not be cleared, that is, passing memory references between functions through internal calls is very effective.

#### External function call

Expression ``this.g(8);`` and ``c.g(2);`` （among them ``c`` is a contract instance）is also a valid function call, but in this case, the function will be "called externally" through a message call, rather than directly jump. Note that this function cannot be called in the constructor because the real contract instance has not been created yet.

If you want to call functions of other contracts, you need to call them externally. For an external call, all function parameters need to be copied to memory.

When calling functions of other contracts, the number of Wei and gas sent along with the function call can be determined by specific options respectively .value() And .gas() Specify:

```bash
    pragma solidity ^0.4.0;

    contract InfoFeed {
        function info() public payable returns (uint ret) { return 42; }
    }

    contract Consumer {
        InfoFeed feed;
        function setFeed(address addr) public { feed = InfoFeed(addr); }
        function callFeed() public { feed.info.value(10).gas(800)(); }
    }
```

``payable`` modifiers should be used for Modifiers ``info``，otherwise，`.value()` options will not be available.

Note, expression ``InfoFeed(addr)`` an explicit type conversion was performed, indicating that "we know that the contract type for a given address is ``InfoFeed`` “and this will not execute the constructor. Explicit type conversion requires caution. Never execute a function call on a contract that you do not know the type.

We can also use it directly ``function setFeed(InfoFeed _feed) { feed = _feed; }`` 。
pay attention to a fact，``feed.info.value(10).gas(800)`` only (partially) the number of Wei values and gas values sent together with the function call is set, and only the final parentheses execute the real call. if the contract where the function is called does not exist (that is, the account does not contain code) or the called contract itself throws an exception or gas runs out, the function call throws an exception.


>	Any interaction with other contracts will impose potential dangers, especially when the contract code cannot be known in advance.
The current contract transfers control to the invoked contract, and the invoked contract may do anything. Even if the called contract is inherited from a known parent contract, the inherited contract only needs to have a correct interface.
The implementation of the called contract can be completely arbitrary, thus bringing danger. In addition, be careful in case it calls other contracts in your system again, or even returns your call contract before the first call returns.
This means that the called contract can change the state variables of the called contract through its own functions.. A suggested function writing method is, for example, calling external functions after various changes have been made to the state variables in your contract, so that your contract will not be easily abused reentrancy (reentrancy) affected


### Named calls and anonymous function parameters


If they are included in `{}` Function call parameters can also be given by name in any order,
As shown in the following example. The parameter list must match the parameter list in the function declaration by name, but can be arranged in any order.

```bash
    pragma solidity ^0.4.0;

    contract C {
        function f(uint key, uint value) public {
            // ...
        }

        function g() public {
            // 具名参数
            f({value: 2, key: 3});
        }
    }
```

### Omit the function parameter name

Names of unused parameters (especially returned parameters) can be omitted. These parameters still exist in the stack, but they cannot be accessed.

```bash
    pragma solidity ^0.4.16;

    contract C {
        // 省略参数名称
        function func(uint k, uint) public pure returns(uint) {
            return k;
        }
    }
```

### 通Pass `new` Create a contract

Use keywords new You can create a new contract. The complete code of the contract to be created must be known in advance, so recursive dependency creation is impossible.

```bash
    pragma solidity ^0.4.0;

    contract D {
        uint x;
        function D(uint a) public payable {
            x = a;
        }
    }

    contract C {
        D d = new D(4); // 将作为合约 C 构造函数的一部分执行

        function createD(uint arg) public {
            D newD = new D(arg);
        }

        function createAndEndowD(uint arg, uint amount) public payable {
		    //随合约的创建发送 ether
            D newD = (new D).value(amount)(arg);
        }
    }
```

如As shown in the example, use `.value（）` Option creation `D` The Ether can be forwarded to the instance, but it is impossible to limit the amount of gas. If the creation fails (possibly because of Stack Overflow, or insufficient balance or other problems), an exception is thrown.

### Expression Calculation order

The order of expression calculation is not specific (more precisely, the order of calculation between byte points of a node in the expression tree is not specific, but their settlement will certainly be before the node's own settlement). This rule can only ensure that the statements are executed in sequence and the short circuit of Boolean expressions is executed. For more information。

### Assignment

#### Deconstruct assignment and return multiple values

Solidity internally allows tuple class，which is a list of objects with a fixed number of elements at compile time. The elements in the list can be different types of objects. These tuples can be used to return multiple values at the same time, or they can be used to simultaneously give multiple newly declared variables or existing variables (or common LValues):

```bash

    pragma solidity >0.4.23 <0.5.0;

    contract C {
        uint[] data;

        function f() public pure returns (uint, bool, uint) {
            return (7, true, 2);
        }

        function g() public {
            //基于返回的元组来声明变量并赋值
            (uint x, bool b, uint y) = f();
            //交换两个值的通用窍门——但不适用于非值类型的存储 (storage) 变量。
            (x, y) = (y, x);
            //元组的末尾元素可以省略（这也适用于变量声明）。
            (data.length,,) = f(); // 将长度设置为 7
            //省略元组中末尾元素的写法，仅可以在赋值操作的左侧使用，除了这个例外：
            (x,) = (1,);
            //(1,) 是指定单元素元组的唯一方法，因为 (1)
            //相当于 1。
        }
    }
```

> Until version 0.4.24, it is possible to assign values to tuples with fewer elements, whether on the left or on the right (for example, several elements are left at the end). Now, this is not recommended. Both sides of the assignment operation should have the same number of constituent elements.

### Complexity of arrays and structures

Assignment semantics is somewhat complicated for non-value types such as arrays and structures.
Is a state variable Assignment A standalone copy is often created. On the other hand, the assignment of local variables only creates an independent copy for the basic type (that is, the static type within 32 bytes). If a structure or array (including `bytes` And `string` ) is assigned from the state variable to the local variable, the local variable will retain the reference to the original state variable. The second assignment to a local variable does not modify the state variable, but only changes the reference. If a member or element is assigned to a local variable Change State variables.

### Scope and declaration

A variable is declared with a default initial value, whose initial value bytes indicate all zero. The default value of any type variable is the typical zero state of its corresponding type ". For example, ``bool`` default value of the type is ``false`` . ``uint`` or ``int`` default value of the type is ``0``. For static size arrays and ``bytes1``  to ``bytes32`` ，each individual element will be initialized to the default value corresponding to its type.
Finally, for an array of dynamic size. ``bytes`` and ``string`` type，default default value is an empty array or string.

Scope rules in Solidity follow C99 (like many other languages): variables will be visible after they are declared until a pair `{ }` The end of the block. As an exception, the visibility of variables initialized in the for loop statement is maintained only until the end of the for loop.

Variables defined outside code blocks, such as functions, contracts, and custom types, do not affect their scope properties. This means that you can use state variables before actually declaring statements and call functions recursively.

Based on the above rules, compilation warnings will not appear in the following example, because the two variables have the same name but are in different scopes.

```bash

    pragma solidity >0.4.24;
    contract C {
        function minimalScoping() pure public {
            {
                uint same2 = 0;
            }

            {
                uint same2 = 0;
            }
        }
    }
```

As a special case of a C99 scope rule, note that in the following example, for the first time `x` assignment of changes the variable values declared in the previous layer. If the variables declared outside are "shadowed" (that is, replaced by a variable with the same name in the internal scope), you will receive a warning.

```bash
    pragma solidity >0.4.24;
    contract C {
        function f() pure public returns (uint) {
            uint x = 1;
            {
                x = 2; // 这个赋值会影响在外层声明的变量
                uint x;
            }
            return x; // x has value 2
        }
    }
```


>  Earlier than Solidity 0.5.0, Javascript rules were used for scope rules. That is, a variable can be declared anywhere in the function and can be visible throughout the function. This rule will be broken from version 0.5.0. Starting with version 0.5.0, the code segment in the following example will cause compilation errors.


```bash

    // 这将无法编译通过

    pragma solidity >0.4.24;
    contract C {
        function f() pure public returns (uint) {
            x = 2;
            uint x;
            return x;
        }
    }
```

### Error handling：Assert, Require, Revert and Exceptions

Solidity uses status recovery exceptions to handle errors. This exception cancels all changes made to the status of the current call and all its sub-calls and marks the caller with an error. Convenience function ``assert`` and ``require`` can be used to check conditions and throw exceptions when conditions are not met. ``assert`` functions can only be used to test internal errors and check non-variables.

``require`` function is used to confirm condition validity, such as input variable, or contract status variable meets the condition, or to verify the value returned by external contract call. If used properly, the analysis tool can evaluate your contract and mark those that will `assert` Failed conditions and function calls. Normal code does not cause an assert statement to fail. If this happens, a bug that you need to fix appears.

There are two other ways to trigger an exception: ``revert`` functions can be used to mark errors and restore the current call.
``revert`` It is possible that the call contains detailed information about the error, and this message is returned to the caller. Keywords that are not recommended ``throw`` can also be used to replace ``revert()`` （but unable to return an error message.）.

> Starting from version `0.4.13`, throw this keyword has been abandoned and will be gradually eliminated in the future.

When a sub-call exception occurs, they automatically "bubble" (that is, throw an exception again). The exception to this rule is ``send`` and low-level functions ``call`` ， ``delegatecall`` and ``callcode`` -- If these functions are abnormal, false is returned instead of bubbling ".

>  As part of the EVM design, if the called contract account does not exist, the low-level function `call` , `delegatecall` And `callcode` success is returned. Therefore, if you need to use low-level functions, you must check whether the called contract exists before calling. Exception capture has not been implemented

In the following example, you can see how to use it easily `require` Check input conditions and how to use them `assert` Check for internal errors. Note that you can give `require` Provides a message string, and `assert` No.

```bash

    pragma solidity ^0.4.22;

    contract Sharer {
        function sendHalf(address addr) public payable returns (uint balance) {
            require(msg.value % 2 == 0, "Even value required.");
            uint balanceBeforeTransfer = this.balance;
            addr.transfer(msg.value / 2);
			//由于转移函数在失败时抛出异常并且不能在这里回调，因此我们应该没有办法仍然有一半的钱。
            assert(this.balance == balanceBeforeTransfer - msg.value / 2);
            return this.balance;
        }
    }
```

One of the following situations will be generated `assert` Type exception:

- If the index of the array you access is too large or negative（for example ``x[i]`` of which ``i >= x.length`` or ``i < 0``）。
- If you access a fixed length ``bytesN`` index of is too large or negative.
- If you use zero as a divisor for division or modulo（for example ``5 / 0`` or ``23 % 0`` ）。
- If you shift negative digits.
- If you convert a too large or negative value to an enumeration type.
- If you call a zero-initialization variable of the internal function type.
- If you call `assert` final settlement of the parameter (expression) is false.

One of the following situations will be generated `require` type exception:

- call ``throw`` 。
- If you call ``require`` final settlement of the parameter (expression) is ``false`` 。
- If you call a function through a message, but the function does not end correctly (it runs out of gas, does not match the function, or throws an exception itself), the above function does not include low-level operations. ``call`` ， ``send`` ， ``delegatecall`` or ``callcode`` . A low-level operation does not throw an exception but returns false To indicate `failure`.
- If you use `new` Keyword creates a contract, but the contract was not created correctly (see the definition of "not completed correctly" in the above article).
- If you execute an external function call on a contract that does not contain code.
- If your contract passes one payable Public functions of modifiers (including constructors and fallback functions) receive Ether.
- If your contract receives Ether through the public getter function.
- If .transfer() Failed.


Internally, Solidity for a ``require`` abnormal execution rollback operation (instruction `0xfd` ) and execute an invalid operation (instruction `0xfe` ) to trigger assert Abnormal formula. In both cases, EVM will roll back all changes made to the state. The reason for the rollback is that it cannot continue to be safely executed because the expected effect has not been achieved. Because we want to retain the atomicity of the transaction, the safest way is to roll back all changes and make the entire transaction (or at least called) ineffective. Please note, assert The type exception consumes all available call gas, and from the Metropolis version require The abnormality of the formula will not consume any gas.

The following example shows how to use error strings in Invocation and require:

```bash

    pragma solidity ^0.4.22;

    contract VendingMachine {
        function buy(uint amount) payable {
            if (amount > msg.value / 2 ether)
                revert("Not enough Ether provided.");
            // 下边是等价的方法来做同样的检查：
            require(
                amount <= msg.value / 2 ether,
                "Not enough Ether provided."
            );
            // 执行购买操作
        }
    }
```

The string provided here should go through:`ABI 编码 ` later, because it is actually called ``Error(string)`` function. In the above example,``revert("Not enough Ether provided.");`` following hexadecimal error return value is generated:

- 0x08c379a0                                                         // Error(string) function selector
- 0x0000000000000000000000000000000000000000000000000000000000000020 // data offset（32）
- 0x000000000000000000000000000000000000000000000000000000000000001a // string length（26）
- 0x4e6f7420656e6f7567682045746865722070726f76696465642e000000000000 // string data（ASCII encoding of "Not enough Ether provided.", 26 bytes）


## Contract

`Solidity` Contracts are similar to classes in object-oriented languages. The contract contains state variables for data persistence and functions that can modify state variables. When a function of another contract instance is called, an EVM function is called. This operation switches the context of the execution, so that the status variable of the previous contract cannot be accessed.

### Create a contract

You can create contracts "from outside" through Simplechain transactions or from inside Solidity contracts.

Some integrated development environments, such [Remix](https://remix.ethereum.org/), through the use of some user interface elements to make the creation process more smooth. You 'd better use JavaScript API to create a contract by programming on Simplechain [web3.j](https://github.com/ethereum/web3.js). Now, we already have one called [web3.eth.Contract](https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract) method can make it easier to create a contract.

When a contract is created, the constructor (a function with the same name as the contract) is executed once. The constructor is optional. Only one constructor is allowed, which means overloading is not supported.

Internally, the constructor parameters are passed after the contract code `ABI code` pass, but if you use``web3.js``there is no need to care about this problem.

If a contract wants to create another contract, the creator must know the source code (and binary code) of the contract to be created.
This means that it is impossible to create dependencies cyclically.

```bash
    pragma solidity ^0.4.16;

    contract OwnedToken {
        // TokenCreator 是如下定义的合约类型.
        // 不创建新合约的话，也可以引用它。
        TokenCreator creator;
        address owner;
        bytes32 name;

        // 这是注册 creator 和设置名称的构造函数。
        function OwnedToken(bytes32 _name) public {
            // 状态变量通过其名称访问，而不是通过例如 this.owner 的方式访问。
            // 这也适用于函数，特别是在构造函数中，你只能像这样（“内部地”）调用它们，
            // 因为合约本身还不存在。
            owner = msg.sender;
            // 从 `address` 到 `TokenCreator` ，是做显式的类型转换
            // 并且假定调用合约的类型是 TokenCreator，没有真正的方法来检查这一点。
            creator = TokenCreator(msg.sender);
            name = _name;
        }

        function changeName(bytes32 newName) public {
            // 只有 creator （即创建当前合约的合约）能够更改名称 —— 因为合约是隐式转换为地址的，
            // 所以这里的比较是可行的。
            if (msg.sender == address(creator))
                name = newName;
        }

        function transfer(address newOwner) public {
            // 只有当前所有者才能发送 token。
            if (msg.sender != owner) return;
            // 我们也想询问 creator 是否可以发送。
            // 请注意，这里调用了一个下面定义的合约中的函数。
            // 如果调用失败（比如，由于 gas 不足），会立即停止执行。
            if (creator.isTokenTransferOK(owner, newOwner))
                owner = newOwner;
        }
    }

    contract TokenCreator {
        function createToken(bytes32 name)
           public
           returns (OwnedToken tokenAddress)
        {
            // 创建一个新的 Token 合约并且返回它的地址。
            // 从 JavaScript 方面来说，返回类型是简单的 `address` 类型，因为
            // 这是在 ABI 中可用的最接近的类型。
            return new OwnedToken(name);
        }

        function changeName(OwnedToken tokenAddress, bytes32 name)  public {
            // 同样，`tokenAddress` 的外部类型也是 `address` 。
            tokenAddress.changeName(name);
        }

        function isTokenTransferOK(address currentOwner, address newOwner)
            public
            view
            returns (bool ok)
        {
            // 检查一些任意的情况。
            address tokenAddress = msg.sender;
            return (keccak256(newOwner) & 0xff) == (bytes20(tokenAddress) & 0xff);
        }
    }
```

### Visibility and getter functions

Because Solidity has two kinds of function calls (internal calls do not generate actual EVM calls or "message calls", while external calls generate an EVM call), functions and state variables have four visibility types. Function can be specified ``external`` ，``public`` ，``internal`` or ``private``，by default , function type is ``public``。
For state variables, cannot be set ``external`` ，default is ``internal`` 。

- ``external`` ：As part of the contract interface, the external function means that we can call it from other contracts and transactions.
    An external function ``f`` cannot be called from inside (that is f It doesn't work, this.f() Yes). When a large amount of data is received, external functions are sometimes more efficient.
- ``public`` ：The public function is part of the contract interface and can be called internally or through messages. For common state variables, a getter function is automatically generated (see below).
- ``internal`` ：these functions and state variables can only be internal access (I .e. access from inside the current contract or from contracts derived from it), and are not used `this` Call.
``private`` ：private functions and state variables are only used in contracts that currently define them and cannot be used by derived contracts.

> All content in the contract is visible to external observers. Set some `private` The type can only prevent other contracts from accessing and modifying this information, but it is still visible for the whole world outside the blockchain.

The definition position of the visibility identifier. For a state variable, it is after the type. For a function, it is between the parameter list and the returned keyword.

```bash
pragma solidity ^0.4.16;

contract C {
    function f(uint a) private pure returns (uint b) { return a + 1; }
    function setData(uint a) internal { data = a; }
    uint public data;
}
```

In the following example, `D` Can be called `c.getData（）` to obtain status storage data But cannot call `f` .
Contract `E` Inherited from `C` , so it can be called compute .

```bash
    // 下面代码编译错误

    pragma solidity ^0.4.0;

    contract C {
        uint private data;

        function f(uint a) private returns(uint b) { return a + 1; }
        function setData(uint a) public { data = a; }
        function getData() public returns(uint) { return data; }
        function compute(uint a, uint b) internal returns (uint) { return a+b; }
    }

    contract D {
        function readData() public {
            C c = new C();
            uint local = c.f(7); // 错误：成员 `f` 不可见
            c.setData(3);
            local = c.getData();
            local = c.compute(3, 5); // 错误：成员 `compute` 不可见
        }
    }

    contract E is C {
        function g() public {
            C c = new C();
            uint val = compute(3, 5); // 访问内部成员（从继承合约访问父合约成员）
        }
    }
```

#### Getter function

The compiler automatically for all Public State variables create getter functions. For the contract given below, the compiler generates a contract named data The function, This function does not receive any parameters and returns a uint , that is, the state variable data The value. You can complete the initialization of the state variable at the time of declaration.

```bash
    pragma solidity ^0.4.0;

    contract C {
        uint public data = 42;
    }

    contract Caller {
        C c = new C();
        function f() public {
            uint local = c.data();
        }
    }
```

The getter function has external visibility. If getter is accessed internally (that is, none this. ), it is considered a state variable.
If it is externally accessed (that is, use this. ), it is considered as a function.

```bash

    pragma solidity ^0.4.0;

    contract C {
        uint public data;
        function x() public {
            data = 3; // 内部访问
            uint val = this.data(); // 外部访问
        }
    }
```

The next example is slightly more complicated:

```bash

    pragma solidity ^0.4.0;

    contract Complex {
        struct Data {
            uint a;
            bytes3 b;
            mapping (uint => uint) map;
        }
        mapping (uint => mapping(bool => Data[])) public data;
    }
```

This generates a function in the following form:

```bash
function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
    a = data[arg1][arg2][arg3].a;
    b = data[arg1][arg2][arg3].b;
}
```

Note that because there is no good method to provide the keys for mapping, the mapping in the structure is omitted.

#### Function |modifier|

Use | modifier | To easily change the behavior of the function. For example, they can automatically check a condition before executing a function.
| modifier | Is an inheritable property of the contract,
And may be overwritten by derived contracts.

```bash

    pragma solidity ^0.4.11;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;

        // 这个合约只定义一个修饰器，但并未使用： 它将会在派生合约中用到。
        // 修饰器所修饰的函数体会被插入到特殊符号 _; 的位置。
        // 这意味着如果是 owner 调用这个函数，则函数会被执行，否则会抛出异常。
        modifier onlyOwner {
            require(msg.sender == owner);
            _;
        }
    }

    contract mortal is owned {
        // 这个合约从 `owned` 继承了 `onlyOwner` 修饰符，并将其应用于 `close` 函数，
        // 只有在合约里保存的 owner 调用 `close` 函数，才会生效。
        function close() public onlyOwner {
            selfdestruct(owner);
        }
    }

    contract priced {
        // 修改器可以接收参数：
        modifier costs(uint price) {
            if (msg.value >= price) {
                _;
            }
        }
    }

    contract Register is priced, owned {
        mapping (address => bool) registeredAddresses;
        uint price;

        function Register(uint initialPrice) public { price = initialPrice; }

        // 在这里也使用关键字 `payable` 非常重要，否则函数会自动拒绝所有发送给它的以太币。
        function register() public payable costs(price) {
            registeredAddresses[msg.sender] = true;
        }

        function changePrice(uint _price) public onlyOwner {
            price = _price;
        }
    }

    contract Mutex {
        bool locked;
        modifier noReentrancy() {
            require(!locked);
            locked = true;
            _;
            locked = false;
        }

        // 这个函数受互斥量保护，这意味着 `msg.sender.call` 中的重入调用不能再次调用  `f`。
        // `return 7` 语句指定返回值为 7，但修改器中的语句 `locked = false` 仍会执行。
        function f() public noReentrancy returns (uint) {
            require(msg.sender.call());
            return 7;
        }
    }
```

If the same function has multiple `modifier` , they are separated by spaces, `modifier` Checks the execution in sequence.

> In earlier versions of Solidity, there were functions of | modifier |, `return` behavior of the statement is different.

`modifier` or the explicit return statement in the function body only jumps out of the current `modifier` and fucntion bodies. returned variable is assigned a value, but the entire execution logic continues after the "_" defined in the previous | modifier |.

`modifier`parameter of can be any expression, in this context, all symbols visible in the function，in `modifier` all visible. in `modifier` symbols introduced in are invisible in the function (may be overloaded).

### Constant state variable

Status variables can be declared `constant` . In this case, only expressions that determine values at compile time can be used to assign values to them. Any blockchain data (such `now` , `this.balance` Or `block.number` ) or execution data ( `msg.gas` ) or calls to external contracts to assign values to them are not allowed. There is a boundary effect on memory allocation ( `side-effect` ) expressions are allowed, but expressions that produce boundary effects on other memory objects are not allowed. built-in function `keccak256` , `sha256` , `ripemd160` , `ecrecover` , `addmod` and `mulmod` is allowed (even if they do invoke external contracts).

The reason why memory allocators with boundary effects are allowed is that this will allow the construction of complex objects, such as lookup-table.
This feature is not fully available. The compiler does not reserve storage for these variables, and each time they appear, they are replaced with the corresponding constant expression (which may be calculated by the optimizer as an actual value). Not all types of state variables support constant modification. Currently, only value types and strings are supported.

```bash
    pragma solidity ^0.4.0;

    contract C {
        uint constant x = 32**22 + 8;
        string constant text = "abc";
        bytes32 constant myHash = keccak256("abc");
    }
```

### Function

#### View function

You can declare a function `view` type, in this case, make sure that the state is not modified.

The following statement is considered to modify the state:

- Modify the status variable.
- `generate an event`。
- `create smart contract`。
- use `selfdestruct`。
- send SIMPLE coins by calling。
- Call any that is not marked `view` Or `pure` The function.
- Use low-level calls.
- Use an inline assembly that contains specific opcodes.

```bash
    pragma solidity ^0.4.16;

    contract C {
        function f(uint a, uint b) public view returns (uint) {
            return a * (b + 42) + now;
        }
    }
```

>  ``constant`` is ``view`` alias。

>  The Getter method is marked ``view``。

>  The compiler does not force ``view`` method cannot modify the status.

#### Pure function

Functions can be declared `pure` , in this case, promise not to read or modify the status. In addition to the list of state modification statements explained above, the following is considered to be read from the state:

- Read status variables.
- Access ``this.balance`` or ``<address>.balance``。
- Access ``block``，``tx``， ``msg`` any member in （beside ``msg.sig`` and ``msg.data`` outside).
- Call any not marked `pure` function.
- Use an inline assembly that contains certain opcodes.

```bash
    pragma solidity ^0.4.16;

    contract C {
        function f(uint a, uint b) public pure returns (uint) {
            return a * (b + 42);
        }
    }
```

>  The compiler does not force pure method cannot read the status.

#### Fallback function

A contract can have an unnamed function. This function cannot have parameters or return values. If no other function matches the given function identifier (or no call data is provided) in a contract call, the function (fallback function) will be executed.

In addition, this function will be executed every time the contract receives SIMPLE coins (without any data). In addition, the fallback function must be marked payable . If such a function does not exist, the contract cannot receive SIMPLE coins through regular transactions.

In this context, usually only a few gas can be used to complete this function call (to be exact, 2300 gas), so it is important to make the call of fallback function as cheap as possible. Note that the gas required for transactions calling the fallback function (rather than internal calls) is much higher, because an additional 21000 gas or more is charged for each transaction for signature check and other operations.

Specifically, the following operations consume more gas than the fallback function:

- Write storage
- Create a contract
- Call external functions that consume a lot of gas
- Send SIMPLE coins

Make sure that you thoroughly test your fallback function before deploying the contract to ensure that the execution cost is less than 2300 gas.

>  Even if the fallback function cannot have parameters, it can still be used `msg.data` to obtain any valid data provided with the call.

>  A contract that does not define a fallback function directly receives ether coins (no function call, that is, use `send` Or `transfer` ) throws an exception and returns the ether coin (the behavior will be different before Solidity v0.4.0). Therefore, if you want your contract to receive Ether coins, you must implement the fallback function.

>  A contract without the payable fallback function can be used coinbase transaction (Also known miner block reward ) the recipient or selfdestruct The target to receive SIMPLE coins.

>  A contract cannot respond to this ether transfer, so it cannot refuse them either. This is determined by EVM when designing, and Solidity cannot bypass this problem.

>  This also means `this.balance` can be higher than the sum of some manual accounting implemented in the contract (that is, the accumulator updated in the fallback function).

```bash

    pragma solidity ^0.4.0;

    contract Test {
        // 发送到这个合约的所有消息都会调用此函数（因为该合约没有其它函数）。
        // 向这个合约发送以太币会导致异常，因为 fallback 函数没有 `payable` 修饰符
        function() public { x = 1; }
        uint x;
    }
    // 这个合约会保留所有发送给它的以太币，没有办法返还。
    contract Sink {
        function() public payable { }
    }
    contract Caller {
        function callTest(Test test) public {
            test.call(0xabcdef01); // 不存在的哈希
            // 导致 test.x 变成 == 1。
            // 以下将不会编译，但如果有人向该合约发送以太币，交易将失败并拒绝以太币。
            // test.send(2 ether）;
        }
    }
```

#### Function overload

A contract can have functions with the same name with multiple different parameters. This also applies to inheritance functions. The following example shows the contract `A` Overloaded functions in `f` .

```bash

    pragma solidity ^0.4.16;

    contract A {
        function f(uint _in) public pure returns (uint out) {
            out = 1;
        }

        function f(uint _in, bytes32 _key) public pure returns (uint out) {
            out = 2;
        }
    }

The above two `f` Function overloads accept ABI address types, although they are considered different in Solidity.

```bash
    // 以下代码无法编译
    pragma solidity ^0.4.16;

    contract A {
        function f(B _in) public pure returns (B out) {
            out = _in;
        }

        function f(address _in) public pure returns (address out) {
            out = _in;
        }
    }

    contract B {
    }
```

The above two `f` Function overloads accept ABI address types, although they are considered different in Solidity.

#### Overloaded parsing and parameter matching

You can choose to reload the function by matching the function declarations in the current range with the parameters provided in the function call.
If all parameters can be implicitly converted to the expected type, the function is selected as the overload candidate. If none of the candidates exist, the parsing fails.

>  The returned parameter is not used as the basis for overload resolution.

```bash
    pragma solidity ^0.4.16;

    contract A {
        function f(uint8 _in) public pure returns (uint8 out) {
            out = _in;
        }

        function f(uint256 _in) public pure returns (uint256 out) {
            out = _in;
        }
    }
```

Call ``f(50)`` causes type errors because ``50`` can be implicitly converted ``uint8`` can also be implicitly converted ``uint256`` on the other hand, call``f(256)`` parses ``f(uint256)`` overload，because ``256`` cannot be implicitly converted ``uint8``。

### Event

Events allow us to easily use the log infrastructure of EVM. We can listen to events in the user interface of dapp, and the log mechanism of EVM can in turn "call" the Javascript callback function used to listen to events.

Events can be inherited in the contract. When they are called, parameters are stored in transaction logs-a special data structure in the blockchain. These logs are associated with the address and incorporated into the blockchain. They exist as long as the block is accessible (they are permanently saved in Frontier and Homestead versions and may be changed in Serenity versions). Logs and events cannot be directly accessed within the contract (even the contract for creating logs cannot be accessed).

The Simplified Payment Verification of logs is possible. If an external entity provides a contract with this proof, it can check whether the logs actually exist in the blockchain. However, it should be noted that only the latest 256 block hashes can be accessed in the contract, so the block header information needs to be provided.

A maximum of three parameters can be received `indexed` Property so that they can be searched: specific values of indexed parameters can be used for filtering on the user interface.

If the array (including `string` And `bytes` ) type is marked as index item, their keccak-256 hash values are saved as topic. Unless you use anonymous The descriptor declares an event. Otherwise, the hash value of the event signature is one of the topics. It also means that `anonymous` events cannot be filtered by names. All non-index parameters are stored in the data section of the log.


>  The index parameters themselves are not saved. You can only search for their values (to determine whether the corresponding log data exists), not their values themselves.

```bash

    pragma solidity ^0.4.0;

    contract ClientReceipt {
        event Deposit(
            address indexed _from,
            bytes32 indexed _id,
            uint _value
        );

        function deposit(bytes32 _id) public payable {
            // 我们可以过滤对 `Deposit` 的调用，从而用 Javascript API 来查明对这个函数的任何调用（甚至是深度嵌套调用）。
            Deposit(msg.sender, _id, msg.value);
        }
    }
```

Use JavaScript APIs to call events as follows:

```bash

    var abi = /* abi 由编译器产生 */;
    var ClientReceipt = web3.eth.contract(abi);
    var clientReceipt = ClientReceipt.at("0x1234...ab67" /* 地址 */);

    var event = clientReceipt.Deposit();

    // 监视变化
    event.watch(function(error, result){
        // 结果包括对 `Deposit` 的调用参数在内的各种信息。
        if (!error)
            console.log(result);
    });

    // 或者通过回调立即开始观察
    var event = clientReceipt.Deposit(function(error, result) {
        if (!error)
            console.log(result);
    });
```
#### The underlying interface of the log

through function ``log0``，``log1``， ``log2``， ``log3`` and ``log4`` you can access the underlying interface of the log mechanism. ``logi``  accept ``i + 1`` a ``bytes32`` parameter of the type. The first parameter is used as the data part of the log, and the others are used as the topic. The preceding event calls can be executed in the same way.

```bash
    pragma solidity ^0.4.10;

    contract C {
        function f() public payable {
            bytes32 _id = 0x420042;
            log3(
                bytes32(msg.value),
                bytes32(0x50cb9fe53daa9737b786ab3646f04d0150dc50ef4e75f59509d83667ad5adb20),
                bytes32(msg.sender),
                _id
            );
        }
    }
```

The calculation method of the long hexadecimal number is ``keccak256("Deposit(address,hash256,uint256)")``，that is, the signature of the event.

#### Resources for other learning event mechanisms
 
- [Javascript documentation](https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events)
- [event usage routine](https://github.com/debris/smart-exchange/blob/master/lib/contracts/SmartExchange.sol)
- [How to access them in js](https://github.com/debris/smart-exchange/blob/master/lib/exchange_transactions.js)

### Inheritance

Solidity supports multiple inheritance by copying code that includes polymorphism. All function calls are virtual, which means that the farthest derived function will be called unless the contract name is explicitly given. When a contract is inherited from multiple contracts, only one contract is created on the blockchain, and the code of all base-class contracts is copied to the created contract.

In general, Solidity's inheritance system and [Python inherutance system](https://docs.python.org/3/tutorial/classes.html#inheritance)，very similar, especially in terms of multiple inheritance.

The following example is described in detail.

```bash
    pragma solidity ^0.4.16;

    contract owned {
        function owned() { owner = msg.sender; }
        address owner;
    }

    // 使用 is 从另一个合约派生。派生合约可以访问所有非私有成员，包括内部函数和状态变量，
    // 但无法通过 this 来外部访问。
    contract mortal is owned {
        function kill() {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    // 这些抽象合约仅用于给编译器提供接口。
    // 注意函数没有函数体。
    // 如果一个合约没有实现所有函数，则只能用作接口。
    contract Config {
        function lookup(uint id) public returns (address adr);
    }

    contract NameReg {
        function register(bytes32 name) public;
        function unregister() public;
     }

    // 可以多重继承。请注意，owned 也是 mortal 的基类，
    // 但只有一个 owned 实例（就像 C++ 中的虚拟继承）。
    contract named is owned, mortal {
        function named(bytes32 name) {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).register(name);
        }

        // 函数可以被另一个具有相同名称和相同数量/类型输入的函数重载。
        // 如果重载函数有不同类型的输出参数，会导致错误。
        // 本地和基于消息的函数调用都会考虑这些重载。
        function kill() public {
            if (msg.sender == owner) {
                Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
                NameReg(config.lookup(1)).unregister();
                // 仍然可以调用特定的重载函数。
                mortal.kill();
            }
        }
    }

    // 如果构造函数接受参数，
    // 则需要在声明（合约的构造函数）时提供，
    // 或在派生合约的构造函数位置以修饰器调用风格提供（见下文）。
    contract PriceFeed is owned, mortal, named("GoldFeed") {
       function updateInfo(uint newInfo) public {
          if (msg.sender == owner) info = newInfo;
       }

       function get() public view returns(uint r) { return info; }

       uint info;
    }
```

Note that in the above code, we call `mortal.kill()` To "forward" the destruction request. This approach is problematic, as shown in the following example:

```bash
    pragma solidity ^0.4.0;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* 清除操作 1 */ mortal.kill(); }
    }

    contract Base2 is mortal {
        function kill() public { /* 清除操作 2 */ mortal.kill(); }
    }

    contract Final is Base1, Base2 {
    }
```

Call ``Final.kill()`` farthest derived overloaded function is called ``Base2.kill``，but it will bypass ``Base1.kill``，mainly because it doesn't even know ``Base1`` existence. the way to solve this problem is to use ``super``:

```bash
    pragma solidity ^0.4.0;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* 清除操作 1 */ super.kill(); }
    }


    contract Base2 is mortal {
        function kill() public { /* 清除操作 2 */ super.kill(); }
    }

    contract Final is Base1, Base2 {
    }
```

If `Base2` call `super` it does not simply call the function on its base class contract.
On the contrary, it calls this function in the next base class contract of the final inheritance relation graph, so it calls `Base1.kill()` (Note that the Final inheritance sequence is -- starting from the farthest derived contract: Final, Base2, Base1, epoch, ownerd). The actual function called using super in the class is unknown in the context of the current class although its type is known.
This is similar to a common virtual method to find.

### Parameters of the base class constructor

The derived contract needs to provide all the parameters required by the base class constructor. this can be done in two ways:

```bash
    pragma solidity ^0.4.0;

    contract Base {
        uint x;
        function Base(uint _x) public { x = _x; }
    }

    contract Derived is Base(7) {
        function Derived(uint _y) Base(_y * _y) public {
        }
    }
```

One method directly calls the base class constructor in the inheritance list（``is Base(7)``）another method is like the | modifier | Usage method, as part of the derived contract constructor definition header（``Base(_y * _y)``). If the constructor parameter is a constant and defines or describes the behavior of the contract, it is more convenient to use the first method. If the parameters of the base class constructor depend on the derived contract, the second method must be used. If, like this simple example, both parts are used, the | modifier | Style parameter is preferred.

### Multiple inheritance and linearization

Several problems need to be solved for programming languages to implement multiple inheritance.
One problem is[Diamond problem](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem)
Solidity uses Python as a reference and uses [C3 线性化](https://en.wikipedia.org/wiki/C3_linearization) forces a DAG (directed acyclic graph) composed of the base class to maintain a specific order. This is finally reflected as the unique result we hope, but it also makes some inheritance methods invalid. In particular, the base class in is The following order is very important. In the following code, Solidity gives an error such as "Linearization of inheritance graph impossible.

```bash

    // 以下代码编译出错

    pragma solidity ^0.4.0;

    contract X {}
    contract A is X {}
    contract C is A, X {}
```

The cause of code compilation error is ``C`` requirement ``X`` override ``A`` （because the order of definition is ``A, X`` ），
but ``A`` it self requires rewriting ``X``，this conflict cannot be resolved.

You can remember it through a simple rule: from "most base-like" to "most derived" to specify all base classes.

#### Inheriting different types of members with the same name

This is considered an error when inheritance causes a contract to have functions and | modifier | With the same name.
If an event has the same name as | modifier | Or a function has the same name as an event, it is also considered an error.
An exception is that the getter of a state variable can overwrite a public function.

### Abstract contract

The contract function can be missing implementation, as shown in the following example (note that the function declaration header is `;` end):

```
    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }
```

These contracts cannot be compiled successfully (even if they contain other implemented functions besides unimplemented functions), they can be used as base class contracts:

```bash
    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public returns (bytes32) { return "miaow"; }
    }
```

If a contract inherits from an abstract contract and does not implement all unimplemented functions by rewriting, it is abstract in itself.

### interface

Interfaces are similar to abstract contracts, but they cannot implement any functions. There are further restrictions:

- Unable to inherit other contracts or interfaces.
- The constructor cannot be defined.
- Variables cannot be defined.
- Unable to define structure
- Unable to define enumeration.

Some restrictions here may be lifted in the future.

Interfaces are basically limited to what the contract ABI can represent, and the conversion between ABI and interfaces should not lose any information.

Interfaces are represented by their own keywords:

```bash

    pragma solidity ^0.4.11;

    interface Token {
        function transfer(address recipient, uint amount) public;
    }
```

Just like inheriting other contracts, contracts can inherit interfaces.

### Library

Libraries are similar to contracts, they only need to be deployed at a specific address once, and their code can be passed through EVM's DELEGATECALL (Previously used Homestead CALLCODE Keyword) features for reuse. This means that if the library function is called, its code is executed in the context of the call contract, that is this Points to the call contract, especially the storage that can access the call contract. Because each library is a piece of independent code, it can only access the state variables explicitly provided by the call contract (otherwise it cannot access these variables by name). Because we assume that libraries are stateless, so if they do not modify the state (that is, if they are view Or pure Function), library functions can only be used by direct calls (that is, do not use DELEGATECALL Key words), in particular, it is impossible to destroy any library unless Solidity type systems can be avoided.

Libraries can be seen as implicit base class contracts that use their contracts. Although they are not explicitly visible in inheritance relationships, calling library functions is very similar to calling explicit base class contracts (if L If it is a library, it can be used L.f() Call the library function). In addition, just as the library is a base class contract, for all contracts that use the library, internal Functions are visible.
Of course, internal calling conventions must be used to call internal functions, which means that all internal types and memory types are passed by reference rather than replication. To implement these in EVM, the code of the internal library function and all the functions called from it are pulled into the call contract at the compilation stage, and then use one JUMP Call to replace DELEGATECALL .

The following example shows how to use the library（a better example of the implementation set）:

```bash
    pragma solidity ^0.4.16;

    library Set {
      // 我们定义了一个新的结构体数据类型，用于在调用合约中保存数据。
      struct Data { mapping(uint => bool) flags; }

      // 注意第一个参数是“storage reference”类型，因此在调用中参数传递的只是它的存储地址而不是内容。
      // 这是库函数的一个特性。如果该函数可以被视为对象的方法，则习惯称第一个参数为 `self` 。
      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
              return false; // 已经存在
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // 不存在
          self.flags[value] = false;
          return true;
      }

      function contains(Data storage self, uint value)
          public
          view
          returns (bool)
      {
          return self.flags[value];
      }
    }

    contract C {
        Set.Data knownValues;

        function register(uint value) public {
            // 不需要库的特定实例就可以调用库函数，
            // 因为当前合约就是“instance”。
            require(Set.insert(knownValues, value));
        }
        // 如果我们愿意，我们也可以在这个合约中直接访问 knownValues.flags。
    }
```

Of course, you don't have to use libraries in this way: They can also be used without defining structural data types. Functions also do not require any storage reference parameters. Libraries can appear anywhere and have multiple storage reference parameters.

call ``Set.contains``，``Set.insert`` and ``Set.remove`` all are compiled as external calls（ ``DELEGATECALL`` ). If you use libraries, note that external function calls are actually executed. ``msg.sender``， ``msg.value`` and ``this`` values will be retained in the call (before Homestead, because ``CALLCODE``，changed ``msg.sender`` and ``msg.value``)。

The following example shows how to use memory types and internal functions in a library to implement custom types without paying for the overhead of external function calls:

```bash

    pragma solidity ^0.4.16;

    library BigInt {
        struct bigint {
            uint[] limbs;
        }

        function fromUint(uint x) internal pure returns (bigint r) {
            r.limbs = new uint[](1);
            r.limbs[0] = x;
        }

        function add(bigint _a, bigint _b) internal pure returns (bigint r) {
            r.limbs = new uint[](max(_a.limbs.length, _b.limbs.length));
            uint carry = 0;
            for (uint i = 0; i < r.limbs.length; ++i) {
                uint a = limb(_a, i);
                uint b = limb(_b, i);
                r.limbs[i] = a + b + carry;
                if (a + b < a || (a + b == uint(-1) && carry > 0))
                    carry = 1;
                else
                    carry = 0;
            }
            if (carry > 0) {
                // 太差了，我们需要增加一个 limb
                uint[] memory newLimbs = new uint[](r.limbs.length + 1);
                for (i = 0; i < r.limbs.length; ++i)
                    newLimbs[i] = r.limbs[i];
                newLimbs[i] = carry;
                r.limbs = newLimbs;
            }
        }

        function limb(bigint _a, uint _limb) internal pure returns (uint) {
            return _limb < _a.limbs.length ? _a.limbs[_limb] : 0;
        }

        function max(uint a, uint b) private pure returns (uint) {
            return a > b ? a : b;
        }
    }

    contract C {
        using BigInt for BigInt.bigint;

        function f() public pure {
            var x = BigInt.fromUint(7);
            var y = BigInt.fromUint(uint(-1));
            var z = x.add(y);
        }
    }
```

Since the compiler cannot know the deployment location of the library, we need to fill these addresses in the final bytecode through the linker.
If these addresses are not passed to the compiler as parameters, the compiled hexadecimal code will contain __Set______ Placeholder of the form (where Set Is the name of the library). You can manually fill in the address to replace the 40 characters with the hexadecimal code of the library contract address.

Compared with contracts, library restrictions:

- No state variables
- Unable to inherit or be inherited
- Unable to receive SIMPLE coins

（These restrictions may be lifted in the future）

#### Library call protection

If the library code is passed `CALL` To execute, not `DELEGATECALL` Or `CALLCODE` Then the execution result will be rolled back unless it is right `view` Or `pure` The call of the function. EVM does not provide a check for the contract whether to use `CALL` But the contract can use `ADDRESS` The operation code finds the running location ". The generated code determines the call mode by comparing this address with the constructed address.

More specifically, the runtime code of the library always starts with a push instruction, which is zero of 20 bytes at compile time. When the deployment code runs, this constant replaced by the current address in memory, the modified code is stored in the contract. At runtime, this causes the deployment address to be the first constant pushed to the stack, For any non-view and non-pure functions, the scheduler code compares whether the current address is consistent with this constant.

### Using For

Command ``using A for B;`` can be used to attach library functions (from library `A` ) to any type ( `B` ).
These functions will receive the object that calls them as their first parameter (like Python's self Variable).
``using A for *;`` the effect of IS, library `A` The function in is attached to any type. In both cases, all functions are appended with a parameter even if their first parameter type does not match the type of the object.
Type check is performed only when function calls and overload parsing are performed. `using A for B`; The directive is only valid in the current scope and only in the current contract. It may be upgraded to the global scope in the future. By introducing a module, you can use data types including library functions without adding code.

Let's use this way `libraries` Rewrite the set example in:

```bash
    pragma solidity ^0.4.16;

    // 这是和之前一样的代码，只是没有注释。
    library Set {
      struct Data { mapping(uint => bool) flags; }

      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
            return false; // 已经存在
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // 不存在
          self.flags[value] = false;
          return true;
      }

      function contains(Data storage self, uint value)
          public
          view
          returns (bool)
      {
          return self.flags[value];
      }
    }

    contract C {
        using Set for Set.Data; // 这里是关键的修改
        Set.Data knownValues;

        function register(uint value) public {
            // Here, all variables of type Set.Data have
            // corresponding member functions.
            // The following function call is identical to
            // `Set.insert(knownValues, value)`
            // 这里， Set.Data 类型的所有变量都有与之相对应的成员函数。
            // 下面的函数调用和 `Set.insert(knownValues, value)` 的效果完全相同。
            require(knownValues.insert(value));
        }
    }
```

You can also extend basic types like this:

```bash

    pragma solidity ^0.4.16;

    library Search {
        function indexOf(uint[] storage self, uint value)
            public
            view
            returns (uint)
        {
            for (uint i = 0; i < self.length; i++)
                if (self[i] == value) return i;
            return uint(-1);
        }
    }

    contract C {
        using Search for uint[];
        uint[] data;

        function append(uint value) public {
            data.push(value);
        }

        function replace(uint _old, uint _new) public {
            // 执行库函数调用
            uint index = data.indexOf(_old);
            if (index == uint(-1))
                data.push(_new);
            else
                data[index] = _new;
        }
    }

```

Note that all library calls are actual EVM function calls. This means that if the memory or value type is passed, a copy will be generated, even if self Variables.] Using a storage reference variable is the only case where copying does not occur.

## Solidity Assembly

Solidity defines an assembly language that can be used without Solidity. This assembly language can also be embedded into Solidity source code as "inline assembly. We begin with how to use inline assembly, introduce how it differs from independent assembly language, and then describe this assembly language in detail.

### Inline assembly

In order to achieve finer-grained control, especially to enhance the language by writing libraries, you can use a language close to the virtual machine to combine inline assembly with Solidity statements.
Since EVM is a stack-based virtual machine, it is usually difficult to accurately locate the address of the slot (storage location) in the stack and provide the correct stack location for the operation code to obtain parameters.
Solidity's inline assembly attempts to solve this problem and possible problems when writing assembly code manually by providing the following features:

* Function style operation code： ``mul(1, add(2, 3))`` rather ``push1 3 push1 2 add push1 1 mul``
* Assemble local variables： ``let x := add(2, 3)  let y := mload(0x40)  x := add(x, y)``
* External variables can be accessed： ``function f(uint x) public { assembly { x := sub(x, 1) } }``
* Label： ``let x := 10  repeat: x := sub(x, 1) jumpi(repeat, eq(x, 0))``
* Circulation： ``for { let i := 0 } lt(i, x) { i := add(i, 1) } { y := mul(2, y) }``
* if statement: ``if slt(x, 0) { x := sub(0, x) }``
* switch statent： ``switch x case 0 { y := mul(x, 2) } default { y := 0 }``
* Function call： ``function f(x) -> y { switch x case 0 { y := 1 } default { y := mul(x, f(sub(x, 1))) }   }``

Now let's explain the inline assembly language in detail.

> Inline assembly is a language that accesses Simplechain virtual machines at the underlying layer. This abandons many important security features provided by Solidity.

> TODO：describes the nuances of scope rules in an inline assembly and the complexity of using internal functions of library contracts. In addition, symbols about compiler definitions are also written.

Example
-------

The following example shows the code of a library contract, which can get the code of another contract and load it into a `bytes` Variable.
This is impossible for "conventional Solidity", and Assembly Library contracts can enhance language characteristics in this way.

```bash

    pragma solidity ^0.4.0;

    library GetCode {
        function at(address _addr) public view returns (bytes o_code) {
            assembly {
                // 获取代码大小，这需要汇编语言
                let size := extcodesize(_addr)
                // 分配输出字节数组 – 这也可以不用汇编语言来实现
                // 通过使用 o_code = new bytes（size）
                o_code := mload(0x40)
                // 包括补位在内新的“memory end”
                mstore(0x40, add(o_code, and(add(add(size, 0x20), 0x1f), not(0x1f))))
                // 把长度保存到内存中
                mstore(o_code, size)
                // 实际获取代码，这需要汇编语言
                extcodecopy(_addr, add(o_code, 0x20), 0, size)
            }
        }
    }
```

When the optimizer cannot generate efficient code, inline assembly may also be more beneficial. Note that it is definitely more difficult to write assembly code because the compiler cannot check assembly statements; Therefore, you need to use it only when dealing with some relatively complex problems, and you need to know clearly what you want to do.

```bash

    pragma solidity ^0.4.16;

    library VectorSum {
        // 因为目前的优化器在访问数组时无法移除边界检查，
        // 所以这个函数的执行效率比较低。
        function sumSolidity(uint[] _data) public view returns (uint o_sum) {
            for (uint i = 0; i < _data.length; ++i)
                o_sum += _data[i];
        }

        // 我们知道我们只能在数组范围内访问数组元素，所以我们可以在内联汇编中不做边界检查。
        // 由于 ABI 编码中数组数据的第一个字（32 字节）的位置保存的是数组长度，
        // 所以我们在访问数组元素时需要加入 0x20 作为偏移量。
        function sumAsm(uint[] _data) public view returns (uint o_sum) {
            for (uint i = 0; i < _data.length; ++i) {
                assembly {
                    o_sum := add(o_sum, mload(add(add(_data, 0x20), mul(i, 0x20))))
                }
            }
        }

        // 和上面一样，但在内联汇编内完成整个代码。
        function sumPureAsm(uint[] _data) public view returns (uint o_sum) {
            assembly {
               // 取得数组长度（前 32 字节）
               let len := mload(_data)

               // 略过长度字段。
               //
               // 保持临时变量以便它可以在原地增加。
               //
               // 注意：对 _data 数值的增加将导致 _data 在这个汇编语句块之后不再可用。
               //      因为无法再基于 _data 来解析后续的数组数据。
               let data := add(_data, 0x20)

               // 迭代到数组数据结束
               for
                   { let end := add(data, mul(len, 0x20)) }
                   lt(data, end)
                   { data := add(data, 0x20) }
               {
                   o_sum := add(o_sum, mload(data))
               }
            }
        }
    }
```


#### Syntax

Like Solidity, Assembly also parses comments, text, and identifiers, so you can use the usual ``//`` and ``/* */`` to comment.
The inline assembler is composed ``assembly { ... }`` to mark, the following can be used in these braces (see later for more details).

 - Literal constant, that is ``0x123``、``42`` or ``"abc"`` （a string not exceeding 32 characters）
 - Operation code（within “instruction style”），such ``mload sload dup1 sstore``，please see the following operation code list
 - Function-style opcodes, such ``add(1，mlod(0))``
 - Tags, such``name:``
 - Variable declaration, such ``let x := 7``、``let x := add(y, 3)`` or ``let x`` （the initial value will be set to empty(0))）
 - Identifiers (tags or assembly local variables and external variables used as inline assemblies), such `jump(name)` , `3 x add`
 - Assignment (within "instruction style"), such 3 =: x
 - Function style assignment, such x := add(y，3)
 - Some statement blocks that control the scope of local variables, such {let x := 3 { let y := add(x，1) }}

#### Opcode

Reference operation code:

If an opcode requires parameters (always from the top of the stack), they are given in parentheses. Note: the order of parameters can be considered as a reverse order in a non-functional style (explained below). Marked - The operation code of does not push data into the stack, marked * The operation code of has special operations, while all other operations will only push one data into the push stack.
For ``F``、``H``、``B`` or ``C`` marked opcodes represent that they are introduced from Frontier, Homestead, Byzantium, or Constantinople. Constantinople is still planned, so it is marked C Currently, an invalid instruction is abnormal. In the following table, ``mem[a...b)`` indicates from position ``a`` start to（excluding）position ``b`` number of memory bytes,``storage[p]`` indicates the position ``p`` storage content.
``pushi`` and ``jumpdest`` these two operation codes cannot be used directly.

In the syntax table, the opcode is provided as a predefined identifier.

Instruction | symbol |Bool | Explanation  
-|-|-|-
| stop  | `-` | F | 停止执行，与 return(0,0) 等价  |                                |
| add(x, y)               |     | F | x + y       |
| sub(x, y)               |     | F | x - y                                                           |
| mul(x, y)               |     | F | x * y                                                           |
| div(x, y)               |     | F | x / y                                                           |
| sdiv(x, y)              |     | F | x / y，以二进制补码作为符号                                     |
| mod(x, y)               |     | F | x % y                                                           |
| smod(x, y)              |     | F | x % y，以二进制补码作为符号                                     |
| exp(x, y)               |     | F | x 的 y 次幂                                                     |
| not(x)                  |     | F | ~x，对 x 按位取反                                               |
| lt(x, y)                |     | F | 如果 x < y 为 1，否则为 0                                       |
| gt(x, y)                |     | F | 如果 x > y 为 1，否则为 0                                       |
| slt(x, y)               |     | F | 如果 x < y 为 1，否则为 0，以二进制补码作为符号                 |
| sgt(x, y)               |     | F | 如果 x > y 为 1，否则为 0，以二进制补码作为符号                 |
| eq(x, y)                |     | F | 如果 x == y 为 1，否则为 0                                      |
| iszero(x)               |     | F | 如果 x == 0 为 1，否则为 0                                      |
| and(x, y)               |     | F | x 和 y 的按位与                                                 |
| or(x, y)                |     | F | x 和 y 的按位或                                                 |
| xor(x, y)               |     | F | x 和 y 的按位异或                                               |
| byte(n, x)              |     | F | x 的第 n 个字节，这个索引是从 0 开始的                          |
| shl(x, y)               |     | C | 将 y 逻辑左移 x 位                                              |
| shr(x, y)               |     | C | 将 y 逻��右移 x 位                                              |
| sar(x, y)               |     | C | 将 y 算术右移 x 位                                              |
| addmod(x, y, m)         |     | F | 任意精度的 (x + y) % m                                          |
| mulmod(x, y, m)         |     | F | 任意精度的 (x * y) % m                                          |
| signextend(i, x)        |     | F | 对 x 的最低位到第 (i * 8 + 7) 进行符号扩展                      |
| keccak256(p, n)         |     | F | keccak(mem[p...(p + n)))                                        |
| jump(label)             | `-` | F | 跳转到标签 / 代码位置                                           |
| jumpi(label, cond)      | `-` | F | 如果条件为非零，跳转到标签                                      |
| pc                      |     | F | 当前代码位置                                                    |
| pop(x)                  | `-` | F | 删除（弹出）栈顶的 x 个元素                                     |
| dup1 ... dup16          |     | F | 将栈内第 i 个元素（从栈顶算起）复制到栈顶                       |
| swap1 ... swap16        | `*` | F | 将栈顶元素和其下第 i 个元素互换                                 |
| mload(p)                |     | F | mem[p...(p + 32))                                               |
| mstore(p, v)            | `-` | F | mem[p...(p + 32)) := v                                          |
| mstore8(p, v)           | `-` | F | mem[p] := v & 0xff （仅修改一个字节）                           |
| sload(p)                |     | F | storage[p]                                                      |
| sstore(p, v)            | `-` | F | storage[p] := v                                                 |
| msize                   |     | F | 内存大小，即最大可访问内存索引                                  |
| gas                     |     | F | 执行可用的 gas                                                  |
| address                 |     | F | 当前合约 / 执行上下文的地址                                     |
| balance(a)              |     | F | 地址 a 的余额，以 wei 为单位                                    |
| caller                  |     | F | 调用发起者（不包括 ``delegatecall``）                           |
| callvalue               |     | F | 随调用发送的 Wei 的数量                                         |
| calldataload(p)         |     | F | 位置 p 的调用数据（32 字节）                                    |
| calldatasize            |     | F | 调用数据的字节数大小                                            |
| calldatacopy(t, f, s)   | `-` | F | 从调用数据的位置 f 的拷贝 s 个字节到内存的位置 t                |
| codesize                |     | F | 当前合约 / 执行上下文地址的代码大小                             |
| codecopy(t, f, s)       | `-` | F | 从代码的位置 f 开始拷贝 s 个字节到内存的位置 t                  |
| extcodesize(a)          |     | F | 地址 a 的代码大小                                               |
| extcodecopy(a, t, f, s) | `-` | F | 和 codecopy(t, f, s) 类似，但从地址 a 获取代码                  |
| returndatasize          |     | B | 最后一个 returndata 的大小                                      |
| returndatacopy(t, f, s) | `-` | B | 从 returndata 的位置 f 拷贝 s 个字节到内存的位置 t              |
| create(v, p, s)         |     | F | 用 mem[p...(p + s)) 中的代码创建一个新合约、发送 v wei 并返回新地址    |
| create2(v, n, p, s)     |     | C | 用 mem[p...(p + s)) 中的代码，在地址keccak256(<address> . n . keccak256(mem[p...(p + s)))创建新合约、发送 v wei 并返回新地址上 |
| call(g, a, v, in,insize, out, outsize)       |     | F | 使用 mem[in...(in + insize)) 作为输入数据，提供 g gas 和 v wei 对地址 a 发起消息调用，输出结果数据保存在 mem[out...(out + outsize))，发生错误（比如 gas 不足）时返回 0，正确结束返回 1 |                
| callcode(g, a, v, in,insize, out, outsize)   |     | F | 与 ``call`` 等价，但仅使用地址 a 中的代码 且保持当前合约的执行上下文    |
| delegatecall(g, a, in,insize, out, outsize)  |     | F | 与 ``callcode`` 等价且保留 ``caller`` 和 ``callvalue``          |
| staticcall(g, a, in,insize, out, outsize)    |     | F | 与 ``call(g, a, 0, in, insize, out, outsize)`` 等价,但不允许状态修改  |
| return(p, s)            | `-` | F | 终止运行，返回 mem[p...(p + s)) 的数据                          |
| revert(p, s)            | `-` | B | 终止运行，撤销状态变化，返回 mem[p...(p + s)) 的数据            |
| selfdestruct(a)         | `-` | F | 终止运行，销毁当前合约并且把资金发送到地址 a                    |
| invalid                 | `-` | F | 以无效指令终止运行                                              |
| log0(p, s)              | `-` | F | 以 mem[p...(p + s)) 的数据产生不带 topic 的日志                 |
| log1(p, s, t1)          | `-` | F | 以 mem[p...(p + s)) 的数据和 topic t1 产生日志                  |
| log2(p, s, t1, t2)      | `-` | F | 以 mem[p...(p + s)) 的数据和 topic t1、t2 产生日志              |
| log3(p, s, t1, t2, t3)  | `-` | F | 以 mem[p...(p + s)) 的数据和 topic t1、t2、t3 产生日志          |
| log4(p, s, t1, t2, t3, t4)   | `-` | F | 以 mem[p...(p + s)) 的数据和 topic t1、t2、t3 和 t4 产生日志    |
| origin                  |     | F | 交易发起者地址                                                  |
| gasprice                |     | F | 交易所指定的 gas 价格                                           |
| blockhash(b)            |     | F | 区块号 b 的哈希 - 目前仅适用于不包括当前区块的最后 256 个区块   |
| coinbase                |     | F | 当前的挖矿收益者地址                                            |
| timestamp               |     | F | 从当前 epoch 开始的当前区块时间戳（以秒为单位）                 |
| number                  |     | F | 当前区块号                                                      |
| difficulty              |     | F | 当前区块难度                                                    |
| gaslimit                |     | F | 当前区块的 gas 上限                                             |
| |  |

#### Literal constant

You can directly type decimal or hexadecimal symbols to use as integer constants, which automatically generates the corresponding `PUSHi` Instruction.
The following code calculates 2 plus 3 (equal to 5), and then calculates its bitwise sum with the string "abc. The string is left aligned when stored and cannot exceed 32 bytes in length.

```bash
assembly { 2 3 add "abc" and }
```

#### Function style

You can type an opcode after the opcode just like using a bytecode. For example, put `3` And memory location `0x80` Add the data

```bash
3 0x80 mload add 0x80 mstore
```

Because it is usually difficult to see the actual parameters of Some opcodes, Solidity inline assembly also provides a "function style" representation, code with the same function can be written

```bash
mstore(0x80, add(mload(0x80), 3))
```

Instruction style cannot be used in function style expressions, that is ``1 2 mstore(0x80, add)`` is an invalid assembly statement,
It must be written ``mstore(0x80, add(2, 1))`` this form. For opcodes without parameters, brackets can be omitted.

Note that in function style writing, the order of parameters is opposite to the instruction style. If you use function style, the first parameter is at the top of the stack.

#### Access external variables and functions

Solidity variables and other identifiers can be accessed by simply using their names. For memory variables, this pushes the address instead of the value into the stack.
Storage variables are different because the value of the storage variable may not occupy the complete storage slot, so its "address" consists of the byte offset in the storage slot and the slot.
To obtain variables `x` The storage slot used, you can use `x_slot` , and `x_offset` Gets its byte offset.

In assignment statements (see below), we can even use Solidity local variables to assign values.

External functions can also be accessed for inline assemblies: the Assembly pushes their entry tags (with virtual function parsing) into the stack. The call semantics in Solidity are:

 - Caller press in ``return label``、``arg1``、``arg2``、...、``argn``
 - The caller returns``ret1``、``ret2``、...、``retm``

This feature is still a little troublesome to use, because the stack offset has changed fundamentally during the call, so the reference to local variables will go wrong.

```bash

    pragma solidity ^0.4.11;

    contract C {
        uint b;
        function f(uint x) public returns (uint r) {
            assembly {
                r := mul(x, sload(b_slot)) // 因为偏移量为 0，所以可以忽略
            }
        }
    }
```


> If you access a data type with actual data digits less than 256 bits（such ``uint64``、``address``、``bytes16`` or ``byte``），do not make any assumptions about the values on the unused data bits of this type after encoding. In particular, do not assume that they must be 0. For security reasons, before using this data in a context, you must clear the data to 0, which is very important: ``uint32 x = f(); assembly { x := and(x, 0xffffffff) /* now use x */ }``To clear the symbolic type, you can use `signextend` operation code.

#### Label

> Tags are not recommended. Use functions, loops, if, or switch statements.

Another problem with EVM assembly is that jump and jumpi functions use absolute addresses, which are easy to change. Solidity inline assembly provides tags for easier use of jump. Note that tags have underlying characteristics, and efficient assembly code can be written using loop, if, and switch commands (see below) without tags. The following code is used to calculate an element in the Fibonacci sequence.

```bash

    {
        let n := calldataload(4)
        let a := 1
        let b := a
    loop:
        jumpi(loopend, eq(n, 0))
        a add swap1
        n := sub(n, 1)
        jump(loop)
    loopend:
        mstore(0, a)
        return(0, 0x20)
    }
```

Note: stack variables can be automatically accessed only when the assembler knows the current stack height. If the stack heights of the jump source and the target are different, the access fails. Although we can use jump in this way, in this case, you should not access variables in any stack (even assembly variables).

In addition, the stack height analyzer can also check the code opcode through the opcode (rather than according to the control flow), so in the following cases, the assembler `two` The stack height at will generate the wrong impression:

```bash

    {
        let x := 8
        jump(two)
        one:
            // 这里的栈高度是 2（因为我们压入了 x 和 7），
            // 但因为汇编程序是按顺序读取代码的，
            // 它会认为栈高度是 1。
            // 在这里访问栈变量 x 会导致错误。
            x := 9
            jump(three)
        two:
            7 // 把某个数据压入栈中
            jump(one)
        three:
    }
```

#### Assembly local variable declaration

You can use let Keywords to declare variables that are visible only in the inline assembly, actually only in the current `{...}` Visible in the block.
The following things should be: `let` The command creates a new data slot reserved for the variable and automatically deletes it when it reaches the end of the block.
You need to provide an initial value for the variable, which can only `0` But it can also be a complex function style expression.
```bash

    pragma solidity ^0.4.16;

    contract C {
        function f(uint x) public view returns (uint b) {
            assembly {
                let v := add(x, 1)
                mstore(0x80, v)
                {
                    let y := add(sload(v), 1)
                    b := y
                } // y 会在这里被“清除”
                b := add(b, v)
            } // v 会在这里被“清除”
        }
    }
```

#### Assignment

You can assign values to Assembly local variables and function local variables. Note: When assigning values to variables pointing to memory or storage, you just change the pointer instead of the data.

There are two assignment methods: Function style and instruction style. For function style assignment ( variable := value ), you need to provide a value in a function style expression, which can exactly generate a value in a stack;
For Instruction style assignment ( =: variable ), the data is only obtained from the top of the stack. For both methods, colons point to variable names. Assignment is achieved by replacing the variable values in the stack with new values.

```bash

    {
        let v := 0 // 作为变量声明的函数风格赋值
        let g := add(v, 2)
        sload(10)
        =: v // 指令风格的赋值，将 sload(10) 的结果赋给 v
    }
```

> Assignment of instruction style is not recommended.

#### If


if if statements can be used to conditionally execute code and do not have the "else" section; 

```bash

    {
        if eq(value, 0) { revert(0, 0) }
    }
```

Braces for the code body are required.

#### Switch

As a very preliminary version of "if/else", you can use the switch statement. It calculates the value of the expression and compares it with several constants. Select the branch corresponding to the matching constant.
Unlike some programming languages that are prone to errors, the control flow will not continue to be executed from one situation to the next. We can set a fallback or called default The default.

```bash

    {
        let x := 0
        switch calldataload(4)
        case 0 {
            x := calldataload(0x24)
        }
        default {
            x := calldataload(0x44)
        }
        sstore(0, div(x, 2))
    }
```

The Case list does not need braces, but the case body needs braces.

#### Circulating

Assembly language supports a simple for-style loop. The For-style loop has a header that contains the initialization part, conditions, and iteration post-processing parts.
The condition must be a function-style expression, while the other two parts are statement blocks. If the start part declares a variable, the scope of these variables will be extended to the loop body (including conditions and post-iteration parts).

The following example calculates the sum of values in a memory area.

```bash

    {
        let x := 0
        for { let i := 0 } lt(i, 0x100) { i := add(i, 0x20) } {
            x := add(x, mload(i))
        }
    }
```

The For loop can also be written as the while loop: only the initialization part and the iteration post-processing part are left blank.

```bash

    {
        let x := 0
        let i := 0
        for { } lt(i, 0x100) { } {     // while(i < 0x100)
            x := add(x, mload(i))
            i := add(i, 0x20)
        }
    }
```

#### Function

Assembly language allows you to define underlying functions. The underlying function needs to obtain their parameters (and return PC) from the stack and put the results into the stack. The method of calling a function is the same as that of executing a function-style opcode. Functions can be defined anywhere and visible in the statement blocks that declare them. Local variables defined outside the function cannot be accessed within the function. There is no strict here return Statement. If you call a function that returns multiple values, you must use `a，b：= f(x)` Or let `a，b：= f(x)` Assign them to a tuple.

The following example implements the power operation function by square multiplication.

```bash
{
  function power(base, exponent) -> result {
    switch exponent
    case 0 { result := 1 }
    case 1 { result := base }
    default {
        result := power(mul(base, base), div(exponent, 2))
        switch mod(exponent, 2)
        case 1 { result := mul(base, result) }
    } 
 }
}
```

#### Precautions

Inline assembly language may have a fairly high-level appearance, but in fact it is a very low-level programming language. Function calls, loops, if statements, and switch statements are converted through simple rewrite rules, Then, the only thing the assembler does for you is to reorganize function-style opcodes, manage jump tags, and calculate the stack height of access variables, in addition, the stack data of partial assembly variables is deleted when the end of the statement block is reached. Especially for the last two cases, the assembler will only calculate the height of the stack in the order of the code, not necessarily following the control process; Understanding this is very important. In addition, operations such as swap only exchange the data in the stack, not the variable position.

#### Solidity convention

Compared with EVM assembly language, Solidity can identify types less than 256 bits, such `uint24` . In order to improve efficiency, most arithmetic operations only regard them as 256 digits and clear unused data bits only when necessary, this is done only before they are written to memory or compared. This means that if you access such variables from an inline assembly you must first manually clear those unused data bits.

Solidity manages memory in a very simple way: in ``0x40`` location of has a "idle memory pointer". If you plan to allocate memory, just start using memory from here and update the pointer accordingly. The first 64 bytes of memory can be used as temporary storage space for temporary allocation ". The 32-byte position after the "free memory pointer" (that is, from `0x60` Start position) will always be 0, which can be used to initialize an empty dynamic memory array.

In Solidity, elements in memory arrays always occupy a multiple of 32 bytes (yes, even `byte[]` It's all like this, only `bytes` and `string` Not like this). A multi-dimensional memory array is a pointer to a memory array. The length of a dynamic array is stored in the first slot of the array, followed by array elements.

> Static memory arrays do not have length fields, but will soon increase. This is to better convert between static arrays and dynamic arrays, so do not rely on this.

### Independent Assembly

The assembly language described in the preceding inline assembly can also be used alone. In fact, the plan is to use it as an intermediate language for the Solidity compiler. In this sense, it tries to achieve the following goals:

1、Even if the code is generated by Solidity's compiler, the program written with it should also be readable.
2、From assembly to bytecode translation, "accidents" should be included as little as possible ".
3、The control flow should be easy to detect to help carry out formal test and optimization.

To achieve the first and last goals, the Assembly provides an advanced structure: such `for` Circulation, `if` Statement, `switch` Statements and function calls. Should be able to write without using explicit `SWAP` , `DUP` , `JUMP` And `JUMPI` Statement assembler,because the first two confuse the data stream, while the last two confuse the control flow. In addition, the form is `mul(add(x, y), 7)` The function style statement of is better `7 y x add` mul Because it is easier to see which operation is used for which operation code in the first form.

The second goal is to use a very regular way to treat the advanced instruction structure as a bytecode.
The only non-local operation performed by the assembler is the name search of the user-defined identifier (function, variable, azone), it follows very simple and fixed scope rules and clears local variables from the stack.

Scope: Identifiers (tags, variables, functions, assemblies) declared in it are only visible in declared statement blocks (including nested statement blocks in the current statement block). Even if they are within the scope of action, it is illegal to cross the function boundary to access local variables. Shadowing is prohibited. Local variables cannot be accessed before declaration, but tags, functions, and assemblies are possible.
Assembly is a special statement block, for example, used to return runtime codes or create contracts. Identifiers declared in Assembly statement blocks outside the subassembly are all invisible in the subassembly.

If the control flow passes through the end of the block, pop instructions matching the number of local variables declared in the current statement block are inserted. Whenever a local variable is referenced, the code generator needs to know the relative position in the current stack,
Therefore, it is necessary to track the current so-called stack height. Because all local variables declared in the statement block are clear at the end of the statement block, the stack height before and after the statement block should be the same. If this is not the case a warning will be issued.

Use `switch` , `for` And functions should be able to write complex code without manual calls `jump` Or `jumpi` . This will allow improved forms of laboratory certificates and optimization to analyze the control process more simply.

In addition, if manual redirection is allowed, the calculation stack height will be more complicated. The positions of all local variables in the stack need to be clearly known, otherwise, references to local variables cannot be automatically obtained at the end of the statement block, thus clearing them correctly.

Example:

We will refer to an instance from Solidity to assembly instructions. Consider the runtime bytecode of the following Solidity program:

    pragma solidity ^0.4.16;

    contract C {
      function f(uint x) public pure returns (uint y) {
        y = 1;
        for (uint i = 0; i < x; i++)
          y = 2 * y;
      }
    }

The following assembly instructions will be generated:

    {
      mstore(0x40, 0x60) // 保存“空闲内存指针”
      // 函数选择器
      switch div(calldataload(0), exp(2, 226))
      case 0xb3de648b {
        let r := f(calldataload(4))
        let ret := $allocate(0x20)
        mstore(ret, r)
        return(ret, 0x20)
      }
      default { revert(0, 0) }
      // 内存分配器
      function $allocate(size) -> pos {
        pos := mload(0x40)
        mstore(0x40, add(pos, size))
      }
      // 合约函数
      function f(x) -> y {
        y := 1
        for { let i := 0 } lt(i, x) { i := add(i, 1) } {
          y := mul(2, y)
        }
      }
    }

Assembly syntax
-----------------

Parser tasks are as follows:

- Convert the byte stream into a symbolic stream, and discard the comments in the C++ style (there is a special comment on the source code reference, and we will not explain it here).
- According to the following syntax, convert the symbol flow to AST.
- Register the identifier defined in the statement Block (annotated to the AST node) and indicate where the variable can be accessed from.

The Assembly lexical analyzer follows the rules defined by Solidity itself.

A space is used to separate all symbols. It consists of space characters, tabs, and line breaks. The annotation format is regular JavaScript/C envoy style and is interpreted as a space.

Grammar::

    AssemblyBlock = '{' AssemblyItem* '}'
    AssemblyItem =
        Identifier |
        AssemblyBlock |
        AssemblyExpression |
        AssemblyLocalDefinition |
        AssemblyAssignment |
        AssemblyStackAssignment |
        LabelDefinition |
        AssemblyIf |
        AssemblySwitch |
        AssemblyFunctionDefinition |
        AssemblyFor |
        'break' |
        'continue' |
        SubAssembly
    AssemblyExpression = AssemblyCall | Identifier | AssemblyLiteral
    AssemblyLiteral = NumberLiteral | StringLiteral | HexLiteral
    Identifier = [a-zA-Z_$] [a-zA-Z_0-9]*
    AssemblyCall = Identifier '(' ( AssemblyExpression ( ',' AssemblyExpression )* )? ')'
    AssemblyLocalDefinition = 'let' IdentifierOrList ( ':=' AssemblyExpression )?
    AssemblyAssignment = IdentifierOrList ':=' AssemblyExpression
    IdentifierOrList = Identifier | '(' IdentifierList ')'
    IdentifierList = Identifier ( ',' Identifier)*
    AssemblyStackAssignment = '=:' Identifier
    LabelDefinition = Identifier ':'
    AssemblyIf = 'if' AssemblyExpression AssemblyBlock
    AssemblySwitch = 'switch' AssemblyExpression AssemblyCase*
        ( 'default' AssemblyBlock )?
    AssemblyCase = 'case' AssemblyExpression AssemblyBlock
    AssemblyFunctionDefinition = 'function' Identifier '(' IdentifierList? ')'
        ( '->' '(' IdentifierList ')' )? AssemblyBlock
    AssemblyFor = 'for' ( AssemblyBlock | AssemblyExpression )
        AssemblyExpression ( AssemblyBlock | AssemblyExpression ) AssemblyBlock
    SubAssembly = 'assembly' Identifier AssemblyBlock
    NumberLiteral = HexNumber | DecimalNumber
    HexLiteral = 'hex' ('"' ([0-9a-fA-F]{2})* '"' | '\'' ([0-9a-fA-F]{2})* '\'')
    StringLiteral = '"' ([^"\r\n\\] | '\\' .)* '"'
    HexNumber = '0x' [0-9a-fA-F]+
    DecimalNumber = [0-9]+

