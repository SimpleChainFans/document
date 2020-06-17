Some contract templates of Jin Dian can help developers learn quickly `Solidity`,and quickly start the development. Create excellent Dapp applications based on Simplechain.

## Vote

The following contract is quite complex, but shows many Solidity functions. It achieved a voting contract.
Of course, the main problem of electronic voting is how to distribute the voting rights to the right people and how to prevent manipulation.
We will not solve all the problems here, but at least we will show how to conduct delegation voting. At the same time, counting votes is also **Automatic and fully transparent** 。

Our idea is to create a contract for each (vote) vote and provide short for each option. Then as the creator of the contract-the chairman, he will give each independent address the right to vote. People behind the address can choose to vote by themselves or entrust people they trust to vote. At the end of the voting time，``winningProposal()`` the proposal that received the most votes will be returned.

```sh
    pragma solidity ^0.4.22;

    /// @title 委托投票
    contract Ballot {
        // 这里声明了一个新的复合类型用于稍后的变量
        // 它用来表示一个选民
        struct Voter {
            uint weight; // 计票的权重
            bool voted;  // 若为真，代表该人已投票
            address delegate; // 被委托人
            uint vote;   // 投票提案的索引
        }

        // 提案的类型
        struct Proposal {
            bytes32 name;   // 简称（最长32个字节）
            uint voteCount; // 得票数
        }

        address public chairperson;

        // 这声明了一个状态变量，为每个可能的地址存储一个 `Voter`。
        mapping(address => Voter) public voters;

        // 一个 `Proposal` 结构类型的动态数组
        Proposal[] public proposals;

        /// 为 `proposalNames` 中的每个提案，创建一个新的（投票）表决
        constructor(bytes32[] proposalNames) public {
            chairperson = msg.sender;
            voters[chairperson].weight = 1;
            //对于提供的每个提案名称，
            //创建一个新的 Proposal 对象并把它添加到数组的末尾。
            for (uint i = 0; i < proposalNames.length; i++) {
                // `Proposal({...})` 创建一个临时 Proposal 对象，
                // `proposals.push(...)` 将其添加到 `proposals` 的末尾
                proposals.push(Proposal({
                    name: proposalNames[i],
                    voteCount: 0
                }));
            }
        }

        // 授权 `voter` 对这个（投票）表决进行投票
        // 只有 `chairperson` 可以调用该函数。
        function giveRightToVote(address voter) public {
            // 若 `require` 的第一个参数的计算结果为 `false`，
            // 则终止执行，撤销所有对状态和以太币余额的改动。
            // 在旧版的 EVM 中这曾经会消耗所有 gas，但现在不会了。
            // 使用 require 来检查函数是否被正确地调用，是一个好习惯。
            // 你也可以在 require 的第二个参数中提供一个对错误情况的解释。
            require(
                msg.sender == chairperson,
                "Only chairperson can give right to vote."
            );
            require(
                !voters[voter].voted,
                "The voter already voted."
            );
            require(voters[voter].weight == 0);
            voters[voter].weight = 1;
        }

        /// 把你的投票委托到投票者 `to`。
        function delegate(address to) public {
            // 传引用
            Voter storage sender = voters[msg.sender];
            require(!sender.voted, "You already voted.");

            require(to != msg.sender, "Self-delegation is disallowed.");

            // 委托是可以传递的，只要被委托者 `to` 也设置了委托。
            // 一般来说，这种循环委托是危险的。因为，如果传递的链条太长，
            // 则可能需消耗的gas要多于区块中剩余的（大于区块设置的gasLimit），
            // 这种情况下，委托不会被执行。
            // 而在另一些情况下，如果形成闭环，则会让合约完全卡住。
            while (voters[to].delegate != address(0)) {
                to = voters[to].delegate;

                // 不允许闭环委托
                require(to != msg.sender, "Found loop in delegation.");
            }

            // `sender` 是一个引用, 相当于对 `voters[msg.sender].voted` 进行修改
            sender.voted = true;
            sender.delegate = to;
            Voter storage delegate_ = voters[to];
            if (delegate_.voted) {
                // 若被委托者已经投过票了，直接增加得票数
                proposals[delegate_.vote].voteCount += sender.weight;
            } else {
                // 若被委托者还没投票，增加委托者的权重
                delegate_.weight += sender.weight;
            }
        }

        /// 把你的票(包括委托给你的票)，
        /// 投给提案 `proposals[proposal].name`.
        function vote(uint proposal) public {
            Voter storage sender = voters[msg.sender];
            require(!sender.voted, "Already voted.");
            sender.voted = true;
            sender.vote = proposal;

            // 如果 `proposal` 超过了数组的范围，则会自动抛出异常，并恢复所有的改动
            proposals[proposal].voteCount += sender.weight;
        }

        /// @dev 结合之前所有的投票，计算出最终胜出的提案
        function winningProposal() public view
                returns (uint winningProposal_)
        {
            uint winningVoteCount = 0;
            for (uint p = 0; p < proposals.length; p++) {
                if (proposals[p].voteCount > winningVoteCount) {
                    winningVoteCount = proposals[p].voteCount;
                    winningProposal_ = p;
                }
            }
        }

        // 调用 winningProposal() 函数以获取提案数组中获胜者的索引，并以此返回获胜者的名称
        function winnerName() public view
                returns (bytes32 winnerName_)
        {
            winnerName_ = proposals[winningProposal()].name;
        }
    }
```

