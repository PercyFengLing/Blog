# solidity语法笔记

- *// SPDX-License-Identifier: MI*
- pure和view的比较
    - pure：纯纯牛马。包含pure关键字的函数，不能读取也不能写入存储在链上的状态变量
    - view：看客。包含view关键字的函数，能读取单不能写入状态变量。
    - 不写pure和view：函数即可以读取也可以写入状态变量
- payable：带着payable的函数，运行的时候可以给合约转入ETH
- 函数可见性说明符：4种
    - `public`：内部外部均可见。(也可用于修饰状态变量，public变量会自动生成 getter函数，用于查询数值).
    - `private`: 只能从本合约内部访问，继承的合约也不能用（也可用于修饰状态变量）。
    - `external`: 只能从合约外部访问（但是可以用`this.f()`来调用，`f`是函数名）
    - `internal`: 只能从合约内部访问，继承的合约可以用（也可用于修饰状态变量）。
- 返回值return和returns比较：
    - returns：加载函数名后面，用于声明返回的变量类型及变量名
    - return：用于函数主题中，返回指定的变量
- 函数输出方式：
    - 命名式返回：我们可以在`returns`中标明返回变量的名称，这样`solidity`会自动给这些变量初始化，并且自动返回这些函数的值，不需要加`return`
    - 结构式赋值：`solidity`使用解构式赋值的规则，支持读取函数的全部或部分返回值。
- sodility数据存储位置：
    - storage：状态变量默认都是storage，存储在链上
    - memory：函数里的参数和临时变量一般用memory，存储在内存中不上链
    - calldata：和memory类似，存储在内存中不上链，与memory不同的是calldata变量不能修改
- 数据位置和赋值规则：在不同存储类型相互赋值时候，有时会产生独立的副本（修改新变量不会影响原变量），有时会产生引用（修改新变量会影响原变量）。规则如下：
    - storage（合约的状态变量）赋值给本地storage（函数里的）时候，会创建引用，改变新变量会影响原有变量
    - storage赋值给memory，会创建独立的副本，修改其中一个不会影响另一个，反之亦然
    - memory赋值给memory，会创建引用，改变新变量会影响原有变量
    - 其他情况，变量赋值给storage，会创建独立的副本，修改其中一个不会影响另一个
- 变量的作用域：`Solidity`中变量按作用域划分有三种，分别是状态变量（state variable），局部变量（local variable）和全局变量(global variable)
    - 状态变量是数据存储在链上的变量，所有合约内函数都可以访问，gas消耗高。状态变量在合约内、函数外声明
    - 局部变量是仅在函数执行过程中有效的变量，函数推出后，变量无效。局部变量的数据存储在内存里，不上链，gas低
    - 全局变量是全局范围工作的变量，都是solidity预留关键字，他们可以在函数内不声明直接使用
    - 常见的全局变量：
        - `blockhash(uint blockNumber)`: (`bytes32`)给定区块的哈希值 – 只适用于256最近区块, 不包含当前区块。
        - `block.coinbase`: (`address payable`) 当前区块矿工的地址
        - `block.gaslimit`: (`uint`) 当前区块的gaslimit
        - `block.number`: (`uint`) 当前区块的number
        - `block.timestamp`: (`uint`) 当前区块的时间戳，为unix纪元以来的秒
        - `gasleft()`: (`uint256`) 剩余 gas
        - `msg.data`: (`bytes calldata`) 完整call data
        - `msg.sender`: (`address payable`) 消息发送者 (当前 caller)
        - `msg.sig` :(`bytes4`) calldata的前四个字节
        - `msg.value`: (`uint`) 当前交易发送的`wei`值
- 创建数组的一些规则：
    - 对于memory修饰的动态数组，可以用new操作符来创建，但是必须声明长度，并且声明后长度不能改变
- 数组成员：
    - length：长度
    - push（）：在数组最后添加一个0元素
    - push（x）：在数组最后添加一个x元素
    - pop（）：可以移除数组最后一个元素
- 结构体：struct
    - 给结构体赋值的两种方法：
    - 方法一：
    
    ```solidity
    // 方法1:在函数中创建一个storage的struct引用
        function initStudent1() external{
            Student storage _student = student; // assign a copy of student
            _student.id = 11;
            _student.score = 100;
        }
    ```
    
    - 方法二：
    
    ```solidity
    // 方法2:直接引用状态变量的struct
        function initStudent2() external{
            student.id = 1;
            student.score = 80;
        }
    ```
    
