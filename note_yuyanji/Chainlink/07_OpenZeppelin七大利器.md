OpenZeppelin的智能合约库是以太坊开发者的无价之宝，其中包含了 经过社区审查的ERC代币标准、安全协议以及其他辅助工具，可以帮助 以太坊开发者聚焦于业务功能的实现而无需重新发明轮子。在这篇文章 中，我们将介绍OpenZeppelin中的7个最常用的合约，可以极大的提高 Solidity合约的开发效率并保证合约的安全性。

注意：在本文中我们使用的OpenZeppelin版本为2.5.x，solidity编译器 为0.5.x。

用自己熟悉的语言学习以太坊DApp开发： Java | Php | Python | .Net / C# | Golang | Node.JS | Flutter / Dart

1、最流行的访问控制合约：Ownable
OpenZeppelin的Ownable合约提供的onlyOwner模式是用来保护特定合约 方法的访问权限的基础但非常有效的模式，因此这个模式在以太坊智能合约 开发中非常流行。

Ownable合约的部署账号被视为合约的持有人（owner），某些合约方法，例如 转移所有权，只允许owner账号调用。

下面是Ownable合约的源代码：

pragma solidity ^0.5.0;

import "../GSN/Context.sol";

contract Ownable is Context {
    address private _owner;

    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    constructor () internal {
        address msgSender = _msgSender();
        _owner = msgSender;
        emit OwnershipTransferred(address(0), msgSender);
    }

    function owner() public view returns (address) {
        return _owner;
    }

    modifier onlyOwner() {
        require(isOwner(), "Ownable: caller is not the owner");
        _;
    }

    function isOwner() public view returns (bool) {
        return _msgSender() == _owner;
    }

    function renounceOwnership() public onlyOwner {
        emit OwnershipTransferred(_owner, address(0));
        _owner = address(0);
    }

    function transferOwnership(address newOwner) public onlyOwner {
        _transferOwnership(newOwner);
    }

    function _transferOwnership(address newOwner) internal {
        require(newOwner != address(0), "Ownable: new owner is the zero address");
        emit OwnershipTransferred(_owner, newOwner);
        _owner = newOwner;
    }
}
注意在构造函数中是如何设置合约的owner账号的。当子合约初始化时，发起账号默认 就成为_owner。

下面是一个简单的、继承自Ownable的合约，使用onlyOwner保护restrictedFunction：

pragma solidity ^0.5.5;

import "@openzeppelin/contracts/ownership/Ownable.sol";

contract OwnableContract is Ownable {

  function restrictedFunction() public onlyOwner returns (uint) {
    return 99;
  }

  function openFunction() public returns (uint) {
    return 1;
  }

}
由于我们将onlyOwner修饰符应用到restrictFunction()方法，因此只有 合约的owner账号可以成功调用该方法了。

2、最流行的访问控制库：Roles
在访问控制方面OpenZeppelin提供的另一个解决方案是Roles库，它比 Ownable合约更高级一点，支持多种角色，因此如果你的合约需要设置 多个访问控制保护级别时，应当考虑使用Roles库。

下面是OpenZeppelin的Roles库的源代码：

pragma solidity ^0.5.0;

library Roles {
    struct Role {
        mapping (address => bool) bearer;
    }

    function add(Role storage role, address account) internal {
        require(!has(role, account), "Roles: account already has role");
        role.bearer[account] = true;
    }

    function remove(Role storage role, address account) internal {
        require(has(role, account), "Roles: account does not have role");
        role.bearer[account] = false;
    }

    function has(Role storage role, address account) internal view returns (bool) {
        require(account != address(0), "Roles: account is the zero address");
        return role.bearer[account];
    }
}
由于Roles是一个Solidity库而非合约，因此不能像使用Ownable那样 继承它，我们需要使用solidity的using语句来将库中定义的方法挂接 到指定的数据类型上。

例如，下面的代码使用Roles库设置两种角色：_minters和_burners：

pragma solidity ^0.5.0;

import "@openzeppelin/contracts/access/Roles.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20Detailed.sol";

contract MyToken is ERC20, ERC20Detailed {
    using Roles for Roles.Role;

    Roles.Role private _minters;
    Roles.Role private _burners;

    constructor(address[] memory minters, address[] memory burners)
        ERC20Detailed("MyToken", "MTKN", 18)
        public
    {
        for (uint256 i = 0; i < minters.length; ++i) {
            _minters.add(minters[i]);
        }

        for (uint256 i = 0; i < burners.length; ++i) {
            _burners.add(burners[i]);
        }
    }

    function mint(address to, uint256 amount) public {
        // Only minters can mint
        require(_minters.has(msg.sender), "DOES_NOT_HAVE_MINTER_ROLE");

        _mint(to, amount);
    }

    function burn(address from, uint256 amount) public {
        // Only burners can burn
        require(_burners.has(msg.sender), "DOES_NOT_HAVE_BURNER_ROLE");

       _burn(from, amount);
    }
}
第8行的作用是将Roles库中的方法挂接到Roles.Role类型上。第18行代码 展示了如何使用这些挂接的方法：_minters.add()，其中add()就是Roles库 提供的实现。

3、安全的算术运算库：SafeMath
永远不要直接使用算术运算符例如：+、-、*、/ 进行数学计算，除非你 了解如何检查溢出漏洞，否则没法保证这些计算的安全性 —— 这就是SafeMath库的作用，它会帮你进行所有必要的检查，避免你的代码中 因算术运算而引入漏洞。

下面是SafeMath的源代码：

pragma solidity ^0.5.0;