**Possible optimization**

At present, in order to distribute the voting rights to all participants, many transactions need to be executed. Do you have a better idea?

**Secret bidding (blind auction)**

In this section, we will show how to easily create a secret bidding contract on Simplechain.
We will start from the public auction, everyone can see the bid, and then extend this contract to blind auction contract,
The actual bid cannot be seen before the bidding period ends.

**Simple public auction**

The general idea of the following simple auction contract is that everyone can send their bids within the bidding period.
The bid already contains funds/Ethernet coins to bind bidders to their bids.
If the highest bid is raised (it is exceeded by other bidders), the former highest bidder can get her money back. At the end of the bidding period, the beneficiary needs to manually call the contract to receive his money-the contract cannot activate the receipt by itself.

```sh
    pragma solidity ^0.4.22;

    contract SimpleAuction {
        // 拍卖的参数。
        address public beneficiary;
        // 时间是unix的绝对时间戳（自1970-01-01以来的秒数）
        // 或以秒为单位的时间段。
        uint public auctionEnd;

        // 拍卖的当前状态
        address public highestBidder;
        uint public highestBid;

        //可以取回的之前的出价
        mapping(address => uint) pendingReturns;

        // 拍卖结束后设为 true，将禁止所有的变更
        bool ended;

        // 变更触发的事件
        event HighestBidIncreased(address bidder, uint amount);
        event AuctionEnded(address winner, uint amount);

        // 以下是所谓的 natspec 注释，可以通过三个斜杠来识别。
        // 当用户被要求确认交易时将显示。

        /// 以受益者地址 `_beneficiary` 的名义，
        /// 创建一个简单的拍卖，拍卖时间为 `_biddingTime` 秒。
        constructor(
            uint _biddingTime,
            address _beneficiary
        ) public {
            beneficiary = _beneficiary;
            auctionEnd = now + _biddingTime;
        }

        /// 对拍卖进行出价，具体的出价随交易一起发送。
        /// 如果没有在拍卖中胜出，则返还出价。
        function bid() public payable {
            // 参数不是必要的。因为所有的信息已经包含在了交易中。
            // 对于能接收以太币的函数，关键字 payable 是必须的。

            // 如果拍卖已结束，撤销函数的调用。
            require(
                now <= auctionEnd,
                "Auction already ended."
            );

            // 如果出价不够高，返还你的钱
            require(
                msg.value > highestBid,
                "There already is a higher bid."
            );

            if (highestBid != 0) {
                // 返还出价时，简单地直接调用 highestBidder.send(highestBid) 函数，
                // 是有安全风险的，因为它有可能执行一个非信任合约。
                // 更为安全的做法是让接收方自己提取金钱。
                pendingReturns[highestBidder] += highestBid;
            }
            highestBidder = msg.sender;
            highestBid = msg.value;
            emit HighestBidIncreased(msg.sender, msg.value);
        }

        /// 取回出价（当该出价已被超越）
        function withdraw() public returns (bool) {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // 这里很重要，首先要设零值。
                // 因为，作为接收调用的一部分，
                // 接收者可以在 `send` 返回之前，重新调用该函数。
                pendingReturns[msg.sender] = 0;

                if (!msg.sender.send(amount)) {
                    // 这里不需抛出异常，只需重置未付款
                    pendingReturns[msg.sender] = amount;
                    return false;
                }
            }
            return true;
        }

        /// 结束拍卖，并把最高的出价发送给受益人
        function auctionEnd() public {
            // 对于可与其他合约交互的函数（意味着它会调用其他函数或发送以太币），
            // 一个好的指导方针是将其结构分为三个阶段：
            // 1. 检查条件
            // 2. 执行动作 (可能会改变条件)
            // 3. 与其他合约交互
            // 如果这些阶段相混合，其他的合约可能会回调当前合约并修改状态，
            // 或者导致某些效果（比如支付以太币）多次生效。
            // 如果合约内调用的函数包含了与外部合约的交互，
            // 则它也会被认为是与外部合约有交互的。

            // 1. 条件
            require(now >= auctionEnd, "Auction not yet ended.");
            require(!ended, "auctionEnd has already been called.");

            // 2. 生效
            ended = true;
            emit AuctionEnded(highestBidder, highestBid);

            // 3. 交互
            beneficiary.transfer(highestBid);
        }
    }
```
## Secret auction (blind auction)

