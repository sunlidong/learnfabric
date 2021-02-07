OpenZeppelin提供了智能合约的三种访问控制模式：Ownable合约、 Roles库和3.0新增的AccessControl合约。在这个教程中，我们将 学习这三种访问控制模式的差异，以及如何在自己的以太坊智能合约 中利用这些访问控制模式增强Solidity合约的安全性。

用你熟悉的开发语言学习以太坊DApp开发： Java | Php | Python | .Net / C# | Golang | Node.JS | Flutter / Dart

控制对智能合约特定方法的访问权限，对于智能合约的安全性非常重要。 熟悉OpenZeppelin的智能合约库的开发者都知道这个库已经提供了根据 访问等级进行访问限制的选项，其中最常见的就是Ownable合约管理的 onlyOwner模式，另一个是OpenZeppelin的Roles库，它允许合约 在部署前定义多种角色并为每个函数设置规则，以确保msg.sender具有 正确的角色。在OpenZeppelin 3.0中又引入了更强大的AccessControl合约， 其定位是一站式访问控制解决方案。

1、Ownable合约 —— 最简单最流行的访问控制模式
onlyOwner模式是最常见也最容易实现的访问控制方法，它虽然基础但 但非常有效。

该模式假设智能合约存在单一管理员，支持管理员将全新转移给另一个账号。

通过扩展Ownable合约，子合约就可以在定义方法时使用onlyOwner修饰符， 这些被修饰的方法就要求交易发起账号必须是合约的管理员。

下面这个简单的示例展示了如何使用Ownable合约提供的onlyOwner修饰符 来限制对restrictedFunction的访问：

function normalFunction() public {
    // 任何人都可以调用
}

function restrictedFunction() public onlyOwner {
    // 只有合约管理员可以调用
}
2、Roles库 —— OpenZeppelin自己喜欢的访问控制模式
虽然Ownable合约简单易用，但是OpenZeppelin库中的其他合约都是 使用Roles库来实现访问控制。这是因为Roles库比Ownable合约提供 了更多的灵活性。

我们使用using语句引入Roles合约库，来为数据类型增加功能。 Roles库为Role数据类型实现了三个方法。

下面是Role的定义：

pragma solidity ^0.5.0;

/**
 * @title Roles
 * @dev Library for managing addresses assigned to a Role.
 */
library Roles {
    struct Role {
        mapping (address => bool) bearer;
    }

    /**
     * @dev Give an account access to this role.
     */
    function add(Role storage role, address account) internal {
        require(!has(role, account), "Roles: account already has role");
        role.bearer[account] = true;
    }

    /**
     * @dev Remove an account's access to this role.
     */
    function remove(Role storage role, address account) internal {
        require(has(role, account), "Roles: account does not have role");
        role.bearer[account] = false;
    }

    /**
     * @dev Check if an account has this role.
     * @return bool
     */
    function has(Role storage role, address account) internal view returns (bool) {
        require(account != address(0), "Roles: account is the zero address");
        return role.bearer[account];
    }
}
在代码的开头部分，我们剋有看到Role结构，合约使用该结构定义多个角色 以及其成员。方法add()、remove()、has()则提供了与Role结构交互 的接口。

例如，下面的代码展示了如何使用两个不同的角色 —— _minters和_burners —— 来为特定的方法设定访问控制规则：

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
注意在mint()函数中，require语句确保交易发起方具有minter角色， 即_minters.has(msg.sender)。

3、AccessControl合约 —— 官方的一站式访问控制解决方案
Roles库虽然灵活，但也存在一定的局限性。因为它是一个Solidity库， 所以它的数据存储是被引入的合约控制的，而理想的实现应当是让引入 Roles库的合约只需要关心每个方法能实现的访问控制功能。

OpenZeppelin 3.0新增的AccessControl合约被官方称为：

可以满足所有身份验证需求的一站式解决方案，它允许你：1、轻松定义具有 不同权限的多种角色 2、定义哪个账号可以进行角色的授权与回收 3、 枚举系统中所有的特权账号。

上述的3个特性中，后两点都是Roles库不支持的。看起来OpenZeppelin 正在逐渐实现基于角色的访问控制和基于属性的访问控制这些在传统的 计算安全中非常重要的标准。

4、AccessControl合约代码剖析
下面是OpenZeppelin的AccessControl合约的代码：