- 映射的规则：
    - 映射的`_KeyType`只能选择`solidity`默认的类型，比如`uint`，`address`等，不能用自定义的结构体。而`_ValueType`可以使用自定义的类型。
    - 映射的存储位置必须是`storage`，因此可以用于合约的状态变量，函数中`storage`
    变量，和library函数的参数。不能用于`public`函数的参数或返回结果中，因为`mapping`记录的是一种关系 (key - value pair)。
    - 如果映射声明为`public`，那么`solidity`
    会自动给你创建一个`getter`函数，可以通过`Key`来查询对应的`Value`
    - 给映射新增的键值对的语法为`_Var[_Key] = _Value`，其中`_Var`是映射变量名，`_Key`和`_Value`对应新增的键值对。
- 值类型初始值：
    - boolean:false
    - string:””
    - int:0
    - uint:0
    - enum:枚举中的第一个元素
    - address:0x0000000000000000000000000000000000000000
    - function:
        - internal:空白方程
        - external：空白方程
- 引用类型初始值：
    - 映射：所有元素都为其默认值的mapping
    - 结构体：所有成员设为其默认值的结构体
    - 动态数组：[]
    - 静态数组：所有成员设为其默认值的静态数组
- delete操作符：会让变量的值变为初始值
- constant（常量）：变量必须在声明的时候初始化，之后再也不能改变。尝试改变的话，编译不通过
- immutable（不变量）：可以在声明时或构造函数中初始化，因此更加灵活。同样不能在初始化后改变值
- 控制流：
    - if-else：
    - for循环
    - while循环
    - do-while循环
    - 三元运算符
    - continue：立即进入下一个循环
        
        break：跳出当前循环
        
- 插入排序
    - 插入排序代码
        
        ```solidity
        // 插入排序 正确版
            function insertionSort(uint[] memory a) public pure returns(uint[] memory) {
                // note that uint can not take negative value
                for (uint i = 1;i < a.length;i++){
                    uint temp = a[i];
                    uint j=i;
                    while( (j >= 1) && (temp < a[j-1])){
                        a[j] = a[j-1];
                        j--;
                    }
                    a[j] = temp;
                }
                return(a);
            }
        ```
        
- 构造函数：每个合约可以定义一个，并且在部署合约的时候自动运行一次，可以用来初始化合约的一些参数，Solidity 0.4.22之后同合约名的构造函数改为了constructor（）写法
- 修饰器：modifier，声明函数拥有的特性，并减少代码冗余，主要使用场景为运行函数前的检查，例如地址，变量，余额等。
    - 名为onlyOwner的modifier，并定义了一个changeOwner函数，由于onlyOwner修饰符的存在，只有原先的owner可以调用，别人调用就会出错，这是最常用的控制智能合约权限的方法
        
        ```solidity
        // 定义modifier
           modifier onlyOwner {
              require(msg.sender == owner); // 检查调用者是否为owner地址
              _; // 如果是的话，继续运行函数主体；否则报错并revert交易
           }
        ```
        
        ```solidity
        function changeOwner(address _newOwner) external onlyOwner{
              owner = _newOwner; // 只有owner地址运行这个函数，并改变owner
           }
        ```
        
- 事件：事件event是EVM上日志的抽象，具有两个特点：
    - 响应：应用程序（ether.js）可以通过**RPC**接口订阅和监听这些事件，并在前端做响应
    - 经济：事件是`EVM`上比较经济的存储数据的方式，每个大概消耗2,000 `gas`；相比之下，链上存储一个新变量至少需要20,000 `gas`