The previous public auction will then be expanded to a secret auction.
The advantage of secret auction is that there will be no time pressure before the bid ends.
Secret auction on a transparent computing platform sounds self-contradictory, but cryptography can realize it.

In **Bidding period** ，the bidder did not actually send her bid, but only sent a hash version of the bid.
Since it is almost impossible to find two (long enough) values at present and their hash values are equal, tenderers can submit quotations in this way.
After the bidding is completed, bidders must disclose their bids: they send their bids unencrypted, and the contract checks whether the hash value of the bid is the same as that provided during the bidding period.

Another challenge is how to make the auction work at the same time **Binding and secret** :
The only way to prevent the bidder from not paying after she wins the auction is to let her send out the money together with the bid.
However, since the transfer of funds cannot be hidden in Simplechain, anyone can see the transferred funds.

The following contract solves this problem by accepting any value greater than the highest bid.
Of course, because this can only be checked at the disclosure stage, some bids may be **Invalid** of，
Moreover, this is intentional (together with the high bid, it even provides a clear mark to identify invalid bids):
Bidders can confuse competitors by setting several high or low invalid bids.

```sh
    pragma solidity >0.4.23 <0.5.0;

    contract BlindAuction {
        struct Bid {
            bytes32 blindedBid;
            uint deposit;
        }

        address public beneficiary;
        uint public biddingEnd;
        uint public revealEnd;
        bool public ended;

        mapping(address => Bid[]) public bids;

        address public highestBidder;
        uint public highestBid;

        // 可以取回的之前的出价
        mapping(address => uint) pendingReturns;

        event AuctionEnded(address winner, uint highestBid);

        /// 使用 modifier 可以更便捷的校验函数的入参。
        /// `onlyBefore` 会被用于后面的 `bid` 函数：
        /// 新的函数体是由 modifier 本身的函数体，并用原函数体替换 `_;` 语句来组成的。
        modifier onlyBefore(uint _time) { require(now < _time); _; }
        modifier onlyAfter(uint _time) { require(now > _time); _; }

        constructor(
            uint _biddingTime,
            uint _revealTime,
            address _beneficiary
        ) public {
            beneficiary = _beneficiary;
            biddingEnd = now + _biddingTime;
            revealEnd = biddingEnd + _revealTime;
        }

        /// 可以通过 `_blindedBid` = keccak256(value, fake, secret)
        /// 设置一个秘密竞拍。
        /// 只有在出价披露阶段被正确披露，已发送的以太币才会被退还。
        /// 如果与出价一起发送的以太币至少为 “value” 且 “fake” 不为真，则出价有效。
        /// 将 “fake” 设置为 true ，然后发送满足订金金额但又不与出价相同的金额是隐藏实际出价的方法。
        /// 同一个地址可以放置多个出价。
        function bid(bytes32 _blindedBid)
            public
            payable
            onlyBefore(biddingEnd)
        {
            bids[msg.sender].push(Bid({
                blindedBid: _blindedBid,
                deposit: msg.value
            }));
        }

        /// 披露你的秘密竞拍出价。
        /// 对于所有正确披露的无效出价以及除最高出价以外的所有出价，你都将获得退款。
        function reveal(
            uint[] _values,
            bool[] _fake,
            bytes32[] _secret
        )
            public
            onlyAfter(biddingEnd)
            onlyBefore(revealEnd)
        {
            uint length = bids[msg.sender].length;
            require(_values.length == length);
            require(_fake.length == length);
            require(_secret.length == length);

            uint refund;
            for (uint i = 0; i < length; i++) {
                Bid storage bid = bids[msg.sender][i];
                (uint value, bool fake, bytes32 secret) =
                        (_values[i], _fake[i], _secret[i]);
                if (bid.blindedBid != keccak256(value, fake, secret)) {
                    // 出价未能正确披露
                    // 不返还订金
                    continue;
                }
                refund += bid.deposit;
                if (!fake && bid.deposit >= value) {
                    if (placeBid(msg.sender, value))
                        refund -= value;
                }
                // 使发送者不可能再次认领同一笔订金
                bid.blindedBid = bytes32(0);
            }
            msg.sender.transfer(refund);
        }

        // 这是一个 "internal" 函数， 意味着它只能在本合约（或继承合约）内被调用
        function placeBid(address bidder, uint value) internal
                returns (bool success)
        {
            if (value <= highestBid) {
                return false;
            }
            if (highestBidder != address(0)) {
                // 返还之前的最高出价
                pendingReturns[highestBidder] += highestBid;
            }
            highestBid = value;
            highestBidder = bidder;
            return true;
        }

        /// 取回出价（当该出价已被超越）
        function withdraw() public {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // 这里很重要，首先要设零值。
                // 因为，作为接收调用的一部分，
                // 接收者可以在 `transfer` 返回之前重新调用该函数。（可查看上面关于‘条件 -> 影响 -> 交互’的标注）
                pendingReturns[msg.sender] = 0;

                msg.sender.transfer(amount);
            }
        }

        /// 结束拍卖，并把最高的出价发送给受益人
        function auctionEnd()
            public
            onlyAfter(revealEnd)
        {
            require(!ended);
            emit AuctionEnded(highestBidder, highestBid);
            ended = true;
            beneficiary.transfer(highestBid);
        }
    }
```