pragma solidity ^0.6.0;

import "../utils/EnumerableSet.sol";
import "../utils/Address.sol";
import "../GSN/Context.sol";

/**
 * @dev Contract module that allows children to implement role-based access
 * control mechanisms.
 *
 * Roles are referred to by their `bytes32` identifier. These should be exposed
 * in the external API and be unique. The best way to achieve this is by
 * using `public constant` hash digests:
 *
 * 
 * bytes32 public constant MY_ROLE = keccak256("MY_ROLE");
 * 
 *
 * Roles can be used to represent a set of permissions. To restrict access to a
 * function call, use {hasRole}:
 *
 * 
 * function foo() public {
 *     require(hasRole(MY_ROLE, _msgSender()));
 *     ...
 * }
 * 
 *
 * Roles can be granted and revoked dynamically via the {grantRole} and
 * {revokeRole} functions. Each role has an associated admin role, and only
 * accounts that have a role's admin role can call {grantRole} and {revokeRole}.
 *
 * By default, the admin role for all roles is `DEFAULT_ADMIN_ROLE`, which means
 * that only accounts with this role will be able to grant or revoke other
 * roles. More complex role relationships can be created by using
 * {_setRoleAdmin}.
 */
abstract contract AccessControl is Context {
    using EnumerableSet for EnumerableSet.AddressSet;
    using Address for address;

    struct RoleData {
        EnumerableSet.AddressSet members;
        bytes32 adminRole;
    }

    mapping (bytes32 => RoleData) private _roles;

    bytes32 public constant DEFAULT_ADMIN_ROLE = 0x00;

    /**
     * @dev Emitted when `account` is granted `role`.
     *
     * `sender` is the account that originated the contract call, an admin role
     * bearer except when using {_setupRole}.
     */
    event RoleGranted(bytes32 indexed role, address indexed account, address indexed sender);

    /**
     * @dev Emitted when `account` is revoked `role`.
     *
     * `sender` is the account that originated the contract call:
     *   - if using `revokeRole`, it is the admin role bearer
     *   - if using `renounceRole`, it is the role bearer (i.e. `account`)
     */
    event RoleRevoked(bytes32 indexed role, address indexed account, address indexed sender);

    /**
     * @dev Returns `true` if `account` has been granted `role`.
     */
    function hasRole(bytes32 role, address account) public view returns (bool) {
        return _roles[role].members.contains(account);
    }

    /**
     * @dev Returns the number of accounts that have `role`. Can be used
     * together with {getRoleMember} to enumerate all bearers of a role.
     */
    function getRoleMemberCount(bytes32 role) public view returns (uint256) {
        return _roles[role].members.length();
    }

    /**
     * @dev Returns one of the accounts that have `role`. `index` must be a
     * value between 0 and {getRoleMemberCount}, non-inclusive.
     *
     * Role bearers are not sorted in any particular way, and their ordering may
     * change at any point.
     *
     * WARNING: When using {getRoleMember} and {getRoleMemberCount}, make sure
     * you perform all queries on the same block. See the following
     * https://forum.openzeppelin.com/t/iterating-over-elements-on-enumerableset-in-openzeppelin-contracts/2296[forum post]
     * for more information.
     */
    function getRoleMember(bytes32 role, uint256 index) public view returns (address) {
        return _roles[role].members.at(index);
    }

    /**
     * @dev Returns the admin role that controls `role`. See {grantRole} and
     * {revokeRole}.
     *
     * To change a role's admin, use {_setRoleAdmin}.
     */
    function getRoleAdmin(bytes32 role) public view returns (bytes32) {
        return _roles[role].adminRole;
    }

    /**
     * @dev Grants `role` to `account`.
     *
     * If `account` had not been already granted `role`, emits a {RoleGranted}
     * event.
     *
     * Requirements:
     *
     * - the caller must have ``role``'s admin role.
     */
    function grantRole(bytes32 role, address account) public virtual {
        require(hasRole(_roles[role].adminRole, _msgSender()), "AccessControl: sender must be an admin to grant");

        _grantRole(role, account);
    }

    /**
     * @dev Revokes `role` from `account`.
     *
     * If `account` had been granted `role`, emits a {RoleRevoked} event.
     *
     * Requirements:
     *
     * - the caller must have ``role``'s admin role.
     */
    function revokeRole(bytes32 role, address account) public virtual {
        require(hasRole(_roles[role].adminRole, _msgSender()), "AccessControl: sender must be an admin to revoke");

        _revokeRole(role, account);
    }

    /**
     * @dev Revokes `role` from the calling account.
     *
     * Roles are often managed via {grantRole} and {revokeRole}: this function's
     * purpose is to provide a mechanism for accounts to lose their privileges
     * if they are compromised (such as when a trusted device is misplaced).
     *
     * If the calling account had been granted `role`, emits a {RoleRevoked}
     * event.
     *
     * Requirements:
     *
     * - the caller must be `account`.
     */
    function renounceRole(bytes32 role, address account) public virtual {
        require(account == _msgSender(), "AccessControl: can only renounce roles for self");

        _revokeRole(role, account);
    }

    /**
     * @dev Grants `role` to `account`.
     *
     * If `account` had not been already granted `role`, emits a {RoleGranted}
     * event. Note that unlike {grantRole}, this function doesn't perform any
     * checks on the calling account.
     *
     * [WARNING]
     * ====
     * This function should only be called from the constructor when setting
     * up the initial roles for the system.
     *
     * Using this function in any other way is effectively circumventing the admin
     * system imposed by {AccessControl}.
     * ====
     */
    function _setupRole(bytes32 role, address account) internal virtual {
        _grantRole(role, account);
    }

    /**
     * @dev Sets `adminRole` as ``role``'s admin role.
     */
    function _setRoleAdmin(bytes32 role, bytes32 adminRole) internal virtual {
        _roles[role].adminRole = adminRole;
    }

    function _grantRole(bytes32 role, address account) private {
        if (_roles[role].members.add(account)) {
            emit RoleGranted(role, account, _msgSender());
        }
    }

    function _revokeRole(bytes32 role, address account) private {
        if (_roles[role].members.remove(account)) {
            emit RoleRevoked(role, account, _msgSender());
        }
    }
}
在42行定义的RoleData结构使用3.0版本中新增的EnumerableSet（枚举集合） 作为保存角色成员的数据结构，这使得枚举特权用户成为可能。