- 事件eg：
    
    ```solidity
    event Transfer(address indexed from, address indexed to, uint256 value);
    ```
    
    `Transfer`事件共记录了3个变量`from`，`to`和`value`，分别对应代币的转账地址，接收地址和转账数量。同时`from`和`to`前面带着`indexed`关键字，每个`indexed`标记的变量可以理解为检索事件的索引“键”，在以太坊上单独作为一个`topic`进行存储和索引，程序可以轻松的筛选出特定转账地址和接收地址的转账事件。每个事件最多有3个带`indexed`的变量。每个 `indexed`变量的大小为固定的256比特。事件的哈希以及这三个带`indexed`的变量在`EVM`日志中通常被存储为`topic`。其中`topic[0]`是此事件的`keccak256`哈希，`topic[1]`到`topic[3]`存储了带`indexed`变量的`keccak256`哈希。
    
    `value` 不带 `indexed`关键字，会存储在事件的 `data` 部分中，可以理解为事件的“值”。`data` 部分的变量不能被直接检索，但可以存储任意大小的数据。因此一般 `data`部分可以用来存储复杂的数据结构，例如数组和字符串等等，因为这些数据超过了256比特，即使存储在事件的 `topic`部分中，也是以哈希的方式存储。另外，`data`部分的变量在存储上消耗的gas相比于 `topic`更少。
    
    在下面的例子中，每次用`_transfer()`函数进行转账操作的时候，都会释放`Transfer`事件，并记录相应的变量。
    
    - 代码
        
        ```solidity
        // 定义_transfer函数，执行转账逻辑
            function _transfer(
                address from,
                address to,
                uint256 amount
            ) external {
        
                _balances[from] = 10000000; // 给转账地址一些初始代币
        
                _balances[from] -=  amount; // from地址减去转账数量
                _balances[to] += amount; // to地址加上转账数量
        
                // 释放事件
                emit Transfer(from, to, amount);
            }
        ```
        
- 继承：sodility是面向对象的编程，支持继承
    - 规则：
        - virtual：父合约中的函数，如果希望子合约重写，需要加上virtual关键字
        - override：子合约重写父合约中的函数，需要加上override关键字
    - 简单继承：contract is yeye
    - 多重继承：contract is yeye,baba。继承时要按辈分最高到最低的顺序排，不然会报错。如果某一个函数在多个继承的合约里都存在，比如例子中的`hip()`和`pop()`，在子合约里必须重写，不然会报错。 重写在多个父合约中都重名的函数时，`override`关键字后面要加上所有父合约名字，例如`override(Yeye, Baba)`。
    - 修饰器：修饰器modifier同样可以继承，用法和函数类似，在相应的地方加上virtual和override即可
    - 构造函数的继承：子合约有两种方法继承父合约的构造函数
        - 父函数：
            
            ```solidity
            // 构造函数的继承
            abstract contract A {
                uint public a;
            
                constructor(uint _a) {
                    a = _a;
                }
            }
            ```
            
        1. 在继承时声明父构造函数的参数，例如：`contract B is A(1)`
        2. 在子合约的构造函数中声明构造函数的参数，例如：
        - 子函数：
            
            ```solidity
            contract C is A {
                constructor(uint _c) A(_c * _c) {}
            }
            ```
            
    - 子合约调用父合约的函数有两种方式，直接调用和利用super关键字
        
        1. 直接调用：子合约可以直接用`父合约名.函数名()`的方式来调用父合约函数，例如`Yeye.pop()`。
        
        2. `super`关键字：子合约可以利用`super.函数名()`来调用最近的父合约函数。`solidity`继承关系按声明时从右到左的顺序是：`contract Erzi is Yeye, Baba`，那么`Baba`是最近的父合约，`super.pop()`将调用`Baba.pop()`而不是`Yeye.pop()`：
        
