在以太坊区块链上，Gas被用来补偿矿工为智能合约的存储与执行所提供的算力。 目前以太坊的利用在逐渐增长，而交易手续费成本也水涨传告 —— 现在每天的gas成本 已经高达数百万美元。随着以太坊生态系统的扩大，Solidity智能合约开发者也 需要关注gas利用的优化问题了。本文将介绍在使用Solidity开发以太坊智能 合约时常用的一些Gas优化模式。

以太坊教程链接： Dapp入门 | 电商Dapp实战 | Token实战 | Php对接 | Java对接 | Python对接 | C#对接 | Dart对接

1、使用短路模式排序Solidity操作
短路（short-circuiting）是一种使用或/与逻辑来排序不同成本操作的solidity合约 开发模式，它将低gas成本的操作放在前面，高gas成本的操作放在后面，这样如果前面的 低成本操作可行，就可以跳过（短路）后面的高成本以太坊虚拟机操作了。

// f(x) 是低gas成本的操作
// g(y) 是高gas成本的操作

// 按如下排序不同gas成本的操作
f(x) || g(y)
f(x) && g(y)
2、删减不必要的Solidity库
在开发Solidity智能合约时，我们引入的库通常只需要用到其中的部分功能，这意味着 其中可能会包含大量对于你的智能合约而言其实是冗余的solidity代码。如果可以在你自己的 合约里安全有效地实现所依赖的库功能，那么就能够达到优化solidity合约的gas利用的目的。

例如，在下面的solidity代码中，我们的以太坊合约只是用到了SafeMath库的add方法：

import './SafeMath.sol' as SafeMath;

contract SafeAddition {
  function safeAdd(uint a, uint b) public pure returns(uint) {
    return SafeMath.add(a, b);
  }
}
通过参考SafeMath的这部分代码的实现，可以把对这个solidity库的依赖剔除掉：

contract SafeAddition {
  function safeAdd(uint a, uint b) public pure returns(uint) {
    uint c = a + b;
    require(c >= a, "Addition overflow");
    return c;
  }
}
3、显式声明Solidity合约函数的可见性
在Solidity合约开发种，显式声明函数的可见性不仅可以提高智能合约的安全性， 同时也有利于优化合约执行的gas成本。例如，通过显式地标记函数为外部函数（External）， 可以强制将函数参数的存储位置设置为calldata，这会节约每次函数执行 时所需的以太坊gas成本。

4、使用正确的Solidity数据类型
在Solidity中，有些数据类型要比另外一些数据类型的gas成本高。有必要 了解可用数据类型的gas利用情况，以便根据你的需求选择效率最高的那种。 下面是关于solidity数据类型gas消耗情况的一些规则：

在任何可以使用uint类型的情况下，不要使用string类型
存储uint256要比存储uint8的gas成本低，为什么？点击这里查看原文
当可以使用bytes类型时，不要在solidity合约种使用byte[]类型
如果bytes的长度有可以预计的上限，那么尽可能改用bytes1~bytes32这些具有固定长度的solidity类型
bytes32所需的gas成本要低于string类型
5、避免Solidity智能合约中的死代码
死代码（Dead code）是指那些永远也不会执行的Solidity代码，例如那些执行条件永远 也不可能满足的代码，就像下面的两个自相矛盾的条件判断里的Solidity代码块，消耗了 以太坊gas资源但没有任何作用：

function deadCode(uint x) public pure {
  if(x < 1) {
    if(x > 2) {
      return x;
    }
  }
}
6、避免使用不必要的条件判断
有些条件断言的结果不需要Solidity代码的执行就可以了解，那么这样的条件判断就可以 精简掉。例如下面的Solidity合约代码中的两级判断条件，最外层的判断是在浪费宝贵的 以太坊gas资源：

function opaquePredicate(uint x) public pure {
  if(x < 1) {
    if(x < 0) {
      return x;
    }
  }
}
7、避免在循环中执行gas成本高的操作
由于SLOAD和SSTORE操作码的成本高昂，因此管理storage变量的gas成本 要远远高于内存变量，所以要避免在循环中操作storage变量。例如下面的solidity 代码中，num变量是一个storage变量，那么未知循环次数的 若干次操作，很可能会造成solidity开发者意料之外的以太坊gas消耗黑洞：

uint num = 0;

function expensiveLoop(uint x) public {
  for(uint i = 0; i < x; i++) {
    num += 1;
  }
}
解决上述反模式以太坊合约代码的方法，是创建一个solidity临时变量 来代替上述全局变量参与循环，然后在循环结束后重新将临时变量的值赋给全局变量：

uint num = 0;

function lessExpensiveLoop(uint x) public {
  uint temp = num;
  for(uint i = 0; i < x; i++) {
    temp += 1;
  }
  num = temp;
}
8、避免为可预测的结果使用Solidity循环
如果一个循环计算的结果是无需编译执行Solidity代码就可以预测的，那么 就不要使用循环，这可以可观地节省gas。例如下面的以太坊合约代码就可以 直接设置num变量的值：

function constantOutcome() public pure returns(uint) {
  uint num = 0;
  for(uint i = 0; i < 100; i++) {
    num += 1;
  }
  return num;
}
9、循环合并模式
有时候在Solidity智能合约中，你会发现两个循环的判断条件一致，那么在这种情况下 就没有理由不合并它们。例如下面的以太坊合约代码：

function loopFusion(uint x, uint y) public pure returns(uint) {
  for(uint i = 0; i < 100; i++) {
    x += 1;
  }
  for(uint i = 0; i < 100; i++) {
    y += 1;
  }
  return x + y;
}
10、避免循环中的重复计算
如果循环中的某个Solidity表达式在每次迭代都产生同样的结果，那么就可以将其 移出循环先行计算，从而节省掉循环中额外的gas成本。如果表达式中使用的变量是storage变量， 这就更重要了。例如下面的智能合约代码中表达式a*b的值，并不需要每次迭代重新计算：

uint a = 4;
uint b = 5;
function repeatedComputations(uint x) public returns(uint) {
  uint sum = 0;
  for(uint i = 0; i <= x; i++) {
    sum = sum + a * b;
  }
}
