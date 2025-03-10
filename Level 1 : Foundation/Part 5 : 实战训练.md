1.存钱罐合约

// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;
contract Bank {
    address public immutable owner;
    event Deposit(address indexed sender, uint256 amount);
    event Withdraw(address indexed owner, uint256 amount);
    
    constructor() payable {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not the contract owner");
        _;
    }
    function deposit() public payable {
        require(msg.value > 0, "Must send some Ether");
        emit Deposit(msg.sender, msg.value);
    }
    function withdraw(uint256 amount) public onlyOwner {
        require(amount <= address(this).balance, "Insufficient balance");
        payable(owner).transfer(amount);
        emit Withdraw(owner, amount);   
        selfdestruct(payable(owner));
    }
    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}


2.WETH合约

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;
contract WETH {
    string public name = "Wrapped Ether";
    string public symbol = "WETH";
    uint8 public decimals = 18;
    event Approval(address indexed src, address indexed delegateAds, uint256 amount);
    event Transfer(address indexed src, address indexed toAds, uint256 amount);
    event Deposit(address indexed toAds, uint256 amount);
    event Withdraw(address indexed src, uint256 amount);
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    function deposit() public payable {
        balanceOf[msg.sender] += msg.value;
        emit Deposit(msg.sender, msg.value);
    }
    function withdraw(uint256 amount) public {
        require(balanceOf[msg.sender] >= amount);
        balanceOf[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
        emit Withdraw(msg.sender, amount);
    }
    function totalSupply() public view returns (uint256) {
        return address(this).balance;
    }
    function approve(address delegateAds, uint256 amount) public returns (bool) {
        allowance[msg.sender][delegateAds] = amount;
        emit Approval(msg.sender, delegateAds, amount);
        return true;
    }
    function transfer(address toAds, uint256 amount) public returns (bool) {
        return transferFrom(msg.sender, toAds, amount);
    }
    function transferFrom(
        address src,
        address toAds,
        uint256 amount
    ) public returns (bool) {
        require(balanceOf[src] >= amount);
        if (src != msg.sender) {
            require(allowance[src][msg.sender] >= amount);
            allowance[src][msg.sender] -= amount;
        }
        balanceOf[src] -= amount;
        balanceOf[toAds] += amount;
        emit Transfer(src, toAds, amount);
        return true;
    }
    fallback() external payable {
        deposit();
    }
    receive() external payable {
        deposit();
    }
}


3.TodoList合约

// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;
contract Demo {
    struct Todo {
        string name;
        bool isCompleted;
    }
    Todo[] public list;
    function creat(string memory _name) external {
        list.push(
            Todo ({
                name : _name,
                isCompleted : false
            })
        );
    }
    function rename(uint256 _index, string memory _name) external {
        //修改一个
        list[_index].name = _name;
        //修改多个
        // Todo storage temp = list[_index];
        // list[_index].name = _name;
    }
    function modiStatus(uint256 _index, bool _status) external  {
        //手动修改
        list[_index].isCompleted = _status;
        //自动修改
        // list[_index].isCompleted = !list[_index].isCompleted;
    }
    function get(uint256 index_) external view returns(string memory name_,bool status_){
        Todo storage temp = list[index_];
        return (temp.name,temp.isCompleted);
    }
}


4.众筹合约

// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;
contract Demo {
    address public beneficiary;
    uint256 public fundingGoal;
    uint256 public fundingAmount;
    address[] public fundersKey;
    mapping (address => uint256) public funders;
    event Funded(address indexed funder, uint256 amount);
    event FundingGoalReached(uint256 amount);
    modifier onlyBeneficiary() {
        require(msg.sender == beneficiary);
        _;
    }
    constructor(address _beneficiary, uint256 _fundingGoal) {
        beneficiary = _beneficiary;
        fundingGoal = _fundingGoal;
        fundingAmount = 0;
    }
    function fund() public payable {
        require(msg.value > 0, "Donation must be greater than 0.");
        if (funders[msg.sender] == 0) {
            fundersKey.push(msg.sender);
        }
        funders[msg.sender] += msg.value;
        fundingAmount += msg.value;
        emit Funded(msg.sender, msg.value);
        if (fundingAmount >= fundingGoal) {
            emit FundingGoalReached(fundingAmount);
        }
    }
    function withdraw() public onlyBeneficiary {
        require(fundingAmount >= fundingGoal, "Funding goal not reached.");
        payable(beneficiary).transfer(fundingAmount);
        fundingAmount = 0;
    }
    // 查看合约中当前的募集金额
    function getFundingAmount() public view returns (uint256) {
        return fundingAmount;
    }

    // 查看资助者列表
    function getFunders() public view returns (address[] memory) {
        return fundersKey;
    }

    // 查看某个资助者的捐款金额
    function getFunderContribution(address _funder) public view returns (uint256) {
        return funders[_funder];
    }

    // 查看剩余的筹资目标（用于UI展示）
    function remainingGoal() public view returns (uint256) {
        if (fundingAmount >= fundingGoal) {
            return 0;
        }
        return fundingGoal - fundingAmount;
    }
}


5.ETH钱包

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;
contract EtherWallet {
    address payable public immutable owner;
    event Log(string funName, address from, uint256 value, bytes data);
    constructor() {
        owner = payable(msg.sender);
    }
    receive() external payable {
        emit Log("receive", msg.sender, msg.value, "");
    }
    function withdraw1() external {
        require(msg.sender == owner, "Not owner");
        payable(msg.sender).transfer(100);
    }
    function withdraw2() external {
        require(msg.sender == owner, "Not owner");
        bool success = payable(msg.sender).send(200);
        require(success, "Send Failed");
    }
    function withdraw3() external {
        require(msg.sender == owner, "Not owner");
        (bool success, ) = msg.sender.call{value: address(this).balance}("");
        require(success, "Call Failed");
    }
    function getBalance() external view returns (uint256) {
        return address(this).balance;
    }
}


6.多签钱包

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;
contract MultiSigWallet {
    // 状态变量
    address[] public owners;
    mapping(address => bool) public isOwner;
    uint256 public required;
    struct Transaction {
        address to;
        uint256 value;
        bytes data;
        bool exected;
    }
    Transaction[] public transactions;
    mapping(uint256 => mapping(address => bool)) public approved;
    // 事件
    event Deposit(address indexed sender, uint256 amount);
    event Submit(uint256 indexed txId);
    event Approve(address indexed owner, uint256 indexed txId);
    event Revoke(address indexed owner, uint256 indexed txId);
    event Execute(uint256 indexed txId);
    // receive
    receive() external payable {
        emit Deposit(msg.sender, msg.value);
    }
    // 函数修改器
    modifier onlyOwner() {
        require(isOwner[msg.sender], "not owner");
        _;
    }
    modifier txExists(uint256 _txId) {
        require(_txId < transactions.length, "tx doesn't exist");
        _;
    }
    modifier notApproved(uint256 _txId) {
        require(!approved[_txId][msg.sender], "tx already approved");
        _;
    }
    modifier notExecuted(uint256 _txId) {
        require(!transactions[_txId].exected, "tx is exected");
        _;
    }
    // 构造函数
    constructor(address[] memory _owners, uint256 _required) {
        require(_owners.length > 0, "owner required");
        require(
            _required > 0 && _required <= _owners.length,
            "invalid required number of owners"
        );
        for (uint256 index = 0; index < _owners.length; index++) {
            address owner = _owners[index];
            require(owner != address(0), "invalid owner");
            require(!isOwner[owner], "owner is not unique"); // 如果重复会抛出错误
            isOwner[owner] = true;
            owners.push(owner);
        }
        required = _required;
    }
    // 函数
    function getBalance() external view returns (uint256) {
        return address(this).balance;
    }
    function submit(
        address _to,
        uint256 _value,
        bytes calldata _data
    ) external onlyOwner returns(uint256){
        transactions.push(
            Transaction({to: _to, value: _value, data: _data, exected: false})
        );
        emit Submit(transactions.length - 1);
        return transactions.length - 1;
    }
    function approv(uint256 _txId)
        external
        onlyOwner
        txExists(_txId)
        notApproved(_txId)
        notExecuted(_txId)
    {
        approved[_txId][msg.sender] = true;
        emit Approve(msg.sender, _txId);
    }
    function execute(uint256 _txId)
        external
        onlyOwner
        txExists(_txId)
        notExecuted(_txId)
    {
        require(getApprovalCount(_txId) >= required, "approvals < required");
        Transaction storage transaction = transactions[_txId];
        transaction.exected = true;
        (bool sucess, ) = transaction.to.call{value: transaction.value}(
            transaction.data
        );
        require(sucess, "tx failed");
        emit Execute(_txId);
    }
    function getApprovalCount(uint256 _txId)
        public
        view
        returns (uint256 count)
    {
        for (uint256 index = 0; index < owners.length; index++) {
            if (approved[_txId][owners[index]]) {
                count += 1;
            }
        }
    }
    function revoke(uint256 _txId)
        external
        onlyOwner
        txExists(_txId)
        notExecuted(_txId)
    {
        require(approved[_txId][msg.sender], "tx not approved");
        approved[_txId][msg.sender] = false;
        emit Revoke(msg.sender, _txId);
    }
}