- 抽象合约：如果一个智能合约里至少有一个未实现的函数，即某个函数缺少主体`{}`中的内容，则必须将该合约标为`abstract`，不然编译会报错；另外，未实现的函数需要加`virtual`，以便子合约重写。
- 接口：接口类似于抽象合约，但不实现任何功能
    - 接口的规则：
        1. 不能包含状态变量
        2. 不能包含构造函数
        3. 不能继承除接口外的其他合约
        4. 所有函数都必须是external且不能有函数体
        5. 继承接口的合约必须实现接口定义的所有功能
    - 接口提供了两个重要的信息：
        
        1. 合约里每个函数的`bytes4`选择器，以及基于它们的函数签名`函数名(每个参数类型）`。
        
        2.接口id
        
    - 接口和合约ABI等价，可以相互转换
    - 什么时候使用接口：
        
        如果我们知道一个合约实现了`IERC721`接口，我们不需要知道它具体代码实现，就可以与它交互。
        
        无聊猿`BAYC`属于`ERC721`代币，实现了`IERC721`接口的功能。我们不需要知道它的源代码，只需知道它的合约地址，用`IERC721`接口就可以与它交互，比如用`balanceOf()`来查询某个地址的`BAYC`余额，用`safeTransferFrom()`来转账`BAYC`。
        
        - 合约：
            
            ```solidity
            contract interactBAYC {
                // 利用BAYC地址创建接口合约变量（ETH主网）
                IERC721 BAYC = IERC721(0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D);
            
                // 通过接口调用BAYC的balanceOf()查询持仓量
                function balanceOfBAYC(address owner) external view returns (uint256 balance){
                    return BAYC.balanceOf(owner);
                }
            
                // 通过接口调用BAYC的safeTransferFrom()安全转账
                function safeTransferFromBAYC(address from, address to, uint256 tokenId) external{
                    BAYC.safeTransferFrom(from, to, tokenId);
                }
            }
            ```
            
- IERC721事件：包含3个事件
    - 事件：
        - `Transfer`事件：在转账时被释放，记录代币的发出地址`from`，接收地址`to`和`tokenid`。
        - `Approval`事件：在授权时释放，记录授权地址`owner`，被授权地址`approved`和`tokenid`。
        - `ApprovalForAll`事件：在批量授权时释放，记录批量授权的发出地址`owner`，被授权地址`operator`和授权与否的`approved`。
- IERC721函数：
    - 函数：
        - `balanceOf`：返回某地址的NFT持有量`balance`。
        - `ownerOf`：返回某`tokenId`的主人`owner`。
        - `transferFrom`：普通转账，参数为转出地址`from`，接收地址`to`和`tokenId`。
        - `safeTransferFrom`：安全转账（如果接收方是合约地址，会要求实现`ERC721Receiver`接口）。参数为转出地址`from`，接收地址`to`和`tokenId`。
        - `approve`：授权另一个地址使用你的NFT。参数为被授权地址`approve`和`tokenId`
        - `getApproved`：查询`tokenId`被批准给了哪个地址
- 异常：
    - Error：`error`是`solidity 0.8版本`新加的内容，方便且高效（省`gas`）地向用户解释操作失败的原因。人们可以在`contract`
    之外定义异常。下面，我们定义一个`TransferNotOwner`异常，当用户不是代币`owner`的时候尝试转账，会抛出错误：
        
        ```solidity
        error TransferNotOwner(); // 自定义error
        ```
        
        在执行当中，`error`必须搭配`revert`（回退）命令使用。
        
        - 代码：
            
            ```solidity
            function transferOwner1(uint256 tokenId, address newOwner) public {
                    if(_owners[tokenId] != msg.sender){
                        revert TransferNotOwner();
                    }
                    _owners[tokenId] = newOwner;
                }
            ```
            
        
        我们定义了一个`transferOwner1()`函数，它会检查代币的`owner`
        是不是发起人，如果不是，就会抛出`TransferNotOwner`异常；如果是的话，就会转账。
        
    - Require：`require`命令是`solidity 0.8版本`之前抛出异常的常用方法，目前很多主流合约仍然还在使用它。它很好用，唯一的缺点就是`gas`随着描述异常的字符串长度增加，比`error`命令要高。使用方法：`require(检查条件，"异常的描述")`，当检查条件不成立的时候，就会抛出异常。
        - 代码：
            
            ```solidity
            function transferOwner2(uint256 tokenId, address newOwner) public {
                    require(_owners[tokenId] == msg.sender, "Transfer Not Owner");
                    _owners[tokenId] = newOwner;
                }
            ```
            
    - Assert：`assert`命令一般用于程序员写程序`debug`，因为它不能解释抛出异常的原因（比`require`少个字符串）。它的用法很简单，`assert(检查条件）`，当检查条件不成立的时候，就会抛出异常。
        - 代码：
            
            ```solidity
            function transferOwner3(uint256 tokenId, address newOwner) public {
                    assert(_owners[tokenId] == msg.sender);
                    _owners[tokenId] = newOwner;
                }
            ```
            
    - 三种方法的gas比较：
        
        error最少，其次assert，require方法消耗gas最多
        