library SafeMath {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "SafeMath: addition overflow");

        return c;
    }

    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        return sub(a, b, "SafeMath: subtraction overflow");
    }

    function sub(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b <= a, errorMessage);
        uint256 c = a - b;

        return c;
    }

    function mul(uint256 a, uint256 b) internal pure returns (uint256) {
        // Gas optimization: this is cheaper than requiring 'a' not being zero, but the
        // benefit is lost if 'b' is also tested.
        // See: https://github.com/OpenZeppelin/openzeppelin-contracts/pull/522
        if (a == 0) {
            return 0;
        }

        uint256 c = a * b;
        require(c / a == b, "SafeMath: multiplication overflow");

        return c;
    }

    function div(uint256 a, uint256 b) internal pure returns (uint256) {
        return div(a, b, "SafeMath: division by zero");
    }

    function div(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        // Solidity only automatically asserts when dividing by 0
        require(b > 0, errorMessage);
        uint256 c = a / b;
        // assert(a == b * c + a % b); // There is no case in which this doesn't hold

        return c;
    }

    function mod(uint256 a, uint256 b) internal pure returns (uint256) {
        return mod(a, b, "SafeMath: modulo by zero");
    }

    function mod(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b != 0, errorMessage);
        return a % b;
    }
}
和Roles库的用法类似，你需要使用using语句将SafeMath库中的方法挂接到uint256类型上，例如：

using SafeMath for uint256;
4、安全类型转换库：SafeCast
作为一个智能合约开发者，我们常常会思考如何减少合约的执行时间 以及所占的空间，节约代码空间的一个办法就是使用更少位数的整数类型。 但不幸的是，如果你使用uint8作为变量类型，那么在调用SafeMath库 的方法之前，就必须先将其转换为uint256类型，然后在调用SafeMath库 的方法之后，还需要再转换回uint8类型。SafeCast库的作用就在于可以 帮你完成这些转换而无需担心溢出问题。

下面是SafeCast的源代码：

pragma solidity ^0.5.0;

library SafeCast {

    function toUint128(uint256 value) internal pure returns (uint128) {
        require(value < 2**128, "SafeCast: value doesn\'t fit in 128 bits");
        return uint128(value);
    }

    function toUint64(uint256 value) internal pure returns (uint64) {
        require(value < 2**64, "SafeCast: value doesn\'t fit in 64 bits");
        return uint64(value);
    }

    function toUint32(uint256 value) internal pure returns (uint32) {
        require(value < 2**32, "SafeCast: value doesn\'t fit in 32 bits");
        return uint32(value);
    }

    function toUint16(uint256 value) internal pure returns (uint16) {
        require(value < 2**16, "SafeCast: value doesn\'t fit in 16 bits");
        return uint16(value);
    }

    function toUint8(uint256 value) internal pure returns (uint8) {
        require(value < 2**8, "SafeCast: value doesn\'t fit in 8 bits");
        return uint8(value);
    }
}
下面的示例代码展示了如何使用SafeCast将uint转换为uint8：

pragma solidity ^0.5.5;

import "@openzeppelin/contracts/math/SafeCast.sol";

contract BasicSafeCast {

  using SafeCast for uint;
  
  function castToUint8(uint _a) public returns (uint8) {
    return _a.toUint8();
  }
}
5、同质化通证合约：ERC20/ERC20Detailed
没必要自己实现整个ERC20代币合约 —— OpenZeppelin已经帮你完成了， 我们只需要进行适当的扩展。

OpenZeppelin的ERC20实现提供了很多可选项。例如，如果你不需要命名 自己的代码，那么OpenZeppelin的基本ERC20合约就够了。ERC20Detailed 合约包含了额外的功能，例如代币命令、代币符号以及小数点位数等等。

下面的代码展示了如何利用OpenZeppelin的ERC20和ERC20Detailed合约实现 自己的代币：

pragma solidity ^0.5.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20Detailed.sol";

contract GLDToken is ERC20, ERC20Detailed {
    constructor(uint256 initialSupply) ERC20Detailed("Gold", "GLD", 18) public {
        _mint(msg.sender, initialSupply);
    }
}
6、非同质化通证合约：ERC721Enumerable / ERC721Full
同样，OpenZeppelin也提供了非同质化通证的实现。

ERC721Enumerable合约提供了_tokensOfOwner()方法，因此支持 枚举特定账号持有的ERC721资产。如果你希望大而全的合约，那么 可以直接选择ERC721Full。下面的代码展示了基于ERC721Full的 一个非同质化通证实现：

pragma solidity ^0.5.0;

import "@openzeppelin/contracts/token/ERC721/ERC721Full.sol";
import "@openzeppelin/contracts/drafts/Counters.sol";

contract GameItem is ERC721Full {
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIds;

    constructor() ERC721Full("GameItem", "ITM") public {
    }

    function awardItem(address player, string memory tokenURI) public returns (uint256) {
        _tokenIds.increment();

        uint256 newItemId = _tokenIds.current();
        _mint(player, newItemId);
        _setTokenURI(newItemId, tokenURI);

        return newItemId;
    }
}
7、辅助工具库：Address
有时候在Solidity合约中需要了解一个地址是普通钱包地址还是合约地址。 OpenZeppelin的Address库提供了一个方法isContract()可以帮你 解决这个问题。

下面的代码展示了如何使用这个函数：

pragma solidity ^0.5.5;

import "@openzeppelin/contracts/utils/Address.sol";

contract BasicUtils {
    using Address for address;

    function checkIfContract(address _addr) public {
        return _addr.isContract();
    }
}
