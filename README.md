# Assignment-3
คำสั่งตั่างๆๆที่คัญในการเขียน แอปพลิเคชัน Ecommerce ที่ทำงานร่วมกับ Blockchain 

คำสั่งสร้าง Smart Contract 
1. สำหรับ Smart Contract นี้ ผู้เขียนให้ประกอบด้วย 3 ฟังก์ชันด้วยกัน คือ

1.1 ฟังก์ชันเพิ่มสินค้าในฐานข้อมูล
1.2 ฟังก์ชันรับรายละเอียดของสินค้า
1.3 ฟังก์ชันสั่งซื้อสินค้า
โค้ดที่ได้จะเป็นดังด้านล่างค่ะ

    pragma solidity >=0.4.25 <0.6.0;
    import "./Token.sol";
    contract Shop 

    {
    struct Product
    
    {
        string name;
        string imgPath;
        uint256 price;
        uint256 quantity;
        address seller;
    }
    
    event AddedProduct(uint256 pid, address seller, uint256 timestamp);
    event BuyProduct(uint256 pid, address buyer, uint256 timestamp);
    mapping (uint256 => Product) products;
    mapping (uint256 => address[]) buying;
    Token token;
    
    constructor (address _tokenAddress) public {
        token = Token(_tokenAddress);
    }

    function addProduct(
        uint256 _pid,
        string memory _name,
        uint256 _price,
        uint256 _quantity,
        string memory _imgPath,
        uint256 timestamp
    ) public {
        products[_pid] = Product({
            name: _name,
            imgPath: _imgPath,
            price: _price,
            quantity: _quantity,
            seller: msg.sender
        });
        emit AddedProduct(_pid, msg.sender, timestamp);
    }

    function getProduct(uint256 _pid) public view returns (string memory, uint256, uint256, string memory, address) {
        Product memory product = products[_pid];
        return (product.name, product.price, product.quantity, product.imgPath, product.seller);
    }

    function buyProduct(uint256 _pid, uint256 _timestamp) public {
        require(products[_pid].quantity > 0, "Product is sold out");

        Product storage product = products[_pid];
        address _buyer = msg.sender;
        token.transfer(_buyer, product.seller, product.price);

        product.quantity -= 1;

        buying[_pid].push(_buyer);
        emit BuyProduct(_pid, _buyer, _timestamp);   }
 
2. ลังจากที่เราสร้าง Smart Contract เรียบร้อย เราจะต้อง Deploy ส่งไปที่ Ethereum เพื่อใช้งานต่อไปค่ะ
หาก Script ของเราจะ Deploy Smart Contract โปรเจค Simple Ecommerce ใช้ web3 เวอร์ชัน 1.0.0 Script ที่ได้จะเป็นดังนี้

    const Web3 = require('web3');
    const web3 = new Web3('http://localhost:8545');

    const tokenAbi = [{"constant":true,"inputs":[{"name":"","type":"address"}],"name":"balanceOf","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function","signature":"0x70a08231"},{"inputs":[{"name":"initialSupply","type":"uint256"}],"payable":false,"stateMutability":"nonpayable","type":"constructor","signature":"constructor"},{"constant":false,"inputs":[{"name":"_from","type":"address"},{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[{"name":"success","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function","signature":"0xbeabacc8"}]
    const shopAbi = [{"inputs":[{"name":"_tokenAddress","type":"address"}],"payable":false,"stateMutability":"nonpayable","type":"constructor","signature":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"pid","type":"uint256"},{"indexed":false,"name":"seller","type":"address"},{"indexed":false,"name":"timestamp","type":"uint256"}],"name":"AddedProduct","type":"event","signature":"0x283a98e56d9d477bc9675301834a627bc96a173f644ce87ab9bbeee35e192965"},{"anonymous":false,"inputs":[{"indexed":false,"name":"pid","type":"uint256"},{"indexed":false,"name":"buyer","type":"address"},{"indexed":false,"name":"timestamp","type":"uint256"}],"name":"BuyProduct","type":"event","signature":"0x6e963996302b87595be0bd0f2d400e2f43c1cfce4378ba7e9b34305fe2f2746b"},{"constant":false,"inputs":[{"name":"_pid","type":"uint256"},{"name":"_name","type":"string"},{"name":"_price","type":"uint256"},{"name":"_quantity","type":"uint256"},{"name":"_imgPath","type":"string"},{"name":"timestamp","type":"uint256"}],"name":"addProduct","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function","signature":"0xdd31ee57"},{"constant":true,"inputs":[{"name":"_pid","type":"uint256"}],"name":"getProduct","outputs":[{"name":"name","type":"string"},{"name":"price","type":"uint256"},{"name":"quantity","type":"uint256"},{"name":"imgPath","type":"string"},{"name":"seller","type":"address"}],"payable":false,"stateMutability":"view","type":"function","signature":"0xb9db15b4"},{"constant":false,"inputs":[{"name":"_pid","type":"uint256"},{"name":"_timestamp","type":"uint256"}],"name":"buyProduct","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function","signature":"0x53b62866"}]
    const tokenBytecode = 0x608060405234801561001057600080fd5b506040516020806103a08339810180604052602081101561003057600080fd5b8101908080519060200190929190505050806000803373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020819055505061030c806100946000396000f3fe60806040526004361061004c576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806370a0823114610051578063beabacc8146100b6575b600080fd5b34801561005d57600080fd5b506100a06004803603602081101561007457600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190505050610149565b6040518082815260200191505060405180910390f35b3480156100c257600080fd5b5061012f600480360360608110156100d957600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190803573ffffffffffffffffffffffffffffffffffffffff16906020019092919080359060200190929190505050610161565b604051808215151515815260200191505060405180910390f35b60006020528060005260406000206000915090505481565b6000816000808673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002054101515156101b057600080fd5b6000808473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002054826000808673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002054011015151561023d57600080fd5b816000808673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060008282540392505081905550816000808573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000206000828254019250508190555060019050939250505056fea165627a7a72305820237bede4b5ceb2d9241cf6932aeb02c1d0c27feba97a95184c62e7d3462ef27c0029
    const shopBytecode = 0x608060405234801561001057600080fd5b50604051602080610baf8339810180604052602081101561003057600080fd5b810190808051906020019092919050505080600260006101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555050610b1d806100926000396000f3fe608060405260043610610057576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806353b628661461005c578063b9db15b4146100a1578063dd31ee5714610202575b600080fd5b34801561006857600080fd5b5061009f6004803603604081101561007f57600080fd5b810190808035906020019092919080359060200190929190505050610389565b005b3480156100ad57600080fd5b506100da600480360360208110156100c457600080fd5b8101908080359060200190929190505050610691565b6040518080602001868152602001858152602001806020018473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001838103835288818151815260200191508051906020019080838360005b8381101561015c578082015181840152602081019050610141565b50505050905090810190601f1680156101895780820380516001836020036101000a031916815260200191505b50838103825285818151815260200191508051906020019080838360005b838110156101c25780820151818401526020810190506101a7565b50505050905090810190601f1680156101ef5780820380516001836020036101000a031916815260200191505b5097505050505050505060405180910390f35b34801561020e57600080fd5b50610387600480360360c081101561022557600080fd5b81019080803590602001909291908035906020019064010000000081111561024c57600080fd5b82018360208201111561025e57600080fd5b8035906020019184600183028401116401000000008311171561028057600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f8201169050808301925050505050505091929192908035906020019092919080359060200190929190803590602001906401000000008111156102f757600080fd5b82018360208201111561030957600080fd5b8035906020019184600183028401116401000000008311171561032b57600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f820116905080830192505050505050509192919290803590602001909291905050506108a3565b005b600080600084815260200190815260200160002060030154111515610416576040517f08c379a00000000000000000000000000000000000000000000000000000000081526004018080602001828103825260138152602001807f50726f6475637420697320736f6c64206f75740000000000000000000000000081525060200191505060405180910390fd5b600080600084815260200190815260200160002090506000339050600260009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1663beabacc8828460040160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1685600201546040518463ffffffff167c0100000000000000000000000000000000000000000000000000000000028152600401808473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020018373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020018281526020019350505050602060405180830381600087803b15801561055257600080fd5b505af1158015610566573d6000803e3d6000fd5b505050506040513d602081101561057c57600080fd5b81019080805190602001909291905050505060018260030160008282540392505081905550600160008581526020019081526020016000208190806001815401808255809150509060018203906000526020600020016000909192909190916101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff160217905550507f6e963996302b87595be0bd0f2d400e2f43c1cfce4378ba7e9b34305fe2f2746b848285604051808481526020018373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001828152602001935050505060405180910390a150505050565b6060600080606060006106a2610a06565b60008088815260200190815260200160002060a06040519081016040529081600082018054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156107595780601f1061072e57610100808354040283529160200191610759565b820191906000526020600020905b81548152906001019060200180831161073c57829003601f168201915b50505050508152602001600182018054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156107fb5780601f106107d0576101008083540402835291602001916107fb565b820191906000526020600020905b8154815290600101906020018083116107de57829003601f168201915b5050505050815260200160028201548152602001600382015481526020016004820160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681525050905080600001518160400151826060015183602001518460800151849450819150955095509550955095505091939590929450565b60a0604051908101604052808681526020018381526020018581526020018481526020013373ffffffffffffffffffffffffffffffffffffffff16815250600080888152602001908152602001600020600082015181600001908051906020019061090f929190610a4c565b50602082015181600101908051906020019061092c929190610a4c565b50604082015181600201556060820151816003015560808201518160040160006101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff1602179055509050507f283a98e56d9d477bc9675301834a627bc96a173f644ce87ab9bbeee35e192965863383604051808481526020018373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001828152602001935050505060405180910390a1505050505050565b60a06040519081016040528060608152602001606081526020016000815260200160008152602001600073ffffffffffffffffffffffffffffffffffffffff1681525090565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f10610a8d57805160ff1916838001178555610abb565b82800160010185558215610abb579182015b82811115610aba578251825591602001919060010190610a9f565b5b509050610ac89190610acc565b5090565b610aee91905b80821115610aea576000816000905550600101610ad2565b5090565b9056fea165627a7a72305820698f03b8ca8d1f1572f4c44e12148998d2ae07d6c17af72dd937e82a1c7c3f070029
    let tokenContractAddress;
    let shopContractAddress;

    const token = new web3.eth.Contract(tokenAbi);
    token.deploy({
	data: tokenBytecode,
    arguments: [1000000000]
    }).send({
    from: '0x....',
    gas: '5000000'
       }).then((contractInstance) => {
	tokenContractAddress = contractInstance.options.address;
     });
    const shop = new web3.eth.Contract(shopAbi);
    shop.deploy({
    data: shopBytecode,
    arguments: [tokenContractAddress],
    }).send({
    from: '0x....',
    gas: '5000000'
    }).then((contractInstance) => {
	shopContractAddress = contractInstance.options.address;
    });
    
3.สำหรับการ Deploy Smart Contract ที่เราเขียนขึ้น สามารถเขียน Migration ได้ดังนี้ค่ะ

    const Token = artifacts.require('Token');
    const Shop = artifacts.require('Shop');
  
      module.exports = function(deployer) {
    deployer
    .deploy(Token, 1000000)
    .then(async () => {
      const tokenContract = await Token.deployed();
      return deployer.deploy(Shop, tokenContract.address);
    })
    };
    
4. Smart Contract ทั้ง Token และ Shop มี Constructor ดังที่แสดงไว้ข้างล่าง ดังนั้นเราจึงต้องใส่ Argument ลงในคำสั่ง deployer.deploy(...) หลังจากชื่อ Contract ด้วย

        pragma solidity >=0.4.25 <0.6.0;
         contract Token {
	      ...
        constructor(uint256 initialSupply) public {
        balanceOf[msg.sender] = initialSupply;
         }
		    ...
         }
        pragma solidity >=0.4.25 <0.6.0;
       contract Shop {
       ...
        constructor (address _tokenAddress) public {
        token = Token(_tokenAddress);
        }
         ...
         }

5.Config Ethereum Networks
หลังจากเชื่อมต่อกับ Ethereum เรียบร้อยแล้ว เราต้องบอกให้ Truffle รู้ว่าต้องเชื่อมต่อกับ Network ใดบ้าง โดยการแก้ไขที่ไฟล์ truffle-config.js 
ในหัวข้อ networks เราสามารถกำหนดตามตัวอย่างที่ Truffle ยกมาในไฟล์ได้เลย แต่ในที่นี้ให้สร้าง Network ชื่อ development ตามนี้ค่ะ

     networks: {
    development: {
     host: "127.0.0.1",     // Localhost (default: none)
     port: 8545,            // Standard Ethereum port (default: none)
     network_id: "*",       // Any network (default: none)
    },
     },
    development จะเป็น Network ตัวแรกที่ Truffle มองหาอัตโนมัติเมื่อเราทำคำสั่ง truffle migrate เพราะฉะนั้นเมื่อเราทำคำสั่ง truffle migrate Truffle จะพยายามเชื่อมต่อ Network development เสมอ

ตัวอย่างการบันทึก Address ใน build/contracts/Shop.json
Truffle จะบันทึกที่ท้าย ๆ ไฟล์ (หัวข้อ networks) ในตัวอย่างนี้ 5777 คือ Network ID ของ Ganache Blockchain ที่เราได้สร้างในหัวข้อที่แล้วค่ะ 
(Network ID คือค่าระบุตัวตนของ Blockchain นั่นเอง) ภายในนี้ จะเก็บ Address ของ Contract ที่เราต้องนำไปใช้งานใน DApp 
รวมทั้ง Hash ของ Transaction (transactionHash) ที่แสดงว่าเราได้ปล่อย Smart Contract เข้ามาใน Blockchain แล้วด้วย

    "networks": {
    "5777": {
      "events": {},
      "links": {},
      "address": "0x1444ac14a1E725E1B30C4eA2b20be3F29100BB98",
      "transactionHash": "0xb3f3866eb2847e10ce3ceadb6bd3a334a76d738023f74ae2ef02a637912f520a"
    }
    }

6.ในโปรเจคของเรา หลังจาก Deploy Smart Contract ทั้งสองเรียบร้อยแล้ว ผู้เขียนต้องการให้มีการส่งเงินจาก Account แรกไปยัง Account ที่ 2 
เนื่องจากตอนที่ Deploy Token เงินทั้งหมดที่เรากำหนดจะไปกองรวมกันที่ Account แรกทั้งหมด (เราเรียก Account ที่เก็บเงินทั้งหมดว่า coinbase) 
ดังนั้นเราต้องออกเหรียญ (Issue Token) ให้ Account อื่นจับจ่ายซื้อของในระบบเรา

    const Token = artifacts.require('Token');
    const Shop = artifacts.require('Shop');
      module.exports = function(deployer, network, accounts) {
     deployer
    .deploy(Token, 1000000)
    .then(async () => {
      const tokenContract = await Token.deployed();
      return deployer.deploy(Shop, tokenContract.address);
    })
    .then(async () => {
      const token = await Token.deployed();
      const coinbase = accounts[0];
      const value = 50000;
      await token.transfer(coinbase, accounts[1], value);
    });
    };
    
7.onnect to Ethereum with Web3
มาถึงส่วนที่สำคัญของแอปพลิเคชันของเรากันแล้ว นั่นก็คือ การเชื่อมต่อไปที่ Ethereum นั่นเอง ซึ่งการติดต่อจากแอปพลิเคชันของเรานั้น เราสามารถใช้ Package ชื่อว่า web3 ค่ะ
ให้สร้างไฟล์ชื่อว่า blockchain.js และเรียกใช้ Package web3 แบบดังตัวอย่างนี้

    const Web3 = require('web3');
    const web3Provider = new Web3.providers.HttpProvider('http://localhost:7545');
    const web3 = new Web3(web3Provider);

การติดต่อ Blockchain จากแอปพลิเคชันของเรา เราต้องกำหนด Address ที่ติดต่อได้ในตัวแปร web3Provider
http(s)://domain:port ให้ถูกต้องค่ะ อย่างในที่นี้ เราจะติดต่อ Blockchain ในเครื่องเราเอง เราจะกำหนด domain
เป็น http://localhost หรือ http://127.0.0.1 ส่วน Port หรือเลขหลัง : (colon) ขึ้นอยู่กับ Client ที่เราใช้ว่าจะเปิดที่เลขใด
หากคุณผู้อ่านใช้ Ganache โปรแกรมจะเปิด Port 7545 สำหรับ Client อื่น ๆ เช่น Geth จะเปิด Port ที่ 8545


ใน Ethereum การเรียกใช้ฟังก์ชันที่ประกาศใน Smart Contract จะมีวิธีการเรียกใช้อยู่ 2 แบบ คือ

แบบ Call – ใช้กับฟังก์ชันที่ทำหน้าที่ดึงข้อมูลอย่างเดียว (ใช้ keyword constant หรือ view ในการประกาศฟังก์ชัน) เมื่อเรียกแล้วจะไม่เกิด Transaction ขึ้น
แบบ Send Transaction – ใช้กับฟังก์ชันที่ทำหน้าที่แก้ไขข้อมูลในระบบ เมื่อเรียกแล้วจะเกิด Transaction ขึ้น (เพื่อเป็นหลักฐานว่ามีการเปลี่ยนแปลงข้อมูลใน Smart Contract นี้ด้วย)
เมื่อต้องการจะเรียกใช้ฟังก์ชัน addProduct หรือฟังก์ชันเพิ่มสินค้าในแอปพลิเคชันของเราจะต้องเรียกฟังก์ชันแบบ Send Transaction ค่ะ

    const addProduct = async (name, price, quantity, imgPath, seller) => {
    const timestamp = Date.now();
    const pid = timestamp;
    const receipt = await shop.addProduct(pid, name, price, quantity, imgPath, timestamp, { from: seller, gas: 1000000 });
     return { receipt: receipt, pid: pid };
    };
จาก Code ด้านบนจะเห็นได้ว่า การเรียกแบบ Send Transaction ของ @truffle/contract จะทำได้โดย
พิมพ์ชื่อฟังก์ชันลงไป
หลังจากนั้นก็นำ Parameter ใส่ลงไปตามที่กำหนดใน Smart Contract
หลังจาก Parameter ตัวสุดท้าย จะต้องใส่ข้อมูลของ Transaction ที่จะส่งไปยัง Contract นี้ด้วย เช่น 
ส่งจาก Address ใด (from), จะใช้ Gas เท่าไรในการประมวลผล Transaction หรือฟังก์ชันนี้ (gas) เป็นต้น
ฟังก์ชัน buyProduct ก็ต้องใช้วิธีการเรียกแบบนี้เช่นกัน เพราะมีการเปลี่ยนแปลงข้อมูลภายใน Smart Contract 
คือ มีการส่ง Token ไปยังผู้ขาย และเปลี่ยนแปลงจำนวนสินค้า

สุดท้ายคือฟังก์ชัน getProduct ฟังก์ชันนี้ทำหน้าที่ดึงข้อมูลของสินค้าออกมา เพราะฉะนั้นการเรียกใช้ฟังก์ชันในแอปพลิเคชันต้องใช้การเรียกแบบ Call ค่ะ
การเรียกแบบ Call ของ @truffle/contract จะทำได้โดยพิมพ์ชื่อฟังก์ชันลงไป ตามด้วย .call() ภายในวงเล็บนั้นจะใส่ Parameter 
ตามที่กำหนดใน Smart Contract ส่วนข้อมูลของ Transaction ไม่จำเป็นต้องใส่ค่ะ ดังโค้ดด้านล่างนี้

    const getProduct = async pid => {
    const product = await shop.getProduct.call(pid);
      product.id = pid;
      return product;
    };


ทำคำสั่งให้แอปพลิเคชันทำงานกันเลย

     npm start
     
จากนั้นเปิด Web Browser แล้วเข้าไปที่ URL http://localhost:3000 

