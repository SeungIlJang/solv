Trust Fund 
==

```solidity
pragma solidity 0.4.24;

import "../CtfFramework.sol";
import "../../node_modules/openzeppelin-solidity/contracts/math/SafeMath.sol";

contract TrustFund is CtfFramework{ //CtfFramework를 상속

    using SafeMath for uint256; //uint256에 SafeMath 사용

    uint256 public allowancePerYear;
    uint256 public startDate;
    uint256 public numberOfWithdrawls;
    bool public withdrewThisYear;
    address public custodian;

    constructor(address _ctfLauncher, address _player) public payable
        CtfFramework(_ctfLauncher, _player)
    {
        custodian = msg.sender; // contract를 생성한 address를 대입
        allowancePerYear = msg.value.div(10);//생성시에 보낸 eth를 10으로 나눈값을 대입        
        startDate = now; //현재시간을 시작일자에 대입
    }

    function checkIfYearHasPassed() internal{
        if (now>=startDate + numberOfWithdrawls * 365 days){
            withdrewThisYear = false;
        } 
    }

    function withdraw() external ctf{
        require(allowancePerYear > 0, "No Allowances Allowed");
        checkIfYearHasPassed();
        require(!withdrewThisYear, "Already Withdrew This Year");
        /*
        msg.sender.call.value(x)()는 x만큼 msg.sender에게 이더를 보내고 남은 가스를 넘겨줍니다. 
        만약 msg.sender가 악의적으로 작성된 스마트 컨트랙트일 경우 withdraw()를 다시 호출하는 re-entracy 공격을 할 수 있는데, 
        이 경우 코드에 따르면 msg.sender에게 계속 이더를 보내게 되고, 
        가스가 부족해지거나 Fund에 잔고가 다 떨어졌다고 판단되면 더 이상 withdraw()를 호출하지 않고 정상 종료하여 트랜잭션을 성공시킬 수 있습니다. 
        360만 이더가 탈취된 DAO 해킹이 이 취약점이 공격된 사례
        withdrewThisYear = true; 이 실행되기 전에 
        withdraw() 가 재귀 호출함으로써 자신의 보유한 잔고보다 더 많은 잔고를 인출해 나갈수 있습니다.
        */

        if (msg.sender.call.value(allowancePerYear)()){
            withdrewThisYear = true; //현재년도의 출금 상태 
            numberOfWithdrawls = numberOfWithdrawls.add(1); //출금 횟수 추가 
        }
    }
    
    function returnFunds() external payable ctf{
        require(msg.value == allowancePerYear, "Incorrect Transaction Value"); //10개월로 나눈 값인지 확인
        require(withdrewThisYear==true, "Cannot Return Funds Before Withdraw"); //출금이 완료되어야 입금 가능
        withdrewThisYear = false;
        numberOfWithdrawls=numberOfWithdrawls.sub(1); //출금 횟수 빼기 
    }
}
```
