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

### sipc unit

The conversion between sipc units is to add after the number ``wei``、 ``finney``、 ``szabo`` or ``ether`` to implement, if there is no unit behind, the default is Wei. For example ``2 ether == 2000 finney`` logical judgment value of IS ``true``。

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


>  对于每一个**外部函数**调用，包括 ``msg.sender`` 和 ``msg.value`` 在内所有 ``msg`` 成员的值都会变化。这里包括对库函数的调用。

>  不要依赖 ``block.timestamp``、 ``now`` 和 ``blockhash`` 产生随机数，除非你知道自己在做什么。 时间戳和区块哈希在一定程度上都可能受到挖矿矿工影响。例如，挖矿社区中的恶意矿工可以用某个给定的哈希来运行赌场合约的 payout 函数，而如果他们没收到钱，还可以用一个不同的哈希重新尝试。当前区块的时间戳必须严格大于最后一个区块的时间戳，但这里唯一能确保的只是它会是在权威链上的两个连续区块的时间戳之间的数值。
    
>  基于可扩展因素，区块哈希不是对所有区块都有效。你仅仅可以访问最近 256 个区块的哈希，其余的哈希均为零。

### ABI 编码函数

- ``abi.encode(...) returns (bytes)``：`ABI`对给定参数进行编码
- ``abi.encodePacked(...) returns (bytes)``：对给定参数执行`紧打包编码`
- ``abi.encodeWithSelector(bytes4 selector, ...) returns (bytes)``：`ABI`对给定参数进行编码，并以给定的函数选择器作为起始的 4 字节数据一起返回
- ``abi.encodeWithSignature(string signature, ...) returns (bytes)``：等价于 ``abi.encodeWithSelector(bytes4(keccak256(signature), ...)``

> 这些编码函数可以用来构造函数调用数据，而不用实际进行调用。此外，``keccak256(abi.encodePacked(a, b))`` 是更准确的方法来计算在未来版本不推荐使用的 ``keccak256(a, b)``。

更多详情请参考`ABI`和 `紧打包编码`。


#### 错误处理

``assert(bool condition)``:
    如果条件不满足，则使当前交易没有效果 — 用于检查内部错误。
``require(bool condition)``:
    如果条件不满足则撤销状态更改 - 用于检查由输入或者外部组件引起的错误。 
``require(bool condition, string message)``:
    如果条件不满足则撤销状态更改 - 用于检查由输入或者外部组件引起的错误，可以同时提供一个错误消息。
``revert()``:
    终止运行并撤销状态更改。
``revert(string reason)``:
    终止运行并撤销状态更改，可以同时提供一个解释性的字符串。

#### 数学和密码学函数

``addmod(uint x, uint y, uint k) returns (uint)``:计算 ``(x + y) % k``，加法会在任意精度下执行，并且加法的结果即使超过 ``2**256`` 也不会被截取。从 0.5.0 版本的编译器开始会加入对 ``k != 0`` 的校验（assert）。

``mulmod(uint x, uint y, uint k) returns (uint)``:计算 ``(x * y) % k``，乘法会在任意精度下执行，并且乘法的结果即使超过 ``2**256`` 也不会被截取。从 0.5.0 版本的编译器开始会加入对 ``k != 0`` 的校验（assert）。

``keccak256(...) returns (bytes32)``: 计算 :ref:`(tightly packed) arguments <abi_packed_mode>` 的 Ethereum-SHA-3 （Keccak-256）哈希。

``sha256(...) returns (bytes32)``:计算 :ref:`(tightly packed) arguments <abi_packed_mode>` 的 SHA-256 哈希。

``sha3(...) returns (bytes32)``:等价于 keccak256。

``ripemd160(...) returns (bytes20)``:计算 :ref:`(tightly packed) arguments <abi_packed_mode>` 的 RIPEMD-160 哈希。

``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)`` ：利用椭圆曲线签名恢复与公钥相关的地址，错误返回零值。[example usage](https://ethereum.stackexchange.com/q/1777/222)

上文中的`tightly packed`是指不会对参数值进行 `padding` 处理（就是说所有参数值的字节码是连续存放的，译者注），这意味着下边这些调用都是等价的：

    keccak256("ab", "c")
    keccak256("abc")
    keccak256(0x616263)
    keccak256(6382179)
    keccak256(97, 98, 99)

如果需要 padding，可以使用显式类型转换：``keccak256("\x00\x12")`` 和 ``keccak256(uint16(0x12))`` 是一样的。

请注意，常量值会使用存储它们所需要的最少字节数进行打包。例如：``keccak256(0) == keccak256(uint8(0))``，``keccak256(0x12345678) == keccak256(uint32(0x12345678))``。

在一个私链上，你很有可能碰到由于 ``sha256``、``ripemd160`` 或者 ``ecrecover`` 引起的 Out-of-Gas。原因是因为这些密码学函数在Simplechain虚拟机中以“预编译合约”形式存在的，且在第一次收到消息后才被真正存在（尽管合约代码是EVM中已存在的硬编码）。因此发送到不存在的合约的消息非常昂贵，所以实际的执行会导致 Out-of-Gas 错误。在你实际使用你的合约之前，给每个合约发送一点儿Sipc，比如 1 Wei。这在官方网络或测试网络上不是问题。

#### 地址相关

``<address>.balance`` (``uint256``):以 Wei 为单位的 :ref:`address` 的余额。

``<address>.transfer(uint256 amount)``:向 :ref:`address` 发送数量为 amount 的 Wei，失败时抛出异常，发送 2300 gas 的矿工费，不可调节。

``<address>.send(uint256 amount) returns (bool)``:向 :ref:`address` 发送数量为 amount 的 Wei，失败时返回 ``false``，发送 2300 gas 的矿工费用，不可调节。

``<address>.call(...) returns (bool)``:发出低级函数 ``CALL``，失败时返回 ``false``，发送所有可用 gas，可调节。

``<address>.callcode(...) returns (bool)``：发出低级函数 ``CALLCODE``，失败时返回 ``false``，发送所有可用 gas，可调节。

``<address>.delegatecall(...) returns (bool)``:发出低级函数 ``DELEGATECALL``，失败时返回 ``false``，发送所有可用 gas，可调节。

更多信息，参考 :ref:`address` 部分：

>   使用 send 有很多危险：如果调用栈深度已经达到 1024（这总是可以由调用者所强制指定），转账会失败；并且如果接收者用光了 gas，转账同样会失败。为了保证以太币转账安全，总是检查 ``send`` 的返回值，利用 ``transfer`` 或者下面更好的方式： 用这种接收者取回钱的模式。

>  如果在通过低级函数 delegatecall 发起调用时需要访问存储中的变量，那么这两个合约的存储中的变量定义顺序需要一致，以便被调用的合约代码可以正确地通过变量名访问合约的存储变量。这当然不是指像在高级的库函数调用时所传递的存储变量指针那样的情况。

>  不鼓励使用 ``callcode``，并且将来它会被移除。

合约相关


``this`` (current contract's type):当前合约，可以明确转换为 :ref:`address`。

``selfdestruct(address recipient)``:销毁合约，并把余额发送到指定 :ref:`address`。

``suicide(address recipient)``:与 selfdestruct 等价，但已不推荐使用。

此外，当前合约内的所有函数都可以被直接调用，包括当前函数。


## 表达式和控制结构

### 输入参数和输出参数

与 Javascript 一样，函数可能需要参数作为输入;而与 Javascript 和 C 不同的是，它们可能返回任意数量的参数作为输出。

#### 输入参数

输入参数的声明方式与变量相同。但是有一个例外，未使用的参数可以省略参数名。
例如，如果我们希望合约接受有两个整数形参的函数的外部调用，我们会像下面这样写

```bash
    pragma solidity ^0.4.16;

    contract Simple {
        function taker(uint _a, uint _b) public pure {
            // 用 _a 和 _b 实现相关功能.
        }
    }
```

#### 输出参数


输出参数的声明方式在关键词 ``returns`` 之后，与输入参数的声明方式相同。
例如，如果我们需要返回两个结果：两个给定整数的和与积，我们应该写作

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

输出参数名可以被省略。输出值也可以使用 ``return`` 语句指定。
``return`` 语句也可以返回多值，参阅：ref:`multi-return`。
返回的输出参数被初始化为 0；如果它们没有被显式赋值，它们就会一直为 0。

输入参数和输出参数可以在函数体中用作表达式。因此，它们也可用在等号左边被赋值。

### 控制结构

JavaScript 中的大部分控制结构在 Solidity 中都是可用的，除了 ``switch`` 和 ``goto``。因此 Solidity 中有 ``if``，``else``，``while``，``do``，``for``，``break``，``continue``，``return``，``? :`` 这些与在 C 或者 JavaScript 中表达相同语义的关键词。

用于表示条件的括号 *不可以* 被省略，单语句体两边的花括号可以被省略。注意，与 C 和 JavaScript 不同， Solidity 中非布尔类型数值不能转换为布尔类型，因此 ``if (1) { ... }`` 的写法在 Solidity 中 *无效* 。

### 返回多个值

当一个函数有多个输出参数时， ``return (v0, v1, ...,vn)`` 写法可以返回多个值。不过元素的个数必须与输出参数的个数相同。

### 函数调用

#### 内部函数调用

当前合约中的函数可以直接（“从内部”）调用，也可以递归调用，就像下边这个荒谬的例子一样

```bash

    pragma solidity ^0.4.16;

    contract C {
        function g(uint a) public pure returns (uint ret) { return f(); }
        function f() internal pure returns (uint ret) { return g(7) + f(); }
    }
```

这些函数调用在 EVM 中被解释为简单的跳转。这样做的效果就是当前内存不会被清除，也就是说，通过内部调用在函数之间传递内存引用是非常有效的。

#### 外部函数调用

表达式 ``this.g(8);`` 和 ``c.g(2);`` （其中 ``c`` 是合约实例）也是有效的函数调用，但是这种情况下，函数将会通过一个消息调用来被“外部调用”，而不是直接的跳转。
请注意，不可以在构造函数中通过 this 来调用函数，因为此时真实的合约实例还没有被创建。

如果想要调用其他合约的函数，需要外部调用。对于一个外部调用，所有的函数参数都需要被复制到内存。

当调用其他合约的函数时，随函数调用发送的 Wei 和 gas 的数量可以分别由特定选项 ``.value()`` 和 ``.gas()`` 指定::

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

``payable`` 修饰符要用于修饰 ``info``，否则，`.value()` 选项将不可用。

注意，表达式 ``InfoFeed(addr)`` 进行了一个的显式类型转换，说明”我们知道给定地址的合约类型是 ``InfoFeed`` “并且这不会执行构造函数。
显式类型转换需要谨慎处理。绝对不要在一个你不清楚类型的合约上执行函数调用。

我们也可以直接使用 ``function setFeed(InfoFeed _feed) { feed = _feed; }`` 。
注意一个事实，``feed.info.value(10).gas(800)`` 只（局部地）设置了与函数调用一起发送的 Wei 值和 gas 的数量，只有最后的圆括号执行了真正的调用。

如果被调函数所在合约不存在（也就是账户中不包含代码）或者被调用合约本身抛出异常或者 gas 用完等，函数调用会抛出异常。


>	任何与其他合约的交互都会强加潜在危险，尤其是在不能预先知道合约代码的情况下。
	当前合约将控制权移交给被调用合约，而被调用合约可能做任何事。即使被调用合约从一个已知父合约继承，继承的合约也只需要有一个正确的接口就可以了。
	被调用合约的实现可以完全任意，因此会带来危险。此外，请小心万一它再调用你系统中的其他合约，或者甚至在第一次调用返回之前返回到你的调用合约。
	这意味着被调用合约可以通过它自己的函数改变调用合约的状态变量。。一个建议的函数写法是，例如，在你合约中状态变量进行各种变化后再调用外部函数，这样，你的合约就不会轻易被滥用的重入 (reentrancy) 所影响


### 具名调用和匿名函数参数


如果它们被包含在 ``{}`` 中，函数调用参数也可以按照任意顺序由名称给出，
如以下示例中所示。参数列表必须按名称与函数声明中的参数列表相符，但可以按任意顺序排列。

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

### 省略函数参数名称

未使用参数的名称（特别是返回参数）可以省略。这些参数仍然存在于堆栈中，但它们无法访问。

```bash
    pragma solidity ^0.4.16;

    contract C {
        // 省略参数名称
        function func(uint k, uint) public pure returns(uint) {
            return k;
        }
    }
```

### 通过 ``new`` 创建合约

使用关键字 ``new`` 可以创建一个新合约。待创建合约的完整代码必须事先知道，因此递归的创建依赖是不可能的。

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

如示例中所示，使用 ``.value（）`` 选项创建 ``D`` 的实例时可以转发 Ether，但是不可能限制 gas 的数量。如果创建失败（可能因为栈溢出，或没有足够的余额或其他问题），会引发异常。

### 表达式计算顺序

表达式的计算顺序不是特定的（更准确地说，表达式树中某节点的字节点间的计算顺序不是特定的，但它们的结算肯定会在节点自己的结算之前）。该规则只能保证语句按顺序执行，布尔表达式的短路执行。更多相关信息，请参阅：:ref:`order`。

### 赋值

#### 解构赋值和返回多值


Solidity 内部允许元组 (tuple) 类��，也就是一个在编译时元素数量固定的对象列表，列表中的元素可以是不同类型的对象。这些元组可以用来同时返回多个数值，也可以用它们来同时给多个新声明的变量或者既存的变量（或通常的 LValues）：

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

> 直到 0.4.24 版本，给具有更少的元素数的元组赋值都可以可能的，无论是在左边还是右边（比如在最后空出若干元素）。现在，这已经不推荐了，赋值操作的两边应该具有相同个数的组成元素。

### 数组和结构体的复杂性

赋值语义对于像数组和结构体这样的非值类型来说会有些复杂。
为状态变量 *赋值* 经常会创建一个独立副本。另一方面，对局部变量的赋值只会为基本类型（即 32 字节以内的静态类型）创建独立的副本。如果结构体或数组（包括 ``bytes`` 和 ``string``）被从状态变量分配给局部变量，局部变量将保留对原始状态变量的引用。对局部变量的第二次赋值不会修改状态变量，只会改变引用。赋值给局部变量的成员（或元素）则 *改变* 状态变量。

### 作用域和声明

变量声明后将有默认初始值，其初始值字节表示全部为零。任何类型变量的“默认值”是其对应类型的典型“零状态”。例如， ``bool`` 类型的默认值是 ``false`` 。 ``uint`` 或 ``int`` 类型的默认值是 ``0`` 。对于静态大小的数组和 ``bytes1`` 到 ``bytes32`` ，每个单独的元素将被初始化为与其类型相对应的默认值。
最后，对于动态大小的数组， ``bytes`` 和 ``string`` 类型，其默认缺省值是一个空数组或字符串。

Solidity 中的作用域规则遵循了 C99（与其他很多语言一样）：变量将会从它们被声明之后可见，直到一对 ``{ }`` 块的结束。作为一个例外，在 for 循环语句中初始化的变量，其可见性仅维持到 for 循环的结束。

那些定义在代码块之外的变量，比如函数、合约、自定义类型等等，并不会影响它们的作用域特性。这意味着你可以在实际声明状态变量的语句之前就使用它们，并且递归地调用函数。

基于以上的规则，下边的例子不会出现编译警告，因为那两个变量虽然名字一样，但却在不同的作用域里。

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

作为 C99 作用域规则的特例，请注意在下边的例子里，第一次对 ``x`` 的赋值会改变上一层中声明的变量值。如果外层声明的变量被“影子化”（就是说被在内部作用域中由一个同名变量所替代）你会得到一个警告。

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


>  在 Solidity 0.5.0 之前的版本，作用域规则都沿用了 Javascript 的规则，即一个变量可以声明在函数的任意位置，都可以使他在整个函数范围内可见。而这种规则会从 0.5.0 版本起被打破。从 0.5.0 版本开始，下面例子中的代码段会导致编译错误。


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

### 错误处理：Assert, Require, Revert and Exceptions

Solidity 使用状态恢复异常来处理错误。这种异常将撤消对当前调用（及其所有子调用）中的状态所做的所有更改，并且还向调用者标记错误。
便利函数 ``assert`` 和 ``require`` 可用于检查条件并在条件不满足时抛出异常。``assert`` 函数只能用于测试内部错误，并检查非变量。
``require`` 函数用于确认条件有效性，例如输入变量，或合约状态变量是否满足条件，或验证外部合约调用返回的值。
如果使用得当，分析工具可以评估你的合约，并标示出那些会使 ``assert`` 失败的条件和函数调用。
正常工作的代码不会导致一个 assert 语句的失败；如果这发生了，那就说明出现了一个需要你修复的 bug。


还有另外两种触发异常的方法：``revert`` 函数可以用来标记错误并恢复当前的调用。
``revert`` 调用中包含有关错误的详细信息是可能的，这个消息会被返回给调用者。已经不推荐的关键字 ``throw`` 也可以用来替代 ``revert()`` （但无法返回错误消息）。

> 从 0.4.13 版本开始，``throw`` 这个关键字被弃用，并且将来会被逐渐淘汰。

当子调用发生异常时，它们会自动“冒泡”（即重新抛出异常）。这个规则的例外是 ``send`` 和低级函数 ``call`` ， ``delegatecall`` 和 ``callcode`` --如果这些函数发生异常，将返回 false ，而不是“冒泡”。

>    作为 EVM 设计的一部分，如果被调用合约帐户不存在，则低级函数 ``call`` ，``delegatecall`` 和 ``callcode`` 将返回 success。因此如果需要使用低级函数时，必须在调用之前检查被调用合约是否存在。异常捕获还未实现

在下例中，你可以看到如何轻松使用``require``检查输入条件以及如何使用``assert``检查内部错误，注意，你可以给 ``require`` 提供一个消息字符串，而 ``assert`` 不行。

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

下列情况将会产生一个 ``assert`` 式异常：

- 如果你访问数组的索引太大或为负数（例如 ``x[i]`` 其中 ``i >= x.length`` 或 ``i < 0``）。
- 如果你访问固定长度 ``bytesN`` 的索引太大或为负数。
- 如果你用零当除数做除法或模运算（例如 ``5 / 0`` 或 ``23 % 0`` ）。
- 如果你移位负数位。
- 如果你将一个太大或负数值转换为一个枚举类型。
- 如果你调用内部函数类型的零初始化变量。
- 如果你调用 ``assert`` 的参数（表达式）最终结算为 false。

下列情况将会产生一个 ``require`` 式异常：

- 调用 ``throw`` 。
- 如果你调用 ``require`` 的参数（表达式）最终结算为 ``false`` 。
- 如果你通过消息调用调用某个函数，但该函数没有正确结束（它耗尽了 gas，没有匹配函数，或者本身抛出一个异常），上述函数不包括低级别的操作 ``call`` ， ``send`` ， ``delegatecall`` 或者 ``callcode`` 。低级操作不会抛出异常，而通过返回 ``false`` 来指示失败。
- 如果你使用 ``new`` 关键字创建合约，但合约没有正确创建（请参阅上条有关”未正确完成“的定义）。
- 如果你对不包含代码的合约执行外部函数调用。
- 如果你的合约通过一个没有 ``payable`` 修饰符的公有函数（包括构造函数和 fallback 函数）接收 Ether。
- 如果你的合约通过公有 getter 函数接收 Ether 。
- 如果 ``.transfer()`` 失败。


在内部， Solidity 对一个 ``require`` 式的异常执行回退操作（指令 ``0xfd`` ）并执行一个无效操作（指令 ``0xfe`` ）来引发 ``assert`` 式异常。在这两种情况下，都会导致 EVM 回退对状态所做的所有更改。回退的原因是不能继续安全地执行，因为没有实现预期的效果。因为我们想保留交易的原子性，所以最安全的做法是回退所有更改并使整个交易（或至少是调用）不产生效果。请注意， ``assert`` 式异常消耗了所有可用的调用 gas ，而从 Metropolis 版本起 ``require`` 式的异常不会消耗任何 gas。

下边的例子展示了如何在 revert 和 require 中使用错误字符串：

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

这里提供的字符串应该是经过 :ref:`ABI 编码 <ABI>` 之后的，因为它实际上是调用了 ``Error(string)`` 函数。在上边的例子里，``revert("Not enough Ether provided.");`` 会产生如下的十六进制错误返回值： 

- 0x08c379a0                                                         // Error(string) 的函数选择器
- 0x0000000000000000000000000000000000000000000000000000000000000020 // 数据的偏移量（32）
- 0x000000000000000000000000000000000000000000000000000000000000001a // 字符串长度（26）
- 0x4e6f7420656e6f7567682045746865722070726f76696465642e000000000000 // 字符串数据（"Not enough Ether provided." 的 ASCII 编码，26字节）


## 合约

`Solidity` 合约类似于面向对象语言中的类。合约中有用于数据持久化的状态变量，和可以修改状态变量的函数。
调用另一个合约实例的函数时，会执行一个 EVM 函数调用，这个操作会切换执行时的上下文，这样，前一个合约的状态变量就不能访问了。

### 创建合约

可以通过Simplechain交易“从外部”或从 Solidity 合约内部创建合约。

一些集成开发环境，例如 [Remix](https://remix.ethereum.org/), 通过使用一些用户界面元素使创建过程更加流畅。在Simplechain上编程创建合约最好使用 JavaScript API [web3.j](https://github.com/ethereum/web3.js)。现在，我们已经有了一个叫做 [web3.eth.Contract](https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract) 的方法能够更容易的创建合约。

创建合约时，会执行一次构造函数（与合约同名的函数）。构造函数是可选的。只允许有一个构造函数，这意味着不支持重载。

在内部，构造函数参数在合约代码之后通`ABI 编码`传递，但是如果你使用 ``web3.js``则不必关心这个问题。

如果一个合约想要创建另一个合约，那么创建者必须知晓被创建合约的源代码(和二进制代码)。
这意味着不可能循环创建依赖项。

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

### 可见性和 getter 函数

由于 Solidity 有两种函数调用（内部调用不会产生实际的 EVM 调用或称为“消息调用”，而外部调用则会产生一个 EVM 调用），函数和状态变量有四种可见性类型。函数可以指定为 ``external`` ，``public`` ，``internal`` 或者 ``private``，默认情况下函数类型为 ``public``。
对于状态变量，不能设置为 ``external`` ，默认是 ``internal`` 。

- ``external`` ：外部函数作为合约接口的一部分，意味着我们可以从其他合约和交易中调用。
    一个外部函数 ``f`` 不能从内部调用（即 ``f`` 不起作用，但 ``this.f()`` 可以）。
    当收到大量数据的时候，外部函数有时候会更有效率。
- ``public`` ：public 函数是合约接口的一部分，可以在内部或通过消息调用。对于公共状态变量，会自动生成一个 getter 函数（见下面）。
- ``internal`` ：这些函数和状态变量只能是内部访问（即从当前合约内部或从它派生的合约访问），不使用 ``this`` 调用。
``private`` ：private 函数和状态变量仅在当前定义它们的合约中使用，并且不能被派生合约使用。

> 合约中的所有内容对外部观察者都是可见的。设置一些 ``private`` 类型只能阻止其他合约访问和修改这些信息，但是对于区块链外的整个世界它仍然是可见的。

可见性标识符的定义位置，对于状态变量来说是在类型后面，对于函数是在参数列表和返回关键字中间。

```bash
pragma solidity ^0.4.16;

contract C {
    function f(uint a) private pure returns (uint b) { return a + 1; }
    function setData(uint a) internal { data = a; }
    uint public data;
}
```

在下面的例子中，``D`` 可以调用 ``c.getData（）`` 来获取状态存储中 ``data`` 的值，但不能调用 ``f`` 。
合约 ``E`` 继承自 ``C`` ，因此可以调用 ``compute``。

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

#### Getter 函数

编译器自动为所有 **public** 状态变量创建 getter 函数。对于下面给出的合约，编译器会生成一个名为 ``data`` 的函数，
该函数不会接收任何参数并返回一个 ``uint`` ，即状态变量 ``data`` 的值。可以在声明时完成状态变量的初始化。

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

getter 函数具有外部可见性。如果在内部访问 getter（即没有 ``this.`` ），它被认为一个状态变量。
如果它是外部访问的（即用 ``this.`` ），它被认为为一个函数。

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

下一个例子稍微复杂一些：

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

这将会生成以下形式的函数 ::

```bash
function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
    a = data[arg1][arg2][arg3].a;
    b = data[arg1][arg2][arg3].b;
}
```

请注意，因为没有好的方法来提供映射的键，所以结构中的映射被省略。

#### 函数 |modifier|

使用 |modifier| 可以轻松改变函数的行为。 例如，它们可以在执行函数之前自动检查某个条件。
|modifier| 是合约的可继承属性，
并可能被派生合约覆盖。

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

如果同一个函数有多个 `modifier`，它们之间以空格隔开，`modifier`会依次检查执行。

> 在早期的 Solidity 版本中，有 |modifier| 的函数，``return`` 语句的行为表现不同。

`modifier` 或函数体中显式的 return 语句仅仅跳出当前的 `modifier` 和函数体。
返回变量会被赋值，但整个执行逻辑会从前一个 |modifier| 中的定义的 “_” 之后继续执行。

`modifier`的参数可以是任意表达式，在此上下文中，所有在函数中可见的符号，在 `modifier` 中均可见。在 `modifier` 中引入的符号在函数中不可见（可能被重载改变）。

### Constant 状态变量

状态变量可以被声明为 ``constant``。在这种情况下，只能使用那些在编译时有确定值的表达式来给它们赋值。任何通过访问 storage，区块链数据（例如 ``now``, ``this.balance`` 或者 ``block.number``）或执行数据（ ``msg.gas`` ）或对外部合约的调用来给它们赋值都是不允许的。在内存分配上有边界效应（`side-effect`）的表达式是允许的，但对其他内存对象产生边界效应的表达式则不行。内建（built-in）函数 ``keccak256``，``sha256``，``ripemd160``，``ecrecover``，``addmod`` 和 ``mulmod`` 是允许的（即使他们确实会调用外部合约）。

允许带有边界效应的内存分配器的原因是这将允许构建复杂的对象，比如查找表（lookup-table）。
此功能尚未完全可用。编译器不会为这些变量预留存储，它们的每次出现都会被替换为相应的常量表达式（这将可能被优化器计算为实际的某个值）。不是所有类型的状态变量都支持用 constant 来修饰，当前支持的仅有值类型和字符串。

```bash
    pragma solidity ^0.4.0;

    contract C {
        uint constant x = 32**22 + 8;
        string constant text = "abc";
        bytes32 constant myHash = keccak256("abc");
    }
```

### 函数

#### View 函数

可以将函数声明为 ``view`` 类型，这种情况下要保证不修改状态。

下面的语句被认为是修改状态：

- 修改状态变量。
- `产生事件`。
- `创建其它合约`。
- 使用`selfdestruct`。
- 通过调用发送以太币。
- 调用任何没有标记为 ``view`` 或者 ``pure`` 的函数。
- 使用低级调用。
- 使用包含特定操作码的内联汇编。

```bash
    pragma solidity ^0.4.16;

    contract C {
        function f(uint a, uint b) public view returns (uint) {
            return a * (b + 42) + now;
        }
    }
```

>  ``constant`` 是 ``view`` 的别名。

>  Getter 方法被标记为 ``view``。

>  编译器没有强制 ``view`` 方法不能修改状态。

#### Pure 函数

函数可以声明为 ``pure`` ，在这种情况下，承诺不读取或修改状态。除了上面解释的状态修改语句列表之外，以下被认为是从状态中读取：

- 读取状态变量。
- 访问 ``this.balance`` 或者 ``<address>.balance``。
- 访问 ``block``，``tx``， ``msg`` 中任意成员 （除 ``msg.sig`` 和 ``msg.data`` 之外）。
- 调用任何未标记为 ``pure`` 的函数。
- 使用包含某些操作码的内联汇编。

```bash
    pragma solidity ^0.4.16;

    contract C {
        function f(uint a, uint b) public pure returns (uint) {
            return a * (b + 42);
        }
    }
```

>  编译器没有强制 ``pure`` 方法不能读取状态。

#### Fallback 函数

合约可以有一个未命名的函数。这个函数不能有参数也不能有返回值。如果在一个到合约的调用中，没有其他函数与给定的函数标识符匹配（或没有提供调用数据），那么这个函数（fallback 函数）会被执行。

除此之外，每当合约收到以太币（没有任何数据），这个函数就会执行。此外，为了接收以太币，fallback 函数必须标记为 ``payable``。如果不存在这样的函数，则合约不能通过常规交易接收以太币。

在这样的上下文中，通常只有很少的 gas 可以用来完成这个函数调用（准确地说，是 2300 gas），所以使 fallback 函数的调用尽量廉价很重要。请注意，调用 fallback 函数的交易（而不是内部调用）所需的 gas 要高得多，因为每次交易都会额外收取 21000 gas 或更多的费用，用于签名检查等操作。

具体来说，以下操作会消耗比 fallback 函数更多的 gas：

- 写入存储
- 创建合约
- 调用消耗大量 gas 的外部函数
- 发送以太币

请确保您在部署合约之前彻底测试您的 fallback 函数，以确保执行成本低于 2300 个 gas。

>  即使 fallback 函数不能有参数，仍然可以使用 ``msg.data`` 来获取随调用提供的任何有效数据。

>  一个没有定义 fallback 函数的合约，直接接收以太币（没有函数调用，即使用 ``send`` 或 ``transfer``）会抛出一个异常，并返还以太币（在 Solidity v0.4.0 之前行为会有所不同）。所以如果你想让你的合约接收以太币，必须实现 fallback 函数。

>  一个没有 payable fallback 函数的合约，可以作为 `coinbase transaction` （又名 `miner block reward` ）的接收者或者作为 ``selfdestruct`` 的目标来接收以太币。

>  一个合约不能对这种以太币转移做出反应，因此也不能拒绝它们。这是 EVM 在设计时就决定好的，而且 Solidity 无法绕过这个问题。

>  这也意味着 ``this.balance`` 可以高于合约中实现的一些手工记帐的总和（即在 fallback 函数中更新的累加器）。

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

#### 函数重载

合约可以具有多个不同参数的同名函数。这也适用于继承函数。以下示例展示了合约 ``A`` 中的重载函数 ``f``。

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

重载函数也存在于外部接口中。如果两个外部可见函数仅区别于 Solidity 内的类型而不是它们的外部类型则会导致错误。

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

以上两个 ``f`` 函数重载都接受了 ABI 的地址类型，虽然它们在 Solidity 中被认为是不同的。

#### 重载解析和参数匹配

通过将当前范围内的函数声明与函数调用中提供的参数相匹配，可以选择重载函数。
如果所有参数都可以隐式地转换为预期类型，则选择函数作为重载候选项。如果一个候选都没有，解析失败。

>  返回参数不作为重载解析的依据。

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

调用  ``f(50)`` 会导致类型错误，因为 ``50`` 既可以被隐式转换为 ``uint8`` 也可以被隐式转换为 ``uint256``。另一方面，调用 ``f(256)`` 则会解析为 ``f(uint256)`` 重载，因为 ``256`` 不能隐式转换为 ``uint8``。

### 事件

事件允许我们方便地使用 EVM 的日志基础设施。我们可以在 dapp 的用户界面中监听事件，EVM 的日志机制可以反过来“调用”用来监听事件的 Javascript 回调函数。

事件在合约中可被继承。当他们被调用时，会使参数被存储到交易的日志中 —— 一种区块链中的特殊数据结构。这些日志与地址相关联，被并入区块链中，只要区块可以访问就一直存在（在 Frontier 和 Homestead 版本中会被永久保存，在 Serenity 版本中可能会改动)。日志和事件在合约内不可直接被访问（甚至是创建日志的合约也不能访问）。

对日志的 SPV（Simplified Payment Verification）证明是可能的，如果一个外部实体提供了一个带有这种证明的合约，它可以检查日志是否真实存在于区块链中。但需要留意的是，由于合约中仅能访问最近的 256 个区块哈希，所以还需要提供区块头信息。

最多三个参数可以接收 ``indexed`` 属性，从而使它们可以被搜索：在用户界面上可以使用 indexed 参数的特定值来进行过滤。

如果数组（包括 ``string`` 和 ``bytes``）类型被标记为索引项，则它们的 keccak-256 哈希值会被作为 topic 保存。除非你用 ``anonymous`` 说明符声明事件，否则事件签名的哈希值是 topic 之一。同时也意味着对于匿名事件无法通过名字来过滤。所有非索引参数都将存储在日志的数据部分中。


>  索引参数本身不会被保存。你只能搜索它们的值（来确定相应的日志数据是否存在），而不能获取它们的值本身。

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

使用 JavaScript API 调用事件的用法如下：

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
#### 日志的底层接口

通过函数 ``log0``，``log1``， ``log2``， ``log3`` 和 ``log4`` 可以访问日志机制的底层接口。``logi``  接受 ``i + 1`` 个 ``bytes32`` 类型的参数。其中第一个参数会被用来做为日志的数据部分，其它的会做为 topic。上面的事件调用可以以相同的方式执行。

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

其中的长十六进制数的计算方法是 ``keccak256("Deposit(address,hash256,uint256)")``，即事件的签名。

#### 其它学习事件机制的资源
 
- [Javascript 文档](https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events)
- [事件使用例程](https://github.com/debris/smart-exchange/blob/master/lib/contracts/SmartExchange.sol)
- [如何在 js 中访问它们](https://github.com/debris/smart-exchange/blob/master/lib/exchange_transactions.js)

### 继承

通过复制包括多态的代码，Solidity 支持多重继承。所有的函数调用都是虚拟的，这意味着最远的派生函数会被调用，除非明确给出合约名称。当一个合约从多个合约继承时，在区块链上只有一个合约被创建，所有基类合约的代码被复制到创建的合约中。

总的来说，Solidity 的继承系统与 [Python的继承系统](https://docs.python.org/3/tutorial/classes.html#inheritance)，非常相似，特别是多重继承方面。

下面的例子进行了详细的说明。

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

注意，在上边的代码中，我们调用 ``mortal.kill()`` 来“转发”销毁请求。这样做法是有问题的，在下面的例子中可以看到::

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

调用 ``Final.kill()`` 时会调用最远的派生重载函数 ``Base2.kill``，但是会绕过 ``Base1.kill``，主要是因为它甚至都不知道 ``Base1`` 的存在。解决这个问题的方法是使用 ``super``:

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

如果 ``Base2`` 调用 ``super`` 的函数，它不会简单在其基类合约上调用该函数。
相反，它在最终的继承关系图谱的下一个基类合约中调用这个函数，所以它会调用 ``Base1.kill()``（注意最终的继承序列是——从最远派生合约开始：Final, Base2, Base1, mortal, ownerd）。在类中使用 super 调用的实际函数在当前类的上下文中是未知的，尽管它的类型是已知的。
这与普通的虚拟方法查找类似。

### 基类构造函数的参数

派生合约需要提供基类构造函数需要的所有参数。这可以通过两种方式来完成::

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

一种方法直接在继承列表中调用基类构造函数（``is Base(7)``）。另一种方法是像 |modifier| 使用方法一样，作为派生合约构造函数定义头的一部分，（``Base(_y * _y)``)。如果构造函数参数是常量并且定义或描述了合约的行为，使用第一种方法比较方便。如果基类构造函数的参数依赖于派生合约，那么必须使用第二种方法。如果像这个简单的例子一样，两个地方都用到了，优先使用 |modifier| 风格的参数。

### 多重继承与线性化

编程语言实现多重继承需要解决几个问题。
一个问题是[钻石问题](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem)
Solidity 借鉴了 Python 的方式并且使用 [C3 线性化](https://en.wikipedia.org/wiki/C3_linearization) 强制一个由基类构成的 DAG（有向无环图）保持一个特定的顺序。
这最终反映为我们所希望的唯一化的结果，但也使某些继承方式变为无效。尤其是，基类在 ``is`` 后面的顺序很重要。在下面的代码中，Solidity 会给出“ Linearization of inheritance graph impossible ”这样的错误。

```bash

    // 以下代码编译出错

    pragma solidity ^0.4.0;

    contract X {}
    contract A is X {}
    contract C is A, X {}
```

代码编译出错的原因是 ``C`` 要求 ``X`` 重写 ``A`` （因为定义的顺序是 ``A, X`` ），
但是 ``A`` 本身要求重写 ``X``，无法解决这种冲突。

可以通过一个简单的规则来记忆：以从“最接近的基类”（most base-like）到“最远的继承”（most derived）的顺序来指定所有的基类。

#### 继承有相同名字的不同类型成员

当继承导致一个合约具有相同名字的函数和 |modifier| 时，这会被认为是一个错误。
当事件和 |modifier| 同名，或者函数和事件同名时，同样会被认为是一个错误。
有一种例外情况，状态变量的 getter 可以覆盖一个 public 函数。

### 抽象合约

合约函数可以缺少实现，如下例所示（请注意函数声明头由 ``;`` 结尾）:

```
    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }
```

这些合约无法成功编译（即使它们除了未实现的函数还包含其他已经实现了的函数），但他们可以用作基类合约::

```bash
    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public returns (bytes32) { return "miaow"; }
    }
```

如果合约继承自抽象合约，并且没有通过重写来实现所有未实现的函数，那么它本身就是抽象的。

### 接口


接口类似于抽象合约，但是它们不能实现任何函数。还有进一步的限制：

- 无法继承其他合约或接口。
- 无法定义构造函数。
- 无法定义变量。
- 无法定义结构体
- 无法定义枚举。

将来可能会解除这里的某些限制。

接口基本上仅限于合约 ABI 可以表示的内容，并且 ABI 和接口之间的转换应该不会丢失任何信息。

接口由它们自己的关键字表示：

```bash

    pragma solidity ^0.4.11;

    interface Token {
        function transfer(address recipient, uint amount) public;
    }
```

就像继承其他合约一样，合约可以继承接口。

### 库

库与合约类似，它们只需要在特定的地址部署一次，并且它们的代码可以通过 EVM 的 ``DELEGATECALL``(Homestead 之前使用 ``CALLCODE`` 关键字)特性进行重用。这意味着如果库函数被调用，它的代码在调用合约的上下文中执行，即 ``this`` 指向调用合约，特别是可以访问调用合约的存储。因为每个库都是一段独立的代码，所以它仅能访问调用合约明确提供的状态变量（否则它就无法通过名字访问这些变量）。因为我们假定库是无状态的，所以如果它们不修改状态（也就是说，如果它们是 ``view`` 或者 ``pure`` 函数），库函数仅可以通过直接调用来使用（即不使用 ``DELEGATECALL`` 关键字），特别是，除非能规避 Solidity 的类型系统，否则是不可能销毁任何库的。

库可以看作是使用他们的合约的隐式的基类合约。虽然它们在继承关系中不会显式可见，但调用库函数与调用显式的基类合约十分类似（如果 ``L`` 是库的话，可以使用 ``L.f()`` 调用库函数）。此外，就像库是基类合约一样，对所有使用库的合约，库的 ``internal`` 函数都是可见的。
当然，需要使用内部调用约定来调用内部函数，这意味着所有内部类型，内存类型都是通过引用而不是复制来传递。为了在 EVM 中实现这些，内部库函数的代码和从其中调用的所有函数都在编译阶段被拉取到调用合约中，然后使用一个 ``JUMP`` 调用来代替 ``DELEGATECALL``。

下面的示例说明如何使用库（但也请务必看看 :ref:`using for <using-for>` 有一个实现 set 更好的例子）。

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

当然，你不必按照这种方式去使用库：它们也可以在不定义结构数据类型的情况下使用。函数也不需要任何存储引用参数，库可以出现在任何位置并且可以有多个存储引用参数。

调用 ``Set.contains``，``Set.insert`` 和 ``Set.remove`` 都被编译为外部调用（ ``DELEGATECALL`` ）。如果使用库，请注意实际执行的是外部函数调用。``msg.sender``， ``msg.value`` 和 ``this`` 在调用中将保留它们的值，（在 Homestead 之前，因为使用了 ``CALLCODE``，改变了 ``msg.sender`` 和 ``msg.value``)。

以下示例展示了如何在库中使用内存类型和内部函数来实现自定义类型，而无需支付外部函数调用的开销：

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

由于编译器无法知道库的部署位置，我们需要通过链接器将这些地址填入最终的字节码中
（请参阅 :ref:`commandline-compiler` 以了解如何使用命令行编译器来链接字节码）。
如果这些地址没有作为参数传递给编译器，编译后的十六进制代码将包含 ``__Set______`` 形式的占位符（其中 ``Set`` 是库的名称）。可以手动填写地址来将那 40 个字符替换为库合约地址的十六进制编码。

与合约相比，库的限制：

- 没有状态变量
- 不能够继承或被继承
- 不能接收以太币

（将来有可能会解除这些限制）

#### 库的调用保护

如果库的代码是通过 ``CALL`` 来执行，而不是 ``DELEGATECALL`` 或者 ``CALLCODE`` 那么执行的结果会被回退，除非是对 ``view`` 或者 ``pure`` 函数的调用。EVM 没有为合约提供检测是否使用 ``CALL`` 的直接方式，但是合约可以使用 ``ADDRESS`` 操作码找出正在运行的“位置”。生成的代码通过比较这个地址和构造时的地址来确定调用模式。

更具体地说，库的运行时代码总是从一个 push 指令开始，它在编译时是 20 字节的零。当部署代码运行时，这个常数
被内存中的当前地址替换，修改后的代码存储在合约中。在运行时，这导致部署时地址是第一个被 push 到堆栈上的常数，
对于任何 non-view 和 non-pure 函数，调度器代码都将对比当前地址与这个常数是否一致。

### Using For

指令 ``using A for B;`` 可用于附加库函数（从库 ``A``）到任何类型（``B``）。
这些函数将接收到调用它们的对象作为它们的第一个参数（像 Python 的 ``self`` 变量）。
``using A for *;`` 的效果是，库 ``A`` 中的函数被附加在任意的类型上。在这两种情况下，所有函数都会被附加一个参数，即使它们的第一个参数类型与对象的类型不匹配。
函数调用和重载解析时才会做类型检查。``using A for B;`` 指令仅在当前作用域有效，目前仅限于在当前合约中，后续可能提升到全局范围。通过引入一个模块，不需要再添加代码就可以使用包括库函数在内的数据类型。

让我们用这种方式将`libraries`中的 set 例子重写::

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

也可以像这样扩展基本类型:

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

注意，所有库调用都是实际的 EVM 函数调用。这意味着如果传递内存或值类型，都将产生一个副本，即使是 ``self`` 变量。]使用存储引用变量是唯一不会发生拷贝的情况。 

## Solidity汇编

Solidity 定义了一种汇编语言，在没有 Solidity 的情况下也可以使用。这种汇编语言也可以嵌入到 Solidity 源代码中当作“内联汇编”使用。
我们从如何使用内联汇编开始，介绍它如何区别于独立汇编语言，然后详细讲述这种汇编语言。

### 内联汇编

为了实现更细粒度的控制，尤其是为了通过编写库来增强语言，可以利用接近虚拟机的语言将内联汇编与 Solidity 语句结合在一起使用。
由于 EVM 是基于栈的虚拟机，因此通常很难准确地定位栈内插槽（存储位置）的地址，并为操作码提供正确的栈内位置来获取参数。
Solidity 的内联汇编试图通过提供以下特性来解决这个问题以及手工编写汇编代码时可能出现的问题：

* 函数风格操作码： ``mul(1, add(2, 3))`` 而不是 ``push1 3 push1 2 add push1 1 mul``
* 汇编局部变量： ``let x := add(2, 3)  let y := mload(0x40)  x := add(x, y)``
* 可以访问外部变量： ``function f(uint x) public { assembly { x := sub(x, 1) } }``
* 标签： ``let x := 10  repeat: x := sub(x, 1) jumpi(repeat, eq(x, 0))``
* 循环： ``for { let i := 0 } lt(i, x) { i := add(i, 1) } { y := mul(2, y) }``
* if 语句： ``if slt(x, 0) { x := sub(0, x) }``
* switch 语句： ``switch x case 0 { y := mul(x, 2) } default { y := 0 }``
* 函数调用： ``function f(x) -> y { switch x case 0 { y := 1 } default { y := mul(x, f(sub(x, 1))) }   }``

现在我们详细讲解内联汇编语言。

> 内联汇编是一种在底层访问Simplechain虚拟机的语言。这抛弃了很多 Solidity 提供的重要安全特性。

> TODO：写出在内联汇编中作用域规则的细微差别，以及在使用库合约的内部函数时产生的复杂性。此外，还要编写有关编译器定义的符号。

例子
-------

下面例子展示了一个库合约的代码，它可以取得另一个合约的代码，并将其加载到一个 ``bytes`` 变量中。
这对于“常规 Solidity”来说是根本不可能的，汇编库合约则可以通过这种方式来增强语言特性。

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

在优化器无法生成高效代码的情况下，内联汇编也可能更有好处。请注意，由于编译器无法对汇编语句进行相关的检查，所以编写汇编代码肯定更加困难；
因此只有在处理一些相对复杂的问题时才需要使用它，并且你需要明确知道自己要做什么。

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


#### 语法

和 Solidity 一样，Assembly 也会解析注释、文字和标识符，所以你可以使用通常的 ``//`` 和 ``/* */`` 来进行注释。
内联汇编程序由 ``assembly { ... }`` 来标记，在这些大括号内可以使用以下内容（更多详细信息请参阅后面部分）。

 - 字面常数，也就是 ``0x123``、``42`` 或 ``"abc"`` （不超过 32 个字符的字符串）
 - 操作码（在“instruction style”内），比如 ``mload sload dup1 sstore``，操作码列表请看后面
 - 函数风格操作码，比如 ``add(1，mlod(0))``
 - 标签，比如 ``name:``
 - 变量声明，比如 ``let x := 7``、``let x := add(y, 3)`` 或者 ``let x`` （初始值将被置为 empty(0)）
 - 标识符（标签或者汇编局部变量以及用作内联汇编时的外部变量），比如 ``jump(name)``、``3 x add``
 - 赋值（在“instruction style”内），比如 ``3 =: x``
 - 函数风格赋值，比如 ``x := add(y，3)``
 - 一些控制局部变量作用域的语句块，比如 ``{let x := 3 { let y := add(x，1) }}``

#### 操作码

参考操作码：

如果一个操作码需要参数（总是来自堆栈顶部），它们会在括号中给出。请注意：参数顺序可以看作是在非函数风格中逆序（下面会解释）。标有 ``-`` 的操作码不会向栈中压入（push）数据，标有 ``*`` 的操作码有特殊操作，而所有其他操作码都只会将一个数据压入（push）栈中。
用 ``F``、``H``、``B`` 或 ``C`` 标记的操作码代表它们从 Frontier、Homestead、Byzantium 或 Constantinople 开始被引入。Constantinople 目前仍在计划中，所以标记为 ``C`` 的指令目前都会导致一个非法指令异常。在下表中，``mem[a...b)`` 表示从位置 ``a`` 开始至（不包括）位置 ``b`` 的内存字节数，``storage[p]`` 表示位置 ``p`` 处的存储内容。
``pushi`` 和 ``jumpdest`` 这两个操作码不能直接用。

在语法表中，操作码是作为预定义标识符提供的。

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

#### 字面常量

你可以直接键入十进制或十六进制符号来作为整型常量使用，这会自动生成相应的 ``PUSHi`` 指令。
下面的代码将计算 2 加 3（等于 5），然后计算其与字符串 “abc” 的按位与。字符串在存储时为左对齐，且长度不能超过 32 字节。

```bash
assembly { 2 3 add "abc" and }
```

#### 函数风格

你可以像使用字节码那样在操作码之后键入操作码。例如，把 ``3`` 与内存位置 ``0x80`` 处的数据相加就是

```bash
3 0x80 mload add 0x80 mstore
```

由于通常很难看到某些操作码的实际参数是什么，所以 Solidity 内联汇编还提供了一种“函数风格”表示法，同样功能的代码可以写做

```bash
mstore(0x80, add(mload(0x80), 3))
```

函数风格表达式内不能使用指令风格的写法，即 ``1 2 mstore(0x80, add)`` 是无效汇编语句，
它必须写成 ``mstore(0x80, add(2, 1))`` 这种形式。对于不带参数的操作码，括号可以省略。

注意，在函数风格写法中参数的顺序与指令风格相反。如果使用函数风格写法，第一个参数将会位于栈顶。

#### 访问外部变量和函数

通过简单使用它们名称就可以访问 Solidity 变量和其他标识符。对于内存变量，这会将地址而不是值压入栈中。
存储变量是不同的，因为存储变量的值可能不占用完整的存储槽，因此其“地址”由存储槽和槽内的字节偏移量组成。
为了获取变量 ``x`` 所使用的存储槽，你可以使用 ``x_slot``，并用的 ``x_offset`` 获取其字节偏移量。

在赋值语句中（见下文），我们甚至可以使用 Solidity 局部变量来赋值。

对于内联汇编而言的外部函数也可以被访问：汇编会将它们的入口标签（带有虚拟函数解析）压入栈中。Solidity 中的调用语义为：

 - 调用者压入 ``return label``、``arg1``、``arg2``、...、``argn``
 - 被调用方返回 ``ret1``、``ret2``、...、``retm``

这个特性使用起来还是有点麻烦，因为在调用过程中堆栈偏移量发生了根本变化，因此对局部变量的引用将会出错。

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


> 如果你访问一个实际数据位数小于 256 位的数据类型（比如 ``uint64``、``address``、``bytes16`` 或 ``byte``），不要对这种类型经过编码后未使用的数据位上的数值做任何假设。尤其是不要假设它们肯定为 0。安全起见，在某个上下文中使用这种数据之前，请一定先将其数据清空为 0，这非常重要：``uint32 x = f(); assembly { x := and(x, 0xffffffff) /* now use x */ }``要清空有符号类型，你可以使用 ``signextend`` 操作码。

#### 标签

>    标签已经不推荐使用。请使用函数、循环、if 或 switch 语句。

EVM 汇编的另一个问题是 jump 和 jumpi 函数使用绝对地址，这些绝对地址很容易改变。
Solidity 内联汇编提供了标签，以便更容易地使用 jump。注意，标签具有底层特征，使用循环、if 和 switch 指令（参见下文）而不使用标签也能写出高效汇编代码。
以下代码用来计算斐波那契数列中的一个元素。

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

请注意：只有汇编程序知道当前栈高度时，才能自动访问堆栈变量。如果 jump 源和目标的栈高度不同，访问将失败。
虽然我们可以这么使用 jump，但在这种情况下，你不应该去访问任何栈里的变量（即使是汇编变量）。

此外，栈高度分析器还可以通过操作码（而不是根据控制流）检查代码操作码，因此在下面的情况下，汇编程序对标签 ``two`` 处的堆栈高度会产生错误的印象：

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

#### 汇编局部变量声明

你可以使用 ``let`` 关键字来声明只在内联汇编中可见的变量，实际上只在当前的 ``{...}`` 块中可见。
下面发生的事情应该是：``let`` 指令将创建一个为变量保留的新数据槽，并在到达块末尾时自动删除。
你需要为变量提供一个初始值，它可以只是 ``0``，但它也可以是一个复杂的函数风格表达式。

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

#### 赋值


可以给汇编局部变量和函数局部变量赋值。请注意：当给指向内存或存储的变量赋值时，你只是更改指针而不是数据。

有两种赋值方式：函数风格和指令风格。对于函数风格赋值（``变量 := 值``），你需要在函数风格表达式中提供一个值，它恰好可以产生一个栈里的值；
对于指令风格赋值（``=: 变量``），则仅从栈顶部获取数据。对于这两种方式，冒号均指向变量名称。赋值则是通过用新值替换栈中的变量值来实现的。

```bash

    {
        let v := 0 // 作为变量声明的函数风格赋值
        let g := add(v, 2)
        sload(10)
        =: v // 指令风格的赋值，将 sload(10) 的结果赋给 v
    }
```

> 指令风格的赋值已经不推荐。

#### If


if 语句可以用于有条件地执行代码，且没有“else”部分；如果需要多种选择，你可以考虑使用“switch”（见下文）。

```bash

    {
        if eq(value, 0) { revert(0, 0) }
    }
```

代码主体的花括号是必需的。

#### Switch


作为“if/else”的非常初级的版本，你可以使用 switch 语句。它计算表达式的值并与几个常量进行比较。选出与匹配常数对应的分支。
与某些编程语言容易出错的情况不同，控制流不会从一种情形继续执行到下一种情形。我们可以设定一个 fallback 或称为 ``default`` 的默认情况。

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

Case 列表里面不需要大括号，但 case 主体需要。

#### 循环

汇编语言支持一个简单的 for-style 循环。For-style 循环有一个头，它包含初始化部分、条件和迭代后处理部分。
条件必须是函数风格表达式，而另外两个部分都是语句块。如果起始部分声明了某个变量，这些变量的作用域将扩展到循环体中（包括条件和迭代后处理部分）。

下面例子是计算某个内存区域中的数值总和。

```bash

    {
        let x := 0
        for { let i := 0 } lt(i, 0x100) { i := add(i, 0x20) } {
            x := add(x, mload(i))
        }
    }
```

For 循环也可以写成像 while 循环一样：只需将初始化部分和迭代后处理两部分留空。

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

#### 函数

汇编语言允许定义底层函数。底层函数需要从栈中取得它们的参数（和返回 PC），并将结果放入栈中。调用函数的方式与执行函数风格操作码相同。函数可以在任何地方定义，并且在声明它们的语句块中可见。函数内部不能访问在函数之外定义的局部变量。这里没有严格的 ``return`` 语句。如果调用会返回多个值的函数，则必须使用 ``a，b：= f(x)`` 或 ``let a，b：= f(x)`` 的方式把它们赋值到一个元组。

下面例子通过平方和乘法实现了幂运算函数。

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

#### 注意事项

内联汇编语言可能具有相当高级的外观，但实际上它是非常低级的编程语言。函数调用、循环、if 语句和 switch 语句通过简单的重写规则进行转换，
然后，汇编程序为你做的唯一事情就是重新组织函数风格操作码、管理 jump 标签、计算访问变量的栈高度，还有在到达语句块末尾时删除局部汇编变量的栈数据。
特别是对于最后两种情况，汇编程序仅会按照代码的顺序计算栈的高度，而不一定遵循控制流程；了解这一点非常重要。此外，swap 等操作只会交换栈内的数据，而不是变量位置。

#### Solidity 惯例

与 EVM 汇编语言相比，Solidity 能够识别小于 256 位的类型，例如 ``uint24``。为了提高效率，大多数算术运算只将它们视为 256 位数字，仅在必要时才清除未使用的数据位，即在将它们写入内存或执行比较之前才会这么做。这意味着，如果从内联汇编中访问这样的变量，你必须先手工清除那些未使用的数据位。

Solidity 以一种非常简单的方式管理内存：在 ``0x40`` 的位置有一个“空闲内存指针”。如果你打算分配内存，只需从此处开始使用内存，然后相应地更新指针即可。内存的开头 64 字节可以用来作为临时分配的“暂存空间”。“空闲内存指针”之后的 32 字节位置（即从 ``0x60`` 开始的位置）将永远为 0，可以用来初始化空的动态内存数组。

在 Solidity 中，内存数组的元素总是占用 32 个字节的倍数（是的，甚至对于 ``byte[]`` 都是这样，只有 ``bytes`` 和 ``string`` 不是这样）。多维内存数组就是指向内存数组的指针。动态数组的长度存储在数组的第一个槽中，其后才是数组元素。

>   静态内存数组没有长度字段，但很快就会增加，这是为了可以更好地进行静态数组和动态数组之间的转换，所以请不要依赖这点。

### 独立汇编

以上内联汇编描述的汇编语言也可以单独使用，实际上，计划是将其用作 Solidity 编译器的中间语言。在这种意义下，它试图实现以下几个目标：

1、即使代码是由 Solidity 的编译器生成的，用它编写的程序应该也是可读的。
2、从汇编到字节码的翻译应该尽可能少地包含“意外”。
3、控制流应该易于检测，以帮助进行形式化验证和优化。

为了实现第一个和最后一个目标，汇编提供了高级结构：如 ``for`` 循环、``if`` 语句、``switch`` 语句和函数调用。
应该可以编写不使用明确的 ``SWAP``、``DUP``、``JUMP`` 和 ``JUMPI`` 语句的汇编程序，因为前两个混淆了数据流，而最后两个混淆了控制流。
此外，形式为 ``mul(add(x, y), 7)`` 的函数风格语句优于如 ``7 y x add mul`` 的指令风格语句，因为在第一种形式中更容易查看哪个操作数用于哪个操作码。

第二个目标是通过采用一种非常规则的方式来将高级高级指令结构便以为字节码。
汇编程序执行的唯一非局部操作是用户自定义标识符（函数、变量、...）的名称查找，它遵循非常简单和固定的作用域规则并从栈中清除局部变量。

作用域：在其中声明的标识符（标签、变量、函数、汇编）仅在声明的语句块中可见（包括当前语句块中的嵌套语句块）。
即使它们在作用范围内，越过函数边界访问局部变量也是非法的。阴影化是禁止的。在声明之前不能访问局部变量，但标签、函数和汇编是可以的。
汇编是特殊的语句块，例如用于返回运行时代码或创建合约等。在子汇编外部的汇编语句块中声明的标示符在子汇编中全都不可见。

如果控制流经过块尾部，则会插入与在当前语句块中声明的局部变量数量相匹配的 pop 指令。无论何时引用局部变量，代码生成器都需要知道在当前栈的相对位置，
因此，需要跟踪当前所谓的栈高度。由于所有在语句块内声明的局部变量都会在语句块结束时被清楚，所以语句块前后的栈高度应该相同。如果情况并非如此，则会发出警告。

使用 ``switch``、``for`` 和函数应该可以编写复杂的代码，而无需手工调用 ``jump`` 或 ``jumpi``。这将允许改进的形式化验证和优化更简单地分析控制流程。

此外，如果允许手动跳转，计算栈高度将会更加复杂。栈中所有局部变量的位置都需要明确知晓，否则在语句块结束时就无法自动获得局部变量的引用从而正确地清除它们。

例子：

我们将参考一个从 Solidity 到汇编指令的实例。考虑以下 Solidity 程序的运行时字节码::

    pragma solidity ^0.4.16;

    contract C {
      function f(uint x) public pure returns (uint y) {
        y = 1;
        for (uint i = 0; i < x; i++)
          y = 2 * y;
      }
    }

将会生成如下汇编指令::

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

汇编语法
-----------------

解析器任务如下：

- 将字节流转换为符号流，丢弃 C ++ 风格的注释（对源代码引用存在特殊注释，我们这里不解释它）。
- 根据下面的语法，将符号流转换为 AST。
- 注册语句块中定义的标识符（注释到 AST 节点），并注明变量从哪个地方开始可以访问。

汇编词法分析器遵循由 Solidity 自己定义的规则。

空格用于分隔所有符号，它由空格字符、制表符和换行符组成。注释格式是常规的 JavaScript/C++ 风格，并被解释为空格。

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

