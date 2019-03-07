# 井通智能合约公测说明

井通solidity版本的ERC20智能合约开发已进入测试阶段，现邀请广大开发者们公开测试，使用说明如下：
公测节点地址：ws://123.57.209.177:5030  
整个合约包含编译、部署和调用三步，步骤如下：

## 1.创建solidity合约并编译
下面我们创建一个发行ERC20代币的合约代码：

```
pragma solidity ^0.4.19;
contract TokenTest {
    string public name;
    string public symbol;
    uint8 public decimals = 18;  // decimals 可以有的小数点个数，最小的代币单位。18 是建议的默认值
    uint256 public totalSupply;
    // 用mapping保存每个地址对应的余额
    mapping (address => uint256) public balanceOf;
    // 存储对账号的控制
    mapping (address => mapping (address => uint256)) public allowance;
    /**
     * 初始化构造
     */
    function TokenTest(uint256 initialSupply, string tokenName, string tokenSymbol) public {
        totalSupply = initialSupply * 10 ** uint256(decimals);  // 供应的份额，份额跟最小的代币单位有关，份额 = 币数 * 10 ** decimals。
        balanceOf[msg.sender] = totalSupply;
        name = tokenName;                                   // 代币名称
        symbol = tokenSymbol;                               // 代币符号
    }

        /**
     * 代币交易转移的内部实现
     */
    function _transfer(address _from, address _to, uint _value) internal {
        // 确保目标地址不为0x0，因为0x0地址代表销毁
        require(_to != 0x0);
        // 检查发送者余额
        require(balanceOf[_from] >= _value);
        // 确保转移为正数个
        require(balanceOf[_to] + _value > balanceOf[_to]);
        // 以下用来检查交易，
        uint previousBalances = balanceOf[_from] + balanceOf[_to];
        // Subtract from the sender
        balanceOf[_from] -= _value;
        // Add the same to the recipient
        balanceOf[_to] += _value;
        // 用assert来检查代码逻辑。
        assert(balanceOf[_from] + balanceOf[_to] == previousBalances);
    }
    /**
     *  代币交易转移
     * 从自己（创建交易者）账号发送`_value`个代币到 `_to`账号
     * @param _to 接收者地址
     * @param _value 转移数额
     */
    function transfer(address _to, uint256 _value) public {
        _transfer(msg.sender, _to, _value);
}
    function() public {
        revert();
    }
}

```
编译以上的合约代码(可通过https://remix.ethereum.org/ 在线编译)，生成EVM的bytecode如下：

```
60606040526012600260006101000a81548160ff021916908360ff160217905550341561002b57600080fd5b60405161092738038061092783398101604052808051906020019091908051820191906020018051820191905050600260009054906101000a900460ff1660ff16600a0a8302600381905550600354600460003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000208190555081600090805190602001906100d39291906100f3565b5080600190805190602001906100ea9291906100f3565b50505050610198565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061013457805160ff1916838001178555610162565b82800160010185558215610162579182015b82811115610161578251825591602001919060010190610146565b5b50905061016f9190610173565b5090565b61019591905b80821115610191576000816000905550600101610179565b5090565b90565b610780806101a76000396000f300606060405260043610610083576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806306fdde031461009357806318160ddd14610121578063313ce5671461014a57806370a082311461017957806395d89b41146101c6578063a9059cbb14610254578063dd62ed3e14610296575b341561008e57600080fd5b600080fd5b341561009e57600080fd5b6100a6610302565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156100e65780820151818401526020810190506100cb565b50505050905090810190601f1680156101135780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b341561012c57600080fd5b6101346103a0565b6040518082815260200191505060405180910390f35b341561015557600080fd5b61015d6103a6565b604051808260ff1660ff16815260200191505060405180910390f35b341561018457600080fd5b6101b0600480803573ffffffffffffffffffffffffffffffffffffffff169060200190919050506103b9565b6040518082815260200191505060405180910390f35b34156101d157600080fd5b6101d96103d1565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156102195780820151818401526020810190506101fe565b50505050905090810190601f1680156102465780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b341561025f57600080fd5b610294600480803573ffffffffffffffffffffffffffffffffffffffff1690602001909190803590602001909190505061046f565b005b34156102a157600080fd5b6102ec600480803573ffffffffffffffffffffffffffffffffffffffff1690602001909190803573ffffffffffffffffffffffffffffffffffffffff1690602001909190505061047e565b6040518082815260200191505060405180910390f35b60008054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156103985780601f1061036d57610100808354040283529160200191610398565b820191906000526020600020905b81548152906001019060200180831161037b57829003601f168201915b505050505081565b60035481565b600260009054906101000a900460ff1681565b60046020528060005260406000206000915090505481565b60018054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156104675780601f1061043c57610100808354040283529160200191610467565b820191906000526020600020905b81548152906001019060200180831161044a57829003601f168201915b505050505081565b61047a3383836104a3565b5050565b6005602052816000526040600020602052806000526040600020600091509150505481565b6000808373ffffffffffffffffffffffffffffffffffffffff16141515156104ca57600080fd5b81600460008673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020541015151561051857600080fd5b600460008473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205482600460008673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002054011115156105a657600080fd5b600460008473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002054600460008673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205401905081600460008673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000206000828254039250508190555081600460008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000206000828254019250508190555080600460008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002054600460008773ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020540114151561074e57fe5b505050505600a165627a7a72305820b2665df0d8d8522803a19ac6bc98ff010121e11c16d0342eaced01d94100ce180029
```
同时生成的abi如下：

```
[
	{
		"constant": true,
		"inputs": [],
		"name": "name",
		"outputs": [
			{
				"name": "",
				"type": "string"
			}
		],
		"payable": false,
		"stateMutability": "view",
		"type": "function"
	},
	{
		"constant": true,
		"inputs": [],
		"name": "totalSupply",
		"outputs": [
			{
				"name": "",
				"type": "uint256"
			}
		],
		"payable": false,
		"stateMutability": "view",
		"type": "function"
	},
	{
		"constant": true,
		"inputs": [],
		"name": "decimals",
		"outputs": [
			{
				"name": "",
				"type": "uint8"
			}
		],
		"payable": false,
		"stateMutability": "view",
		"type": "function"
	},
	{
		"constant": true,
		"inputs": [
			{
				"name": "",
				"type": "address"
			}
		],
		"name": "balanceOf",
		"outputs": [
			{
				"name": "",
				"type": "uint256"
			}
		],
		"payable": false,
		"stateMutability": "view",
		"type": "function"
	},
	{
		"constant": true,
		"inputs": [],
		"name": "symbol",
		"outputs": [
			{
				"name": "",
				"type": "string"
			}
		],
		"payable": false,
		"stateMutability": "view",
		"type": "function"
	},
	{
		"constant": false,
		"inputs": [
			{
				"name": "_to",
				"type": "address"
			},
			{
				"name": "_value",
				"type": "uint256"
			}
		],
		"name": "transfer",
		"outputs": [],
		"payable": false,
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"constant": true,
		"inputs": [
			{
				"name": "",
				"type": "address"
			},
			{
				"name": "",
				"type": "address"
			}
		],
		"name": "allowance",
		"outputs": [
			{
				"name": "",
				"type": "uint256"
			}
		],
		"payable": false,
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"name": "initialSupply",
				"type": "uint256"
			},
			{
				"name": "tokenName",
				"type": "string"
			},
			{
				"name": "tokenSymbol",
				"type": "string"
			}
		],
		"payable": false,
		"stateMutability": "nonpayable",
		"type": "constructor"
	},
	{
		"payable": false,
		"stateMutability": "nonpayable",
		"type": "fallback"
	}
]
```
## 2.部署合约
bytecode和abi生成之后，我们引入jingtum-lib库，通过initContract方法部署合约，代码如下：
```
var jlib = require('jingtum-lib');
var Remote = jlib.Remote;
var remote = new Remote({server: 'ws://123.57.209.177:5030', local_sign:true});
remote.connect(function (err, result) {
    if (err) {
        return console.log('err:', err);
    }
     var v = {
        secret: 's...UTb',
        address: 'j...yTh'
};
var req = remote.initContract({
	 account: v.address,
    amount: 10,
    payload: bytecode,//上面solidity编译生成的bytecode
    abi: abi,//上面solidity编译生成的abi
    params:[2000, 'TestCurrency', 'TEST1']
});
    req.setSecret(v.secret);
    tx.submit(function (err, result) {
        if (err) {
            console.log('err:', err);
        }
        else if (result) {
            console.log('res:', result);
        }
    });
});
```
如果成功，结果中会返回一个合约账号，供后面的调用使用。具体接口说明详见接口文档。

## 3.调用合约
调用通部署一样，都是引入jingtum-lib库，然后通过invokeContract方法调用合约，代码如下：
```
var jlib = require('jingtum-lib');
var Remote = jlib.Remote;
var remote = new Remote({server: 'ws://123.57.209.177:5030', local_sign:true});
remote.connect(function (err, result) {
    if (err) {
        return console.log('err:', err);
    }
     var v = {
        secret: 's...UTb',
        address: 'j...yTh'
};
var req = remote.invokeContract({
	 account: v.address,
    destination: 'jPZ1....9Kkh', //部署返回的合约地址
    abi: abi,//solidity合约编译生成的abi
    func:"transfer('jPZ1....9Kkh', 15)"});//调用合约的某个方法
    req.setSecret(v.secret);
    tx.submit(function (err, result) {
        if (err) {
            console.log('err:', err);
        }
        else if (result) {
            console.log('res:', result);
        }
    });
});
```
其中，func参数是具体调用的合约方法，包含合约名及参数，如本例子中给某个账号发币表示为”transfer(jPZ1....9Kkh, 15)”；若无参数则不需要传，但函数的括号必须写全。
