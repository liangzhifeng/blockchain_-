# blockchain_grab
本以太坊智能合约是使用抓阄的方式，将6个球队，分成A、B、C三个小组

pragma solidity ^0.4.20;

// 本以太坊智能合约是使用抓阄的方式，将6个球队，分成A、B、C三个小组
contract Grab {

    // 球队结构体
    struct Team {
        uint weight;//是否已授权获得抓阄权限
        bool grabed;//是否完成抓阄动作
    }
 
    // 6个阄的名称
    string[] lotNameList;

    // 主席或组织者，本程序约定该人也可以获得抓阄权限
    address public chairperson;
    
    // 地址和球队的映射 
    mapping(address => Team) teams; 
    
    // 名称和球队地址的映射
    mapping(string => address[]) nameAddressMapping;
    
    // 已经抓阄的数量
    uint selectCount; 
    
    // 已经给球队赋予抓阄权限的数量，最多只能给6个球队赋予抓阄权限
    uint rightNumber; 

    // 构造方法
    constructor() public {
        chairperson = msg.sender;
        lotNameList = ['A','A','B','B','C','C'];
        selectCount = 0;
        rightNumber = 0;
    }

    // 给球队赋予抓阄的权限
    function giveRightToGrab(address _toGrab) public {
        require(
            msg.sender == chairperson && 
            !teams[_toGrab].grabed &&
            teams[_toGrab].weight == 0 &&
            rightNumber < 6 // 最多只能给6个球队赋予抓阄权限
            );
        teams[_toGrab].weight = 1;
        rightNumber++;
    }

    // 抓阄
    function grab() public{
        Team storage sender = teams[msg.sender];

        require(sender.weight == 1);
        require(!sender.grabed);
     
        uint randomNumber = uint(keccak256(now,msg.sender,10))%(6-selectCount);
        nameAddressMapping[lotNameList[randomNumber]].push(msg.sender);
        selectCount++;
        sender.grabed = true;
        deleteStrAt(randomNumber);
    }

    // 判断抓阄是否已经结束
    function isEnded() public view returns (bool _isEnded) {
           _isEnded = (selectCount == 6);
    }
    
    // 查询分组结果，请求参数只能是"A"、"B"、"C"其中一个，string类型
    function queryResult(string _str) public view returns (address[] _result) {
         require(selectCount == 6);
         // 利用hash值判断字符串是否相等
         require(keccak256(_str) == keccak256("A") || keccak256(_str) == keccak256("B") || keccak256(_str) == keccak256("C"));
         _result =  nameAddressMapping[_str];
    }
    
    // 删除数组的某个元素，私有方法
    function deleteStrAt (uint _index) private {
        uint len = lotNameList.length;
        require(_index < len);
        for (uint i = _index; i<len-1; i++) {
          lotNameList[i] = lotNameList[i+1];
        }
    
        delete lotNameList[len-1];
        lotNameList.length--;
  }

}