- 重载：
    - `solidity`中允许函数进行重载（`overloading`），即名字相同但输入参数类型不同的函数可以同时存在，他们被视为不同的函数。注意，`solidity`不允许修饰器（`modifier`）重载。
    - 最终重载函数在经过编译器编译后，由于不同的参数类型，都变成了不同的函数选择器（selector）。
- 实参匹配：在调用重载函数时，会把输入的实际参数和函数参数的变量类型做匹配。 如果出现多个匹配的重载函数，则会报错。
- 库函数：
    - 库函数是一种特殊的合约，为了提升`solidity`代码的复用性和减少`gas`而存在。库合约一般都是一些好用的函数合集（`库函数`），由大神或者项目方创作，咱们站在巨人的肩膀上，会用就行了。
    - 和普通合约的不同：
        1. 不能存在状态变量
        2. 不能够继承或被继承
        3. 不能接收以太币
        4. 不可以被销毁
    - String库合约：将uint256转换为相应的string类型的代码库，主要包含两个函数，`toString()`将`uint256`转为`string`，`toHexString()`
    将`uint256`转换为`16进制`，再转换为`string`
    - 如何使用库合约：用String库函数的toHexString()来演示两种使用库合约中函数的办法。
        - 利用using for指令：指令`using A for B;`可用于附加库函数（从库 A）到任何类型（B）。
            - 代码：
                
                ```solidity
                // 利用using for指令
                    using Strings for uint256;
                    function getString1(uint256 _number) public pure returns(string memory){
                        // 库函数会自动添加为uint256型变量的成员
                        return _number.toHexString();
                    }
                ```
                
        - 通过库合约名称调用库函数
            - 代码：
                
                ```solidity
                // 直接通过库合约名调用
                    function getString2(uint256 _number) public pure returns(string memory){
                        return Strings.toHexString(_number);
                    }
                ```
                
- import用法：引用(`import`)在代码中的位置为：在声明版本号之后，在其余代码之前。
    - 通过源文件相对位置导入
        
        ```solidity
        文件结构
        ├── Import.sol
        └── Yeye.sol
        
        // 通过文件相对位置import
        import './Yeye.sol';
        ```
        
    - 通过源文件网址导入网上的合约
        
        ```solidity
        // 通过网址引用
        import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol';
        ```
        
    - 通过`npm`的目录导入
        
        ```solidity
        import '@openzeppelin/contracts/access/Ownable.sol';
        ```
        
    - 通过`全局符号`import {Yeye} from './Yeye.sol';导入特定的合约
        
        ```solidity
        import {Yeye} from './Yeye.sol';
        ```
        
- `Solidity`支持两种特殊的回调函数，`receive()`和`fallback()`，他们主要在两种情况下被使用：
    1. 接收ETH
    2. 处理合约中不存在的函数调用（代理合约proxy contract）
    
    ps：在solidity 0.6.x版本之前，语法上只有 `fallback()`函数，用来接收用户发送的ETH时调用以及在被调用函数签名没有匹配到时，来调用。 0.6版本之后，solidity才将 `fallback()`函数拆分成 `receive()`和 `fallback()`两个函数。
    
- 接收ETH函数 receive：
    - `receive()`只用于处理接收`ETH`。一个合约最多有一个`receive()`函数，声明方式与一般函数不一样，不需要`function`关键字：`receive() external payable { ... }`。`receive()`函数不能有任何的参数，不能返回任何值，必须包含`external`和`payable`。
    - 当合约接收ETH的时候，`receive()`会被触发。`receive()`最好不要执行太多的逻辑因为如果别人用`send`和`transfer`方法发送`ETH`的话，`gas`会限制在`2300`，`receive()`太复杂可能会触发`Out of Gas`报错；如果用`call`就可以自定义`gas`执行更复杂的逻辑（这三种发送ETH的方法我们之后会讲到）。
    - 有些恶意合约，会在`receive()`函数（老版本的话，就是 `fallback()`函数）嵌入恶意消耗`gas`的内容或者使得执行故意失败的代码，导致一些包含退款和转账逻辑的合约不能正常工作，因此写包含退款等逻辑的合约时候，一定要注意这种情况。
