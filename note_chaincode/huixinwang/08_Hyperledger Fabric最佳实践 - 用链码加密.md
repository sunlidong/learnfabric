Hyperledger Fabric是最流行的区块链开发框架之一，它有独特的定位和一些鲜明 的特点，例如许可制架构、可插拔组件、支持私密交易的通道、模块化以及可扩展性， 因此适合企业联盟链应用的开发。在这篇文章中，我们将介绍如何使用链码加密/解密 保存在Hyperledger Fabric区块链上的敏感数据。

Hyperledger Fabric区块链开发教程： Node.js | Java | Golang

1、Hyperledger Fabric链码加密/解密的应用背景
在企业环境中，有时我们需要处理一些敏感的数据，例如保存信用卡 数据、银行信息、生物识别数据、健康信息等等，这些敏感数据是我们 基于分布式账本的业务应用的一部分，最终用户通常希望即使在数据库 被渗透的情况下也能保证这些私密信息的安全性。

在一些传统的应用中，我们会将数据库中的数据加密，这样即使有人 偷偷进入数据库，也无法理解数据的真实含义。同样，加密区块链数据库 中的用户数据也是保护隐私的有效手段。现在让我们看看如何使用NodeJS 链码来加密要写入区块链数据库的数据。

在我们开始后续的实现之前，需要先指出一点：由于引入了额外的加密 和解密环节，这会导致生产环境中的应用处理速度变慢，要进行加密/解密 处理的数据量越大，对应用性能的影响就越大。

2、Hyperledger Fabric链码加密/解密的处理流程
在我们开始加密处理之前，先解释一下要用到的链码的逻辑。示例链码 就是一个简单的用户注册和登录实现。用户需要先提供用户名和密码进行 注册，这些身份信息将以明文形式保存在数据库中，当用户登录时，将使用 保存在数据库中的身份信息进行身份验证。因此我们将为这些身份信息添加 加密/解密层。

在这个示例中，我将尽力保持代码的简单，使用nodejs内置的crypto库。 根据应用的不同，你也可以使用定制的加密实现。

加密方法实现如下：

function encrypt(data,password){   
  const cipher = crypto.createCipher('aes256', password);  
  let encrypted = cipher.update(data, 'utf8', 'hex');
  encrypted += cipher.final('hex');   
  return encrypted;
}
encrypt()方法使用aes256算法加密指定的数据，它有两个参数：

data：要加密的数据
password：加密口令
解密方法实现如下：

function decrypt(cipherData,password)  {    
   const decipher = crypto.createDecipher('aes256', password);    
   let decrypted = decipher.update(cipherData, 'hex', 'utf8');
   decrypted += decipher.final('utf8');   
   return decrypted.toString();}
decrypt()方法使用aes256算法和传入的加密口令，解密传入 的加密数据。

3、Hyperledger Fabric链码加密/解密的实现代码
我们之前介绍了用户注册/登录链码的逻辑。当用户注册时提交其用户名和登录密码， 因此我们需要在将用户的身份数据存入Hyperledger Fabric区块链之前先进行加密操作。

async signUp(stub, args) {
      if (args.length != 3) {
     return Buffer.from('Incorrect number of arguments. Expecting 3');
            }else{
   console.info('**Storing Credentials on Blockchain**');

   const credentials  = {userName:args[0],password:args[1]};
   let data = JSON.stringify(credentials);
   let cipher = encrypt(data,args[2]);
   
   await stub.putState(args[0], Buffer.from(JSON.stringify(cipher)));
   console.info('*Signup Successfull..Your Username is '+args[0]);
   return Buffer.from('Signup Successfull..Your Username is '+args[0]);
    }
}
同样，当登录时，链码需要验证用户名是否在Hyperledger Fabric的链上数据库中存在并检查 密码是否正确。因此在检查身份信息之前，需要首先解密Hyperledger Fabric的链上数据：

async login(stub, args) {
  if (args.length != 3) {
     return Buffer.from('Incorrect number of arguments. Expecting 3');
        }
    
  let userName=args[0];
  let password=args[1];
  let credentialsAsBytes = await stub.getState(args[0]); 
    
  if (!credentialsAsBytes || credentialsAsBytes.toString().length <= 0) {
    return Buffer.from('Incorrect Username..!');
         }
  else{
  let data= JSON.parse(credentialsAsBytes);
  let decryptData= decrypt(data,args[2]);
  let credentials= JSON.parse(decryptData);
  if (password!=credentials.password) {
  return Buffer.from('Incorrect Password..!');
        }

  //Functions go here after signin
  console.log('Login Successfull..✓');
  return Buffer.from('Login Successfull..');
      }
  }

}
最后我们实现用户注册/登录链码的路由分发：

async Invoke(stub) {
    let ret = stub.getFunctionAndParameters();
    console.info(ret);

    let method = this[ret.fcn];
    if (!method) {
      console.error('no function of name:' + ret.fcn + ' found');
      throw new Error('Received unknown function ' + ret.fcn + ' invocation');
            }
    try {
      let payload = await method(stub, ret.params);
      return shim.success(payload);
         } 
    catch (err) {
      console.log(err);
      return shim.error(err);
            }
     }