RoleData结构将adminRole定义为bytes32变量，它表示哪个角色可以作为 一个特定角色的管理员，也就是说负责这个角色成员的授权与回收。

在代码的57行和66行，分别触发角色授权和回收事件。

Roles库提供提供了三个函数： has(), add()和 remove()。AccessControl 合约中也包含这些方法，并且提供额外的功能，例如获取角色数量、获取 指定角色的特定成员等等。

5、使用AccessControl合约实现代币合约的访问控制
前面的代币合约示例使用Roles库定义了两种不同的角色_minters和_burners。 在这里我们使用AccessControl实现同样的功能。

pragma solidity ^0.6.0;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

    constructor() public ERC20("MyToken", "TKN") {
        // Grant the contract deployer the default admin role: it will be able
        // to grant and revoke any roles
        _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    }

    function mint(address to, uint256 amount) public {
        require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
        _mint(to, amount);
    }

    function burn(address from, uint256 amount) public {
        require(hasRole(BURNER_ROLE, msg.sender), "Caller is not a burner");
        _burn(from, amount);
    }
}
和用Roles库的实现有什么区别？

首先，不再需要在子合约中定义每一个角色，因为这些角色都保存在父合约中。 只有bytes32类型的ID作为状态常量存在子合约中，例如这个示例中的MINTER_ROLE 和BURNER_ROLE。

_setupRole()用于在构造函数中设置初始的角色管理员，以便跳过AccessControl 中的grantRole()进行的检查（因为在创建合约时还没有管理员）。

另外，不再需要像调用库方法那样需要借助于特定的数据类型，例如_minters.has(msg.sender)， 现在角色操作的相关方法可以在子合约中直接调用，例如hasRole(MINTER_ROLE,msg.sender)， 这可以让子合约的代码看起来更加干净，可读性更高。

5、教程小结
在这个教程中，我们学习了OpenZeppelin合约库中的三种访问控制设计模式 以及其使用方法：Ownable合约、Roles库以及3.0新增的AccessControl合约， Ownable最简单，AccessControl最强大，你可以根据自己的需求选择合适的 访问控制手段来保护自己的Solidity智能合约。