- 回退函数 fallback：
    
    `fallback()`函数会在调用合约不存在的函数时被触发。可用于接收ETH，也可以用于代理合约`proxy contract`。`fallback()`声明时不需要`function`关键字，必须由`external`修饰，一般也会用`payable`修饰，用于接收ETH:`fallback() external payable { ... }`。
    
- receive和fallback的区别：
    - 触发规则：
        
        ```solidity
        触发fallback() 还是 receive()?
                   接收ETH
                      |
                 msg.data是空？
                    /  \
                  是    否
                  /      \
        receive()存在?   fallback()
                / \
               是  否
              /     \
        receive()   fallback()
        ```
        
    - 简单来说，合约接收`ETH`时，`msg.data`为空且存在`receive()`时，会触发`receive()`；`msg.data`不为空或不存在`receive()`时，会触发`fallback()`
    ，此时`fallback()`必须为`payable`。
    - `receive()`和`payable fallback()`均不存在的时候，向合约**直接**发送`ETH`
    将会报错（你仍可以通过带有`payable`的函数向合约发送`ETH`）。
- solidity的三种向其他合约发送ETH：`transfer()`，`send()`和`call()`，其中`call()`是被鼓励的用法。
    - transfer：
        - 用法是`接收方地址.transfer(发送ETH数额)`。
        - `transfer()`的`gas`限制是`2300`，足够用于转账，但对方合约的`fallback()`或`receive()`函数不能实现太复杂的逻辑。
        - • `transfer()`如果转账失败，会自动`revert`（回滚交易）。
    - send：
        - 用法是`接收方地址.send(发送ETH数额)`。
        - send()的`gas`限制是`2300`，足够用于转账，但对方合约的`fallback()`
        或`receive()`函数不能实现太复杂的逻辑。
        - `send()`如果转账失败，不会`revert`。
        - `send()`的返回值是`bool`，代表着转账成功或失败，需要额外代码处理一下。
    - call：
        - 用法是`接收方地址.call{value: 发送ETH数额}("")`。
        - `call()`没有`gas`限制，可以支持对方合约`fallback()`或`receive()`函数实现复杂逻辑。
        - `call()`如果转账失败，不会`revert`。
        - `call()`的返回值是`(bool, data)`，其中`bool`代表着转账成功或失败，需要额外代码处理一下。
- 调用其他合约：
    - 开发者写智能合约来调用其他合约，这让以太坊网络上的程序可以复用，从而建立繁荣的生态。很多`web3`项目依赖于调用其他合约。
    - 我们可以利用合约的地址和合约代码（或接口）来创建合约的引用：`_Name(_Address)`，其中`_Name`是合约名，`_Address`是合约地址。然后用合约的引用来调用它的函数：`_Name(_Address).f()`，其中`f()`是要调用的函数。
    - 4种调用合约的例子：
        - 传入合约地址
            
            我们可以在函数里传入目标合约地址，生成目标合约的引用，然后调用目标函数。
            
            ```solidity
            function callSetX(address _Address, uint256 x) external{
                    OtherContract(_Address).setX(x);
                }
            ```
            
        - 传入合约变量
            
            我们可以直接在函数里传入合约的引用，只需要把上面参数的`address`
            类型改为目标合约名，比如`OtherContract`
            
            ```solidity
            function callGetX(OtherContract _Address) external view returns(uint x){
                    x = _Address.getX();
                }
            ```
            
        - 创建合约变量
            
            我们可以创建合约变量，然后通过它来调用目标函数。
            
            ```solidity
            function callGetX2(address _Address) external view returns(uint x){
                    OtherContract oc = OtherContract(_Address);
                    x = oc.getX();
                }
            ```
            
        - 调用合约并发送ETH
            
            如果目标合约的函数是`payable`的，那么我们可以通过调用它来给合约转账：`_Name(_Address).f{value: _Value}()`，其中`_Name`是合约名，`_Address`是合约地址，`f`是目标函数名，`_Value`是要转的`ETH`数额（以`wei`为单位）。
            
            ```solidity
            function setXTransferETH(address otherContract, uint256 x) payable external{
                    OtherContract(otherContract).setX{value: msg.value}(x);
                }
            ```
            
- call：
    - `call` 是`address`类型的低级成员函数，它用来与其他合约交互。它的返回值为`(bool, data)`，分别对应`call`是否成功以及目标函数的返回值。