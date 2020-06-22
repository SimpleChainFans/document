## 一. Project code management

[Project code base](https://github.com/SimpleChainFans/funding.git)

## 二. Single crowdfunding contract implementation

### 1.Create empty contract CrowFunding

Enter the item catalog and create the file `basicFunding.sol`

![33.1.png](https://i.loli.net/2020/06/09/nWhL2mjlqwIEa4t.png)

And add the following code:
```go
pragma solidity ^0.4.24;

contractCrowFunding	{

}
```
### 2.Basic attributes (state variables)

State variable|	Type |	use
-|-|-
creator	| address |	Project Initiation, responsible for contract creation, expense application, expense execution
projectName	| string |	Name of the crowdfunding item
supportBalance |	uint |	Crowdfunding holding amount
targetBalance	| uint |	Crowdfunding project bid raising amount
endTime	|uint	| Crowdfunding is cut off. If the amount of crowdfunding cannot be raised at this time, crowdfunding fails.

### 3.Constructor implementation

```go
pragma solidity ^0.4.24;

contract CrowFunding {
   address public creator; // 发起人
   string public projectName; // 项目名称
   uint public supportBalance; // 参与众筹金额
   uint public targetBalance; // 众筹目标金额 
   uint public endTime; // 众筹截止时间

constructor(string _projectName, uint _supportBalance, uint _targetBalance, uin t _durationInSeconds) public {
   creator = msg.sender;
   projectName = _projectName;
   supportBalance = _supportBalance;
   targetBalance = _targetBalance; 
   //传递进来剩余的秒数，比如若众筹30天，则传入：30天 * 24小时 * 60分 * 60秒 = 2592000 
   endTime = now + _durationInSeconds; //2592000
   } 
}
```
**Test**

![](https://i.loli.net/2020/06/09/qWgeNnbSyvUOo3P.png)

### 4.Participate in crowdfunding

**Implementation**

Add participant attributes

 address[] public investors; //people who participate in crowdfunding, namely investors

Participating in crowdfunding means transferring money to the contract and adding the participants' addresses to the collection. The code is as follows:

```go
function invest() public payable { 
   require(investorExistMap[msg.sender] == false);//每个人只能参与一次

   require(msg.value == supportBalance); // 支持固定金额
   investors.push(msg.sender); // 添加到众筹人数组中
   investorExistMap[msg.sender] = true; // 标记当前账户为参与人
}
```
In order to quickly verify whether an account is in an array of participants, we provide a `mapping(address=>bool)`to mark `mapping` the feature is that all keys exist by default, but the default value is`false`，if it does not exist, return `false`,we use the user address `key`,set the value `true`,to complete the index, mapping is a linear index, than using for loop traversal `investors` arrays are efficient and economical, so you need to add the following attributes:

    mapping(address => bool) public investorExistMap; //标记一个人是否参与了当前众筹

**test**

Test, please operate in digital order after deployment

![33.6.png](https://i.loli.net/2020/06/09/2iB5fNXOwYkcFlM.png)

### Refund for crowdfunding failure (implementation)

Refund means that all the money raised will be returned to investors one by one. Two auxiliary functions are added at the same time to facilitate testing.

```go
//众筹失败，退款
function drawBack() public {
    for (uint i = 0 ; i < investors.length; i++) {
        investors[i].transfer(supportBalance);
}
//查看合约当前余额
function getCurrentBalance() public view returns(uint) {
    return address(this).balance;
}
//返回所有投资人
function getInvestors() public view returns(address[]) {
    return investors;
}
```
**Test**

![33.7.png](https://i.loli.net/2020/06/09/tpwDoV3yMWrjxN4.png)

### Cost request (implementation)

- Define structure
If crowdfunding is successful and the project is started, a fee needs to be pointed out, which needs to include the following information:

1. Purpose: What to buy?
2. Cost: How much does it cost?
3. Merchant address: from whom?
4. Number of votes approved at present: how many people approve, more than half of them approve the expenditure
5. Current status of this expense application: current status of this application: completed? To be approved? On behalf of the implementation?
6. Mark the set of people who have voted: mapping(address =>bool), a mark set of people in favor, to prevent one person from voting for many times.

Define the structure code based on the analysis:

```go
struct Request {
   string purpose; //买什么？
   uint cost; //需要多少钱？
   address shopAddress; // 向谁购买？
   uint voteCount; // 多少人赞成了，超半数则批准支出
   mapping(address => bool) investorVotedMap; //赞成人的标记集合，防止一人重复投票多次
   RequestStatus status; //这个申请的当前状态：投票中？已批准？已完成？
}
```
Define an enumeration to describe the application status:

```go
enum RequestStatus {Voting,Approved,Completed}
```
- Definition method
This function is relatively simple. Create a new request structure and add it to the array.

The code is as follows:

```go
Request[] public requests; //请求可能有多个，所以定义一个数组
function createRequest(string _purpose, uint _cost, address _shopAddress) public {
       Request memory request = Request({
           purpose : _purpose,
           cost : _cost,
           shopAddress : _shopAddress,
           voteCount : 0,
           status : RequestStatus.Voting
       });
      requests.push(request); //将新的请求添加至数组中
}
```
**Test**

In `createRequest` add parameters in:

```json
"小胖子减肥", 100, "0xca35b7d915458ef540ade6068dfe2f44e8fa733c"
```
Search for the 0th request in the request, as shown in the following figure:

![33.9.png](https://i.loli.net/2020/06/09/Nyg2HJB6Q1DtUiq.png)

### Approval of payment application

**Implementation**

After the project Party initiates the application, it will be approved by the crowdfunding personnel, if the investors do not support it. Ignore it. The default value is false (not supported). If you want to support it, you need to perform an approval action. That is, modify the status of the data of the application structure, including:

1. Check whether this person has voted. If not, vote is allowed. Otherwise, exit.
2. voteCount data plus 1.
3. Set the value of this voter's re-investorVotedMap mapping to true.

The code is as follows:

```golang
//批准⽀付申请
function approveRequest(uint256 index) public {
    // 1. 检验这个⼈是否投过票，若未投过，则允许投票，反之退出
    // 2. voteCount数据加1。
    // 3. 将该投票⼈在investorVotedMap映射中的值设置为true。

    //⾸先要确保是参与众筹的⼈，否则⽆权投票
    require(investExitMapping[msg.sender]);
    //根据索引找到特定的请求
    Request storage req = requests[index];
    //确保没有投过票，⼈⼿⼀票
    require(req.investorVotedMap[msg.sender] == false);
    //如果已经完成，或者已经获得批准了，就不⽤投票了，当前投票不会影响决策。
    require(req.status == RequestStatus.Voting);

    //⽀持票数加1
    req.voteCount += 1;
    //标记为已投票
    req.investorVotedMap[msg.sender] = true;
    if (req.voteCount * 2 > investors.length) {
        req.status == RequestStatus.Approved;
    }
}
```
**Test**

View the request after approval, and the voteCount becomes 1

![33.11.png](https://i.loli.net/2020/06/09/5GvWw7rMt1H4RZU.png)

### Complete expense request

**Implementation**

When more than half of the votes are voted, if the expenses are approved, the expenses can be executed by the project Party, or by the project Party, or by the contract automatically. We choose to execute it manually, because it is possible that the project Party changes and pays attention to it, and there is no need to purchase on the right side of the plan, so we transfer the rights to the project party.

This function mainly does two things:

1.If the number of votes exceeds half, the transfer will be executed.
2.Update the status of the request

Coding are as follows:

```golang
function finalizeRequest(uint256 index) public onlyManager{
    // 这个函数主要做两件事：
    // 1. 票数过半，则执⾏转账。
    // 2. 更新request的状态。

    Request storage req = requests[index];
    //合约⾦额充⾜才可以执⾏
    require(address(this).balance >= req.cost);
    //赞成⼈数过半
    require(req.voteCount * 2 > investors.length);
    //转账
    req.shopAddress.transfer(req.cost);
    //更新请求状态为已完成
    req.status = RequestStatus.Completed;
}

```

**Test**

Three people invest, one is in favor, two are against, and the execution of payment fails.

![33.12.png](https://i.loli.net/2020/06/09/XLOFqN3Bs6gHSxh.png)

Two people agree, the execution of the payment is successful

![33.13.png](https://i.loli.net/2020/06/09/Bo82mNzg4kSIldP.png)

To fully implement smart contracts, several methods need to be implemented and the mutual calling of contract methods. These can be learned directly from the source code. But the contract writing and testing are now in the above several basic methods.