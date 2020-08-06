
### How to issue digital assets on SimpleChain

**Prerequisites**

Preparations for implementing publish coin on the chain include:

-  Download and deploy SimpleChain.
-  Start the node.
-  Create an account.
-  Ensure that the account is unlocked and has a token.

**Preparatory work**

1. Understand the ERC20 token standard.

`Explanation: The implementation of tokens on SimpleChain needs to follow the ERC20 standard. `

    //ERC20Token.sol
    //ERC 合约标准，该标准规定在发 Token 之前，需要指定 token 的名称、标识、总量、实现合 
    //约标准函数等    
    pragma solidity ^0.4.26; 
    
    contract ERC20Token {
        //获取 token 名称
        function name() public constant returns (string name);
        //获取 token 标识
        function symbol() public constant returns (string symbol);
        //获取 token 的最小分割量
        function decimals() public constant returns (uint8 decimals);
        //获取 token 的总量
        function totalSupply() public constant returns (uint256 totalSupply);
        //获取_owner 账户当前的 token 量
        function balanceOf(address _owner) public constant returns (uint256 balance);
        //转账交易
        function transfer(address _to, uint256 _value) public returns (bool success);
        //由_from 向_to 进行转账
        function transferFrom(address _from, address _to, uint256 _value) public returns (bool success);
        //许可_spender 能从调用合约方法的账户转出总量为_value 的 token
        function approve(address _spender, uint256 _value) public returns (bool success);
        //获取_spender 可以从账户_owner 中转出 token 的剩余数量
        function allowance(address _owner, address _spender) public constant returns (uint remaining);
        //转账事件(transfer、transferFrom 会触发该事件)
        event Transfer(address indexed _from, address indexed _to, uint256 _value);
        //许可事件(approve 会触发该事件)
        event Approval(address indexed _owner, address indexed _spender, uint256 _value);
    }

2. Compile token contracts that meet ERC20 standards.

    //Mytoken.sol
    pragma solidity ^0.4.26;
    import "./ERC20Token.sol";
      
    contract MyToken is ERC20Token {
        string private _name;
        string private _symbol;
        uint8 private _decimals = 18; //此处建议为 18，代表最小单位为 0.1^18 
        uint256 private _totalSupply;

        //存储账户的 token 总量
        mapping(address => uint256) private _balances;
        //存储前一个address允许后一个address转出token的剩余数量 
        mapping(address => mapping(address => uint256)) private _allowances;

        function MyToken(uint256 initialSupply, string tokenName, string tokenSymbol) public{
            _name = tokenName;
            _symbol = tokenSymbol;
            _totalSupply = initialSupply * 10 ** uint256(_decimals);
            _balances[msg.sender] = _totalSupply;
       }

       function name() public constant returns (string name){ 
            name = _name;
       }
       function symbol() public constant returns (string symbol){ 
            symbol = _symbol;
       }
       function decimals() public constant returns (uint8 decimals){ 
            decimals = _decimals;
       }
       function totalSupply() public constant returns (uint256 totalSupply){ 
            totalSupply = _totalSupply;                
       }
       function balanceOf(address _owner) public constant returns (uint256 balance){ 
            balance = _balances[_owner];
       }
       function transfer(address _to, uint256 _value) public returns (bool success){ 
           require(_balances[msg.sender] >= _value); //保证发出交易的账户 token 足够完成转账
           _balances[msg.sender] -= _value; _balances[_to] += value; Transfer(msg.sender, _to, _value); success = true;
       }
       function transferFrom(address _from, address _to, uint256 _value) public returns (bool success){
           require(_balances[_from] >= _value); //保证_from 账户 token 足够完成转 账
           require(_allowances[_from][msg.sender] >= _value); //保证_from 账户允许 执行账户转出的 token 剩余量足够
           _balances[_from] -= _value; _allowances[_from][msg.sender] -= _value; _balances[_to] += _value;
           Transfer(_from, _to, _value);
           success = true;
       }
                                 }
       //许可_spender 能从调用合约方法的账户转出总量为_value 的 token
       function approve(address _spender, uint256 _value) public returns (bool success){
           _allowances[msg.sender][_spender] = _value; 
           Approval(msg.sender, _spender, _value); success = true;
       }
                  
       //获取_spender 可以从账户_owner 中转出 token 的剩余数量
       function allowance(address _owner, address _spender) public constant returns (uint256 remaining){
            remaining = _allowances[_owner][_spender]; 
        }
    }

 **Contract compilation:**

1. Use a browser to open [Remix Solidity IDE](http://remix.sipc.vip/).
2. Add ERC20Token.sol and MyToken.sol to the browser folder.
3. Select the Compile option in the right-side box of the webpage and select the version of the contract compiler (0.4.26 is selected in this article).
4. Click Start to compile.

![6.1.png](https://i.loli.net/2020/05/07/DbgwWI8Yztu7Unx.png)
![6.1.png](1.png)
**Contract deployment**

1. Select the Run option and select the Web3 Provider in the Environment. In the dialog box that appears, enter the port of the currently enabled sipe node.

![6.2.png](https://i.loli.net/2020/05/07/umSzyZqigevbMxY.png)
![6.2.png](2.png)
2. Select an unlocked Account with a token.
3. Select the MyToken contract, enter the initialization parameters in the Deploy field: total number of tokens, token name, token identifier, and click Deploy.
4. Wait about one minute until the contract deployment is completed. If the contract appears in the Deployed Contracts column, the deployment is completed. So far, the Fa Bi completed.

![6.3.png](https://i.loli.net/2020/05/07/sJiXawq9SDo7Gl6.png)
![6.3.png](3.png)
    
**Contract Verification**

1. Execute the decimals,name,symbol,totalSupply methods of the contract to check whether the token is successfully created.

![6.4.png](https://i.loli.net/2020/05/07/ltjSce5JfPLDqxI.png)
![6.4.png](4.png)
2. Execute the transfer method, expand the transfer method to transfer money to other accounts, and use balanceOf to check whether the transfer is successful.

![6.5.png](https://i.loli.net/2020/05/07/NblfOHyevhS3kDr.png)
![6.5.png](5.png)




