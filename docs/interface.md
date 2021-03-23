## CALL接口


### Bank.sol （计算【ETH】利息ERC20合约）

```js
// 普通公共函数
// 获取下一轮次的未结算的利息
function pendingInterest(uint256 msgValue) public view returns (uint256)

// 通过给定债务份额返回对应的（ETH）债务价值 （需要小心未计利息）
function debtShareToVal(uint256 debtShare) public view returns (uint256)

// 通过给定债务价值返回对应的债务份额 （需要小心未计利息）
function debtValToShare(uint256 debtVal) public view returns (uint256)

// 通过用户地址对应的位置（position）获取债务份额和债务价值（需要小心未计利息）
function positionInfo(uint256 id) public view returns (uint256, uint256) 

// 获取合约中所有token持有者存入的ETH个数（需要小心未计利息）
function totalETH() public view returns (uint256) 


// 修饰器，用于在调用函数之前的鉴权或计算
// 管理员权限可操作
modifier onlyOwner()

// 判断是有调用过，防止直接或间接的重入操作
modifier nonReentrant() 

// 保证合约调用者是交易发送者账户，用于避免闪电贷攻击
modifier onlyEOA()

// 根据当前时间（now）更新 reservePool（储蓄池）， glbDebtVal（全局债务价值）， lastAccrueTime（下次计息时间）
modifier accrue(uint256 msgValue) 


// 带修饰器的公共函数
// 储蓄，存入ETH，获取对应的债权份额
function deposit() external payable accrue(msg.value) nonReentrant

// 提现，销毁债权份额，获取对应的ETH
function withdraw(uint256 share) external accrue(0) nonReentrant

// 创建一个新的挖矿位置（position），解锁用户的产生收益潜力
// id 挖矿获取收益的位置，新加入的填0 （老用户用于更新，新用户用于创建）
// goblin 挖矿的用户地址
// loan 向全局池子中借的ETH个数
// maxReturn 从全局池子中最大可以借的ETH个数
// data 用户地址对应的其他数据信息
function work(uint256 id, address goblin, uint256 loan, uint256 maxReturn, bytes calldata data)
        external payable
        onlyEOA accrue(msg.value) nonReentrant

// 清算，根据位置（position）来触发用户的清算操作
function kill(uint256 id) external onlyEOA accrue(0) nonReentrant 

// ======== 以下合约函数无需前端来调用 ========
// 更新管理员的配置账户地址 （管理员操作）
function updateConfig(BankConfig _config) external onlyOwner

// 转移储蓄池中（value）数量的ETH到指定地址（to）（管理员操作）
function withdrawReserve(address to, uint256 value) external onlyOwner nonReentrant

// 减少储蓄池中（value）数量的ETH （管理员操作）
function reduceReserve(uint256 value) external onlyOwner

// 恢复意外发送到此智能合约的ERC20代币 （管理员操作）
function recover(address token, address to, uint256 value) external onlyOwner nonReentrant
```


### SimpleBankConfig.sol （存储借贷配置合约）

```js
// 返回每秒的利率，使用 1e18 作为基本单位.
function getInterestRate(uint256 debt, uint256 floating) external view returns (uint256);

// 查看用户地址是否在挖矿池子中.
function isGoblin(address goblin) external view returns (bool);

// 查看用户地址是否能接受更多的债务.
function acceptDebt(address goblin) external view returns (bool);

// 查看用户地址的借贷操作因子，使用 1e4 作为基本单位.
function workFactor(address goblin, uint256 debt) external view returns (uint256);

// 查看用户地址的清算因子，使用 1e4 作为基本单位. 
function killFactor(address goblin, uint256 debt) external view returns (uint256);
```


### StrategyAllETHOnly.sol （ETH的挖矿策略合约）

```js
// 选择挖矿策略，获取用户的ETH，将等比例份额的 ETH 转换成 tokens，并添加 tokens 和 ETH 流动性，转换成流动性代币 tokens + ETH.
function execute(address /* user */, uint256 /* debt */, bytes calldata data)
    external
    payable
    nonReentrant

// 恢复意外发送到此智能合约的ERC20代币 （管理员操作）
function recover(address token, address to, uint256 value) external onlyOwner nonReentrant
```


### StrategyLiquidate.sol （ETH的清算策略合约）

```js
// 选择清算策略，触发清算操作，提取流动性代币 tokens 和 ETH，将 tokens 兑换成 ETH，将清算的 ETH 给管理员账户.
function execute(address /* user */, uint256 /* debt */, bytes calldata data)
    external
    payable
    nonReentrant

// 恢复意外发送到此智能合约的ERC20代币 （管理员操作）
function recover(address token, address to, uint256 value) external onlyOwner nonReentrant
```


### StrategyLiquidate.sol （ETH的清算策略合约）

```js
// 选择清算策略，触发清算操作，提取流动性代币 tokens 和 ETH，将 tokens 兑换成 ETH，将清算的 ETH 给管理员账户.
function execute(address /* user */, uint256 /* debt */, bytes calldata data)
    external
    payable
    nonReentrant

// 恢复意外发送到此智能合约的ERC20代币 （管理员操作）
function recover(address token, address to, uint256 value) external onlyOwner nonReentrant
```



### UniswapGoblin.sol （挖矿清算的内部操作合约）

```js
// 构造函数
constructor(
    address _operator,          // bank（储蓄代币）合约地址
    IStakingRewards _staking,   // 抵押获息奖励（synthetix）合约地址
    IUniswapV2Router02 _router, // uniswap2路由器合约地址
    address _fToken,            // 流动性代币token合约地址
    address _uni,               // uniswap合约地址
    Strategy _addStrat,         // 储蓄ETH并添加流动性的合约地址
    Strategy _liqStrat,         // 清算合约地址
    uint256 _reinvestBountyBps  // 重新投资赏金比率
) public

// 修饰器，用于在调用函数之前的鉴权或计算
// 保证合约调用者是交易发送者账户，用于避免闪电贷攻击
modifier onlyEOA()

// 操作合约的地址（bank合约地址）
modifier onlyOperator()

// 通过给定的流动性代币数量计算出对应的债权份额
function shareToBalance(uint256 share) public view returns (uint256)

// 通过债权份额计算出对应的流动性代币数量
function balanceToShare(uint256 balance) public view returns (uint256)

// 将获取的奖励重新投资到池子中
function reinvest() public onlyEOA nonReentrant 

// 通过地址位置（position）执行挖矿操作
function work(uint256 id, address user, uint256 debt, bytes calldata data)
    external payable
    onlyOperator nonReentrant

// 根据Uniswap流动性池子的状态，通过输入in的个数，计算得到的out个数
function getMktSellAmount(uint256 aIn, uint256 rIn, uint256 rOut) public pure returns (uint256) 

// 如果执行清算操作，计算可获取的ETH个数
function health(uint256 id) external view returns (uint256)

// 用户触发清算操作
function liquidate(uint256 id) external onlyOperator nonReentrant

// ======== 以下合约函数无需前端来调用 ========
// 恢复意外发送到此智能合约的ERC20代币 （管理员操作）
function recover(address token, address to, uint256 value) external onlyOwner nonReentrant

// 更新重新投资赏金的比率
function setReinvestBountyBps(uint256 _reinvestBountyBps) external onlyOwner

// 更新策略合约状态
function setStrategyOk(address[] calldata strats, bool isOk) external onlyOwner

// 重新设置挖矿合约和清算合约
function setCriticalStrategies(Strategy _addStrat, Strategy _liqStrat) external onlyOwner
```


### UniswapGoblinConfig.sol （UniswapGoblin的配置合约）

（测试合约中暂未启用）