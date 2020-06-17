On Simplechain, you can run the contract deployed on Simplechain to complete the operations that require consensus. Simplechain virtual machine is the executor of intelligent contract code. Therefore, when the smart contract is compiled into a binary file, it is deployed on Simplechain. The user calls the smart contract interface to trigger the execution of the smart contract. EVM executes the code of the smart contract to modify the data (status) on the current blockchain. The modified data will be agreed to ensure consistency.

## SVMC

EVM can be stripped from Simplechain to form an independent module. The interaction between EVM and nodes abstracts the SVMC interface standard. Through the SVMC interface standard, nodes can connect to a variety of virtual machines, not limited to traditional solidity-based virtual machines.

The traditional solidity virtual machine is called interpreter in Simplechain. The implementation of the interpreter is mainly explained in the following section.

## SVMC interface

SVMC mainly defines two calling interfaces:

- Instance interface: the interface that the node calls EVM
- Callback interface: the interface of the EVM Callback node.

EVM itself does not save status data. The node operates EVM through the instance interface. In turn, EVM calls the Callback interface to operate the status of the node.

**Instance interface**

Defines the operations of nodes on virtual machines, including creation, destruction, and setting.

The interface is defined in evmc_instance(evmc.h)

* abi_version  
* name  
* version  
* destroy  
* execute  
* set_tracer  
* set_option

**Callback interface**

Defines how EVM operates on nodes, mainly reading and writing state and block information.

The interface is defined in evmc_context_fn_table(evmc.h).

* evmc_account_exists_fn account_exists
* evmc_get_storage_fn get_storage
* evmc_set_storage_fn set_storage
* evmc_get_balance_fn get_balance
* evmc_get_code_size_fn get_code_size
* evmc_get_code_hash_fn get_code_hash
* evmc_copy_code_fn copy_code
* evmc_selfdestruct_fn selfdestruct
* evmc_call_fn call
* evmc_get_tx_context_fn get_tx_context
* evmc_get_block_hash_fn get_block_hash
* evmc_emit_log_fn emit_log


## EVM execution

### EVM instruction

ssolidity is the execution language of the contract. solidity is compiled by solc and becomes an EVM instruction similar to assembly. The Interpreter defines a complete set of instructions. After solidity is compiled, a binary file is generated. The binary file is a collection of EVM instructions. The transaction is sent to the node in the form of binary. After the node receives it, it calls EVM to execute these instructions through SVMC. In EVM, the logic of these instructions is simulated with code.

Solidity is a stack-based language. When EVM executes binary, it is also called as a stack.

**Arithmetic instruction example**

An ADD instruction. The code in EVM is implemented as follows. SP is the pointer of the stack, from the first and second positions on the top of the stack（```SP[0]```、```SP[1]```）take out the data, add and write it to the top of the result stack SP ```SPP[0]```。

``` cpp
CASE(ADD)
{
    ON_OP();
    updateIOGas();

    // pops two items and pushes their sum mod 2^256.
    m_SPP[0] = m_SP[0] + m_SP[1];
}
```

**Jump instruction example**

The JUMP command realizes the JUMP between binary codes. First from the top of the stack SP[0] Take out the address to be redirected, verify whether it is out of line, and put it in the program counter PC. The next instruction will start from the position pointed by the PC.

``` cpp
CASE(JUMP)
{
    ON_OP();
    updateIOGas();
    m_PC = verifyJumpDest(m_SP[0]);
}
```

**Example of state read instruction**

SLOAD can query status data. The general process is from the top of the stack```SP[0]```take out the key to be accessed, take the key as a parameter, and then adjust the callback function of evmc```get_storage()```，To query the value of the corresponding key. Then write the read value to the top of the result stack SPP ```SPP[0]```。

``` cpp
CASE(SLOAD)
{
    m_runGas = m_rev >= EVMC_TANGERINE_WHISTLE ? 200 : 50;
    ON_OP();
    updateIOGas();

    evmc_uint256be key = toEvmC(m_SP[0]);
    evmc_uint256be value;
    m_context->fn_table->get_storage(&value, m_context, &m_message->destination, &key);
    m_SPP[0] = fromEvmC(value);
}
```

**Example of state write instruction**

The SSTORE command can write data to the state of a node. The general process is from the first and second positions at the top of the stack（```SP[0]```、```SP[1]```）take out key and value, take key and value as parameters, and call the callback function of evmc ```set_storage()``` , write the status of the node.

``` cpp
CASE(SSTORE)
{
    ON_OP();
    if (m_message->flags & EVMC_STATIC)
        throwDisallowedStateChange();

    static_assert(
        VMSchedule::sstoreResetGas <= VMSchedule::sstoreSetGas, "Wrong SSTORE gas costs");
    m_runGas = VMSchedule::sstoreResetGas;  // Charge the modification cost up front.
    updateIOGas();

    evmc_uint256be key = toEvmC(m_SP[0]);
    evmc_uint256be value = toEvmC(m_SP[1]);
    auto status =
        m_context->fn_table->set_storage(m_context, &m_message->destination, &key, &value);

    if (status == EVMC_STORAGE_ADDED)
    {
        // Charge additional amount for added storage item.
        m_runGas = VMSchedule::sstoreSetGas - VMSchedule::sstoreResetGas;
        updateIOGas();
    }
}
```

**Contract call instruction example**

The CALL instruction can CALL another contract based on the address. First, EVM determines that the CALL instruction is called ```caseCall()```，caseCall() ```used```caseCallSetup()```Take the data from the stack, encapsulate it into msg, and call evmc's callback function as a parameter. Eth is being called back```call()```after，Start a new EVM, process the call, and then execute the result of the new EVM by```call()```parameter is returned to the current EVM. The current EVM writes the result to the result stack SSP and the call ends. The logic of contract creation is similar to this logic.

``` cpp
CASE(CALL)
CASE(CALLCODE)
{
    ON_OP();
    if (m_OP == Instruction::DELEGATECALL && m_rev < EVMC_HOMESTEAD)
        throwBadInstruction();
    if (m_OP == Instruction::STATICCALL && m_rev < EVMC_BYZANTIUM)
        throwBadInstruction();
    if (m_OP == Instruction::CALL && m_message->flags & EVMC_STATIC && m_SP[2] != 0)
        throwDisallowedStateChange();
    m_bounce = &VM::caseCall;
}
BREAK

void VM::caseCall()
{
    m_bounce = &VM::interpretCases;

    evmc_message msg = {};

    // Clear the return data buffer. This will not free the memory.
    m_returnData.clear();

    bytesRef output;
    if (caseCallSetup(msg, output))
    {
        evmc_result result;
        m_context->fn_table->call(&result, m_context, &msg);

        m_returnData.assign(result.output_data, result.output_data + result.output_size);
        bytesConstRef{&m_returnData}.copyTo(output);

        m_SPP[0] = result.status_code == EVMC_SUCCESS ? 1 : 0;
        m_io_gas += result.gas_left;

        if (result.release)
            result.release(&result);
    }
    else
    {
        m_SPP[0] = 0;
        m_io_gas += msg.gas;
    }
    ++m_PC;
}
```

## Summary

EVM is a state execution machine. The input is the binary instruction compiled by solidity and the state data of the node. The output is the change of the node state. Simplechain implements compatibility with various virtual machines through EVMC.