## Secure Remote purchase

```sh
    pragma solidity ^0.4.22;

    contract Purchase {
        uint public value;
        address public seller;
        address public buyer;
        enum State { Created, Locked, Inactive }
        State public state;

        //确保 `msg.value` 是一个偶数。
        //如果它是一个奇数，则它将被截断。
        //通过乘法检查它不是奇数。
        constructor() public payable {
            seller = msg.sender;
            value = msg.value / 2;
            require((2 * value) == msg.value, "Value has to be even.");
        }

        modifier condition(bool _condition) {
            require(_condition);
            _;
        }

        modifier onlyBuyer() {
            require(
                msg.sender == buyer,
                "Only buyer can call this."
            );
            _;
        }

        modifier onlySeller() {
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }

        modifier inState(State _state) {
            require(
                state == _state,
                "Invalid state."
            );
            _;
        }

        event Aborted();
        event PurchaseConfirmed();
        event ItemReceived();

        ///中止购买并回收以太币。
        ///只能在合约被锁定之前由卖家调用。
        function abort()
            public
            onlySeller
            inState(State.Created)
        {
            emit Aborted();
            state = State.Inactive;
            seller.transfer(address(this).balance);
        }

        /// 买家确认购买。
        /// 交易必须包含 `2 * value` 个以太币。
        /// 以太币会被锁定，直到 confirmReceived 被调用。
        function confirmPurchase()
            public
            inState(State.Created)
            condition(msg.value == (2 * value))
            payable
        {
            emit PurchaseConfirmed();
            buyer = msg.sender;
            state = State.Locked;
        }

        /// 确认你（买家）已经收到商品。
        /// 这会释放被锁定的以太币。
        function confirmReceived()
            public
            onlyBuyer
            inState(State.Locked)
        {
            emit ItemReceived();
            // 首先修改状态很重要，否则的话，由 `transfer` 所调用的合约可以回调进这里（再次接收以太币）。
            state = State.Inactive;

            // 注意: 这实际上允许买方和卖方阻止退款 - 应该使用取回模式。
            buyer.transfer(value);
            seller.transfer(address(this).balance);
        }
    }
```


