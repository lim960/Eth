因为先写的博客 直接复制过来了
博客地址https://blog.csdn.net/weixin_42704356/article/details/105377649


#  必读
 **本文将从0开始完成一些普遍的以太坊功能开发 本文涉及内容如下：**
 	1.geth节点搭建与基本使用
 	2.创建账户
 	3.查询余额&查询代币余额
 	4.以太币交易
 	5.合约使用
 	6.代币交易(本文以ERC20 USDT为例)
 	7.代币的代理交易(代币归集核心)
 	8.交易监听
 	

## 1.前言
项目需要，没接触过以太坊的我被分配了开发以太坊的任务，网上的资料也是少得可怜，以至于走了很多弯路，终于摸爬滚打了十天时间，终于有些成果，决定写个博客，首先可以帮助一些和我当初同样处境的朋友(目前网上的资料是真的非常碎片化，像代理交易部分根本找不到资料)，其次避免日后再次使用时有遗忘。

## 2.基础（必看）
1.[以太坊官方文档(中文版)](https://pan.baidu.com/s/1ktODJKLMBmkOsi8MPrpIJA)
首先这个文档必看，认真读两遍，通过文档需要清楚 节点 去中心化 gas ether 账户 智能合约  都是什么
wei kwei gwei ...ether 单位之间的区别及转换 基础的资料网上很多 不多废话 不理解的自行百度
2.[web3j(简单了解一下)](https://docs.web3j.io/)
web3j（org.web3j）是Java版本的以太坊JSON RPC 接口协议封装实现，如果需要将你的Java应用或安卓应用接入以太坊，或者希望用java开发一个 钱包应用，那么用web3j就对了。
简单来说就是java开发以太坊的类库
3.[区块链浏览器](https://etherscan.io/)
目前被墙了，首先里边是有一些查询类的api的 香港服务器可以直接调用 不过本文内没有使用
但是调用智能合约部分需要在此网站内看合约代码 abi bin等 也可以查看交易记录 有助于加强对区块链的理解 还是很重要的
据说：可以利用这个api以离线签名然后广播的方式开发以太坊，这样可以不需要搭建节点，因为听说的时候已经完成了开发任务，就没再研究过，有兴趣的可以自己找一下
4.说明一下
以太坊交易所之类的都是第三方的，不是官方提供的，有一定局限性(要求账号在它的平台之类)，如不介意可以直接接入，会方便很多，我当初在这浪费了不少时间，后来发现方向不对
## 3.web3j引入

```javascript
		<dependency>
            <groupId>org.web3j</groupId>
            <artifactId>core</artifactId>
            <version>3.2.0</version>
        </dependency>
        <!-- 下面两个创建账户使用的 另外还有五个jar包 因为用阿里代理仓库引不进来另外添加 -->
        <dependency>
            <groupId>com.lambdaworks</groupId>
            <artifactId>scrypt</artifactId>
            <version>1.4.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.madgag.spongycastle/core -->
        <dependency>
            <groupId>com.madgag.spongycastle</groupId>
            <artifactId>core</artifactId>
            <version>1.58.0.0</version>
        </dependency>
```

[BIP32-0.0.9jar包下载连接](https://mvnrepository.com/artifact/io.github.novacrypto/BIP32/0.0.9)
[BIP39-0.1.9jar包下载连接](https://mvnrepository.com/artifact/io.github.novacrypto/BIP39/0.1.9)
[BIP44-0.0.3jar包下载连接](https://mvnrepository.com/artifact/io.github.novacrypto/BIP44/0.0.3)
[SHA256-0.0.1jar包下载连接](https://mvnrepository.com/artifact/io.github.novacrypto/SHA256/0.0.1)
[ToRuntime-0.9.0jar包下载连接](https://mvnrepository.com/artifact/io.github.novacrypto/ToRuntime/0.9.0)
下载完jar包放到项目里 groupId artifactId随便写 但是千万不能重复![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040800505477.jpg#pic_center)

```javascript
<!-- fastdfs所需jar包依赖[注：这里是在本地lib中引入，maven中央仓库中暂无此jar包]，要与<includeSystemScope>true</includeSystemScope>配合使用-->
        <dependency>
            <groupId>io.github.novacrypto4</groupId>
            <artifactId>io.github.novacrypto</artifactId>
            <version>1.0</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/src/main/resources/lib/BIP32-0.0.9.jar</systemPath>
        </dependency>
        <dependency>
            <groupId>io.github.novacrypto3</groupId>
            <artifactId>io.github.novacrypto</artifactId>
            <version>1.0</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/src/main/resources/lib/BIP39-0.1.9.jar</systemPath>
        </dependency>
        <dependency>
            <groupId>io.github.novacrypto2</groupId>
            <artifactId>io.github.novacrypto</artifactId>
            <version>1.0</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/src/main/resources/lib/BIP44-0.0.3.jar</systemPath>
        </dependency>
        <dependency>
            <groupId>io.github.novacrypto1</groupId>
            <artifactId>io.github.novacrypto</artifactId>
            <version>1.0</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/src/main/resources/lib/SHA256-0.0.1.jar</systemPath>
        </dependency>
        <dependency>
            <groupId>io.github.novacrypto</groupId>
            <artifactId>io.github.novacrypto</artifactId>
            <version>1.0</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/src/main/resources/lib/ToRuntime-0.9.0.jar</systemPath>
        </dependency>
```

```javascript
 	<plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <!-- 作用:项目打成jar的同时将本地jar包也引入进去 -->
        <configuration>
            <includeSystemScope>true</includeSystemScope>
        </configuration>
    </plugin>
```


## 4.创建账户（离线创建）
创建账户是不需要连接节点的 创建账户只是一种算法 离线创建就好
废话少说直接上代码 没什么难点 注释部分是生成keystore文件的 需要就放出来 filepath 生成文件路径
address就是账户地址 privateKey是私钥 后面基本就用这两个 privateKey一定保存好 用它可以直接免密码操作账户 重置密码等操作

```javascript
public JSONObject createAccount(String pwd) {

//        String filePath = "/lim";
        StringBuilder sb = new StringBuilder();
        byte[] entropy = new byte[Words.TWELVE.byteLength()];
        new SecureRandom().nextBytes(entropy);
        new MnemonicGenerator(English.INSTANCE).createMnemonic(entropy, sb::append);
        String mnemonic = sb.toString();
        List mnemonicList = Arrays.asList(mnemonic.split(" "));
        byte[] seed = new SeedCalculator().withWordsFromWordList(English.INSTANCE).calculateSeed(mnemonicList, pwd);
        ECKeyPair ecKeyPair = ECKeyPair.create(Sha256.sha256(seed));
        String privateKey = ecKeyPair.getPrivateKey().toString(16);
        String publicKey = ecKeyPair.getPublicKey().toString(16);
        String address = "0x" + Keys.getAddress(publicKey);
        //创建钱包地址与密钥
//        String fileName = null;
//        try { fileName = WalletUtils.generateWalletFile(pwd, ecKeyPair, new File(filePath), true);
//        } catch (Exception e) {  e.printStackTrace(); }
//        if (fileName == null) { return null; }
//        String accountFilePath = filePath + File.separator + fileName;
        JSONObject json = new JSONObject();
        json.put("privateKey", privateKey);
        json.put("publicKey", publicKey);
        json.put("address", address);
        return json;
    }
```

## 5.geth节点搭建、基本使用及一般问题

**1.节点搭建及基本使用**
后面的操作需要连接节点操作 所以这里先来搭建一个节点
这里使用docker容器 我直接连了主网 没有搭私链
```javascript
首先时间2020.4.8 fast方式同步区块数据230G 数据会越来越大 先保证服务器磁盘空间足够 比区块数据多个200G比较好 空间小会降低同步效率 空间不够就无法成功
1 拉取镜像
docker pull ethereum/client-go
2 运行geth
//--name容器名 -v /webdata/ethdata 指定区块数据存放目录（后面不要动）--syncmode=fast 
//指定同步模式 fast模式 另外还有full light 下边会贴个图说明
//--rpcaddr 0.0.0.0 向所有ip开放rpc连接 要么指定ip 要么配置安全组 不然一天之内必被攻击
docker run -d --name ethereum -p 8545:8545 -p 30303:30303  -v /webdata/ethdata:/root/.ethereum ethereum/client-go --syncmode=fast --cache=1024 --rpc --rpcaddr 0.0.0.0  --rpcport 8545 --rpcapi "web3,eth,net,personal,admin,,txpool" --rpccorsdomain "*"
3 进入geth镜像
docker exec -it ethereum sh
4 进入geth终端
geth attach ipc://root/.ethereum/geth.ipc   geth的js控制台
5 查看区块同步情况//下面常见问题会具体讲
eth.syncing 
6 查看blockNumber（块高）//下面常见问题会具体讲
eth.blockNumber
6 查看以太坊账户
personal.listAccounts
7创建以太坊账户
personal.newAccount("admin")	密码admin 返回的是账户地址
8查看账户余额
web3.eth.getBalance(personal.listAccounts[0])  下标第几个账户
9 账户秘钥存放位置（没什么用）
ls /root/.ethereum/keystore/
10 复制镜像内文件到本地 （没什么用）
docker cp 镜像id:镜像路径  本地路径
docker ps查看镜像id 也可以用name
拓展：
docker ps -a 查看容器 包括未运行的
docker kill 镜像id  停止
docker restart 重启镜像
docker logs -f ethereum（镜像名） 查看geth实时日志
docker logs -f --tail=50 ethereum 只打印最后50行日志，因为日志多了全部打印太慢
//geth很吃内存和cpu 我刚启动的时候，发现服务器里的项目访问不了了  服务器繁忙 
//top看一下 cpu被geth占用99% 然后我采用cpulimit来限制geth使用cpu
# cd /usr/local
# wget -O cpulimit.zip https://github.com/opsengine/cpulimit/archive/master.zip
# unzip cpulimit.zip //没装unzip的先跑一下 yum install -y unzip zip
# cd cpulimit-master
# make
# sudo cp src/cpulimit /usr/bin
//限制geth使用cpu 40%
nohup cpulimit -e geth -l 40 &   
```
**2. 常见问题**

```javascript

1.刚启动节点eth.syncing看同步情况返回false
首先看一下日志有没有报错，上面有看日志的命令，正常的话说明还没有开始同步，五分钟之后再看一下
2.eth.syncing 怎么看出同步情况
首先 看到currentBlock highestBlock等数据说明正在同步，currentBlock是当前同步的块高，
highestBlock是总块高 当currentBlock = highestBlock 的时候eth.syncing会返回false说明同步完成
3.currentBlock 总是比highestBlock少一百甚至几十
首先 highestBlock是不断增加的 这个情况持续时间在2天左右是正常的 一般同步需要3-7天 只差一百左右的时候大概也要1-3天才能同步完成
第二 服务器配置不能太低 至少也要4核8G
第三 保证剩余磁盘空间充足，剩余空间小会影响同步效率
第四 如果是通过本博客搭建的geth，看一下cpulimit是否设置的过低了 杀掉cpulimit进程 重新限制为60
4.eth.blockNumber返回0||账户中明明有以太币，查询结果却是0
在fast同步模式下 需要完成同步之后才可以获取到链上数据，返回0说明还没同步完
5.区块已经同步完成了，过了几天，发现eth.blockNumber比区块链浏览器上的低几百甚至上千，以至于监听不到数据（先提一下，后面会讲监听）
geth是会有卡住的情况，影响不大，只要逻辑上处理妥善不会有数据丢失，docker kill docker restart重启一下

```
**3.同步模式拓展**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200408215507939.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjcwNDM1Ng==,size_16,color_FFFFFF,t_70#pic_center)
##  6.常量类 后面内容都要用到（必备！！！！！！）

```javascript
public class EthConstant {

    //redis key 存储用户的账户地址集
    public static final String USER_ETH_ADDRESS_JSON_KEY = "user.eth.address.json.key";

    //redis key 块的高度
    public static final String ETH_BLOCK_NUMBER_KEY = "eth.block.number.key";

    //ERC20_USDT合约地址
    public static final String CONTRACTA_DDRESS = "0xdac17f958d2ee523a2206206994597c13d831ec7";

    //geth客户端
    public static final String SERVER = "http://xx.xx.xx.xx:8545";

    //gas price
    public static final String GAS_PRICE = "5";

    //gas limit
    public static final String GAS_LIMIT = "100000";

    //入账账户
    public static final String IN_ADDRESS = "0x000";

    //出账账户
    public static final String OUT_ADDRESS = "0x000";

    //出账账户私钥
    public static final String OUT_KEY = "your private key";
    
}
```

**简单说下gas price 和gas limit
首先回顾一下单位**
```javascript

	WEI("wei", 0),
    KWEI("kwei", 3),
    MWEI("mwei", 6),
    GWEI("gwei", 9),
    SZABO("szabo", 12),
    FINNEY("finney", 15),
    ETHER("ether", 18),
    KETHER("kether", 21),
    METHER("mether", 24),
    GETHER("gether", 27);
    
```
***wei = e0 = 1 
kwei = e3 = 1000***

gas price 是你为你的操作愿意出的单价，一般单位是GWEI，单价越高，交易处理的速度越快，同时交易消耗的gas也就越多
gas limit 是gas price的基础上 你最多愿意出多少份，也就是说gas price * gas limit 是本次交易消耗的最大的燃料数，既然说到最大，也就意味着交易不一定会把你设定的gas limit全部消耗掉，实际消耗多少就扣多少，gas limit 只是你的底线，提到一点，如果交易中出了异常，那你的gas price * gas limit 会被消耗光，是不会返还的，所有开发中可以把gas limit 调到适当的值 即使出错也不会损失很大



##  7.查询以太币余额
废话少说直接上代码 这里很简单

```javascript
	public BigInteger getBalance(String address) {
		Web3j web3 = Web3j.build(new HttpService(EthConstant.SERVER));
        BigInteger banlance = new BigInteger("0");
        try {
            banlance = web3j.ethGetBalance(address, DefaultBlockParameterName.LATEST).send().getBalance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        //这里返回的单位是wei 以太币的单位是ether
        //如果需要转成ether
        //BigDecimal ether = Convert.fromWei(banlance.toString(), Convert.Unit.ETHER);
        return banlance;
    }
    
```
## 8.以太币交易
也很简单，直接上代码

```javascript
	public String ethTran(String from,String to, BigInteger num) throws Exception {
	
		Web3j web3 = Web3j.build(new HttpService(EthConstant.SERVER));
		String privateKey = "通过from账户拿到私钥";
        Credentials credentials = Credentials.create(privateKey );

        EthGetTransactionCount ethGetTransactionCount = web3j.ethGetTransactionCount(
                from, DefaultBlockParameterName.LATEST).sendAsync().get();

        BigInteger nonce = ethGetTransactionCount.getTransactionCount();

        RawTransaction rawTransaction = RawTransaction.createEtherTransaction(
                nonce, Convert.toWei(EthConstant.GAS_PRICE, Convert.Unit.GWEI).toBigInteger(),
                Convert.toWei(EthConstant.GAS_LIMIT, Convert.Unit.WEI).toBigInteger(), to, num);
        byte[] signedMessage = TransactionEncoder.signMessage(rawTransaction, credentials);
        String hexValue = Numeric.toHexString(signedMessage);

        EthSendTransaction ethSendTransaction = web3j.ethSendRawTransaction(hexValue).send();
        if (ethSendTransaction.hasError()) {
            log.info("transfer error:", ethSendTransaction.getError().getMessage());
            throw new Exception(ethSendTransaction.getError().getMessage());
        } else {
            String transactionHash = ethSendTransaction.getTransactionHash();
            log.info("Transfer transactionHash:" + transactionHash);
            return transactionHash;
        }
    }
```

## 9.智能合约编译java文件
首先下载web3j命令行工具
[web3j命令行工具下载连接](https://github.com/lim960/Eth/blob/master/web3j-3.2.0.zip)
下载zip解压出来 可以看到bin和lib两个文件夹 进入bin 可以看到web3j 
**通过合约abi bin 可以用web3j生成对应合约的java类文件**
[ERC20-USDT查看合约信息连接](https://etherscan.io/address/0xdac17f958d2ee523a2206206994597c13d831ec7#code)
**点击contract可以看到合约代码，abi,bin**
![](https://img-blog.csdnimg.cn/20200409074424791.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjcwNDM1Ng==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409074721462.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjcwNDM1Ng==,size_16,color_FFFFFF,t_70#pic_center)


**在web3j命令行工具bin目录下新建两个txt 分别把abi bin内容复制进来 然后改名为EthContract.abi   EthContract.bin**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409081505794.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjcwNDM1Ng==,size_16,color_FFFFFF,t_70#pic_center)
然后当前目录cmd打开命令行


```javascript
web3j solidity generate EthContract.bin EthContract.abi -o java -p com.tes.eth
//-o生成的java文件存放目录 -p设置文件所在的包
```
下面附上我编译好的java文件及bin，abi，合约源码
[下载连接](https://github.com/lim960/Eth/tree/master/ERC20_USDT%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6)
然后把java文件添加到项目中 就可以使用合约方法直接操作代币了
**合约中方法简单讲一下 首先使用之前需要load，之后可以调用balanceOf是查询余额， transfer是代币交易，其他的用到再解释**

## 10.使用智能合约查询代币余额
**坑点说明讲解**
**1.因为区块链数据是公开的 账户数据也是公开的  任何人都可以查看任何账户的余额、交易情况
所以这里查询操作之前创建Credentials 证书时可以使用任何账户的私钥来创建
2.USDT和Ether的小数点位不同，ether是18位，USDT是6位，链上的数据的单位都是wei，也就是说，如果要转成个数，ether是要除以e18的，而usdt是除以e6，所以
Convert.fromWei(banlance.toString(), Convert.Unit.MWEI) 这里单位要特别注意 MWEI**
```javascript
	public BigInteger getUsdtBalance(String address) {

		Web3j web3 = Web3j.build(new HttpService(EthConstant.SERVER));
        BigInteger banlance = new BigInteger("0");
        try {
            Credentials credentials = Credentials.create(EthConstant.OUT_KEY);
            EthContract contract = EthContract.load(EthConstant.CONTRACTA_DDRESS, web3j, credentials, 
             Convert.toWei(EthConstant.GAS_PRICE, Convert.Unit.GWEI).toBigInteger(), Convert.toWei(EthConstant.GAS_LIMIT, Convert.Unit.WEI).toBigInteger());
            banlance = contract.balanceOf(address).send();
        } catch (Exception e) {
            e.printStackTrace();
        }
        //这里返回的单位是wei  如果需要转成个数
        //BigDecimal ether = Convert.fromWei(banlance.toString(), Convert.Unit.MWEI);
        return banlance;
    }
```

## 11.代币交易
**同样这里注意单位 MWEI GWEI WEI**
```javascript
	public JSONObject TranUsdt(String from, String to, String num) throws Exception{

        Web3j web3 = Web3j.build(new HttpService(EthConstant.SERVER));
        String sendKey = "";//from账户的私钥 根据自己情况去查
        Credentials credentials = Credentials.create(sendKey);
        EthContract contract = EthContract.load(EthConstant.CONTRACTA_DDRESS, web3j, credentials,
                Convert.toWei(EthConstant.GAS_PRICE, Convert.Unit.GWEI).toBigInteger(), Convert.toWei(EthConstant.GAS_LIMIT, Convert.Unit.WEI).toBigInteger());
        TransactionReceipt result = contract.transfer(to, Convert.toWei(num, Convert.Unit.MWEI).toBigInteger()).send();
        //result 中有交易hash，块高，gas使用数等数据 去TransactionReceipt类中看 
        return null;
    }
```

## 12.交易监听
**先讲一下使用场景和常见问题解答
以太坊交易不是实时到账的，那么交易发送出去了怎么知道到没到账呢，首先想到可以定时任务根据交易返回的hash去查，那如果交易是外部发出的呢（在其他平台向你平台的账户转账），如果定时去查交易记录数，比较交易数有变化再处理就很复杂了，所以这里使用监听，可以指定监听的记录类型，如Transfer，只要合约上有有交易的记录，这里就可以监听到。
1那么这个监听器要什么时候运行呢？
应该项目一启动就开始监听
2如果服务器挂了会不会数据丢失呢？
监听可以指定块的开始数结束数，只要记录号已经处理过的数据的块高，数据就基本不会丢失
3. 合约内transferEventObservable方法是监听transfer函数，可以看一下**

**下面开始上代码**
先建一个注册类
**这里只有一个注意点 fromblock 从redis中拿 拿不到就取最新的 不用redis根据自己情况改**
```javascript
@Configuration
public class ContractConfig {

    @Resource
    private IRedisService redisService;

    @Bean
    @Scope("prototype")
    public Web3j web3j() {
        return Web3j.build(new HttpService(EthConstant.SERVER));
    }

    @Bean
    @Autowired
    public EthContract ethContract(Web3j web3j) throws IOException {
        EthContract contract;
        try {
            Web3ClientVersion web3ClientVersion = web3j.web3ClientVersion().send();
            String clientVersion = web3ClientVersion.getWeb3ClientVersion();
            System.out.println("clientVersion" + clientVersion);
            // 以某个用户的身份调用合约
            TransactionManager transactionManager = new ClientTransactionManager(web3j, EthConstant.OUT_ADDRESS);
            //加载智能合约
            contract = EthContract.load(EthConstant.CONTRACTA_DDRESS, web3j, transactionManager
                    , Convert.toWei(EthConstant.GAS_PRICE, Convert.Unit.GWEI).toBigInteger()
                    , Convert.toWei(EthConstant.GAS_LIMIT, Convert.Unit.WEI).toBigInteger());
            return contract;
        } catch (Exception e) {
            e.printStackTrace();
            throw e;
            //throw new RRException("连接智能合约异常");
        }
    }

    @Bean
    //监听这里才用每次都生成一个新的对象，因为同时监听多个事件不能使用同一个实例
    @Scope("prototype")
    @Autowired
    public EthFilter filter(EthContract ethFilter, Web3j  web3j) throws IOException {
        //获取启动时监听的区块
        Request<?, EthBlockNumber> request = web3j.ethBlockNumber();
        BigInteger fromblock = request.send().getBlockNumber();
        System.out.println(fromblock);
        String redisBlockNum = redisService.getString(EthConstant.ETH_BLOCK_NUMBER_KEY);
        if(redisBlockNum != null){
            fromblock = new BigInteger(redisBlockNum);
        }
        return new EthFilter(DefaultBlockParameter.valueOf(fromblock),
//        return new EthFilter(DefaultBlockParameter.valueOf(new BigInteger("9818217")),
                DefaultBlockParameterName.LATEST, EthConstant.CONTRACTA_DDRESS);
    }

}
```
再建一个监听的类

```javascript

/**
 * 服务监听器，继承ApplicationRunner，在spring启动时启动
 */
@Slf4j
@Component
public class ServiceRunner implements ApplicationRunner {


    @Autowired
    private Web3j web3j;

    //如果多个监听，必须要注入新的过滤器
    @Autowired
    private EthFilter ethFilter;

    @Resource
    private IRedisService redisService;

    @Override
    public void run(ApplicationArguments var1) throws Exception{
        uploadProAuth();
        log.info("This will be execute when the project was started!");
    }


    /**
     * 收到上链事件
     */
    public void uploadProAuth(){
        Event event = new Event("Transfer",
                Arrays.<TypeReference<?>>asList(new TypeReference<Address>() {}, new TypeReference<Address>() {}),
                Arrays.<TypeReference<?>>asList(new TypeReference<Uint256>() {}));

        ethFilter.addSingleTopic(EventEncoder.encode(event));
        String redisBlockNum = redisService.getString(EthConstant.ETH_BLOCK_NUMBER_KEY);
        //blockNumber不是唯一的 减小redis压力
        //lambda表达式中无法修改外部变量 因为lambda表达式执行时 外部变量已经被释放 需要用数组方式可以实现
        final BigInteger[] num = new BigInteger[1];
        num[0] = redisBlockNum == null ? new BigInteger("0") : new BigInteger(redisBlockNum);

        log.info("启动监听transfer");
        web3j.ethLogObservable(ethFilter).subscribe(log -> {

            this.log.info("收到ERC20合约transfer事件");
            BigInteger blockNum = log.getBlockNumber();
            //逻辑上只能大于等于 注册的时候已经取了redis中的block 暂不处理小于情况
            if(blockNum.compareTo(num[0]) == 1){
                num[0] = blockNum;
                redisService.setString(EthConstant.ETH_BLOCK_NUMBER_KEY,blockNum.toString());
            }
            EventValues eventValues = Contract.staticExtractEventParameters(event, log);
            String from = (String) eventValues.getIndexedValues().get(0).getValue();
            String to = (String) eventValues.getIndexedValues().get(1).getValue();
            BigInteger value = (BigInteger) eventValues.getNonIndexedValues().get(0).getValue();
            String transactionHash = log.getTransactionHash();
            this.log.info("from: " + from + " to: " + to + " value: " + value);//1111111111111111111111111111111111
            this.log.info("hash: " + transactionHash + " block: " + blockNum);//222222222222222222222222
            this.log.info(JSON.toJSONString(log));//222222222222222222222222

            //写自己的逻辑 判断to账户是不是自己平台的账号 是的话加余额 加交易记录 加上链记录
        });
    }
}
```
**下面大概分享一下我的做法
1.创建账户的时候把地址放进redis**
```javascript
		//先说一下存储形式
		//value格式
		//	{
		//		“address0”:1,
		//		“address2”:1,
		//		“address3”:1
		//	}
		//每次创建账户更新 存储平台全部账户地址
		//用户地址放入redis的账户地址集 redis一个value最大512M 足够存放
        JSONObject addJson = null;
        String jsonValue = redisService.getString(EthConstant.USER_ETH_ADDRESS_JSON_KEY);
        if(jsonValue == null){
            addJson = new JSONObject();//此处还有优化空间
        } else {
            addJson = JSONObject.parseObject(jsonValue);
        }
        addJson.put(address,1);
        redisService.setString(EthConstant.USER_ETH_ADDRESS_JSON_KEY, JSON.toJSONString(addJson));
```
**2.刚才监听处处理 
直接复制到 //写自己的逻辑 判断to账户是不是自己平台的账号 是的话加余额 加交易记录 加上链记录 
这里**

```javascript
		String add = redisService.getString(EthConstant.USER_ETH_ADDRESS_JSON_KEY);
        if(add != null){
            JSONObject addJson = JSONObject.parseObject(add);
            if(addJson.getIntValue(to) == 1){
            	//说明是需要处理的数据 进行处理
            }
        }
```
## 13.代币的代理交易
**NO1:讲一下使用场景**
a要转账给b 10个usdt，而a没有ether，无法支付燃料，所以一般需要先给a充值才可以进行操作，但是这样要支付两次手续费，所以用代理交易的方式便可以解决这个问题。
代理交易就是a转账给b，但是手续费由c来支付。需要两步
1.a账号授权给c账号可操作的额度（坑点，授权也需要手续费，所以一次授权额度大一些）
2.c来进行a转给b的操作
**NO2:讲一下需要用到的合约方法**
approve 授权给一个地址一定的额度
transferFrom 比transfer多一个from参数 指定哪个账户转给哪个账户多少usdt 数量不能比剩余授权的额度高
allowance  两个参数_owner ，_spender    查询剩余的_owner  授权给_spender 的额度
**NO3:注意事项**
**1.下面上代码 这里出账账户就是c(支付手续费的账户)
2.注意单位   WEI  MWEI  GWEI  FINNEY  ETHER**

```javascript
	public String proxyTranUsdt(String from, String to, String num) throws Exception {

        BigInteger tranNum = Convert.toWei(num,Convert.Unit.MWEI).toBigInteger();//交易数量转成biginteger
        String sendKey = "";//发送方私钥
        if (from.equals(EthConstant.OUT_ADDRESS)){
            sendKey = EthConstant.OUT_KEY;
        } else {
            //根据自己情况查from账户的私钥
        }
        //from账户的证书
        Credentials credentials = Credentials.create(sendKey);
        //from账户的合约
        EthContract contract = EthContract.load(EthConstant.CONTRACTA_DDRESS, web3j, credentials,
            Convert.toWei(EthConstant.GAS_PRICE, Convert.Unit.GWEI).toBigInteger(), Convert.toWei(EthConstant.GAS_LIMIT, Convert.Unit.WEI).toBigInteger());
        //出账账户的证书
        Credentials outCreden = Credentials.create(EthConstant.OUT_KEY);
        //出账账户的合约
        EthContract outContract = EthContract.load(EthConstant.CONTRACTA_DDRESS, web3j, outCreden,
                Convert.toWei(EthConstant.GAS_PRICE, Convert.Unit.GWEI).toBigInteger(), Convert.toWei(EthConstant.GAS_LIMIT, Convert.Unit.WEI).toBigInteger());
        //usdt余额
        BigInteger balance = contract.balanceOf(from).send();
        log.info(balance.toString());
        if(tranNum.compareTo(balance) == 1){
            //余额不足
            throw new Exception("USDT余额不足");
        }
        //EthRecordEntity ethRecord = null;//上链记录
        //授权的数量
        BigInteger allow = contract.allowance(from, EthConstant.OUT_ADDRESS).send();
        log.info(allow.toString());
        if(tranNum.compareTo(allow) == 1){
            //授权数量不足
            //以太币余额
            BigInteger ethBalance = web3j.ethGetBalance(from, DefaultBlockParameterName.LATEST).send().getBalance();
            log.info(ethBalance.toString());
            BigInteger fee = Convert.toWei("5",Convert.Unit.FINNEY).toBigInteger();//0.005以太
            if(fee.compareTo(ethBalance) == 1){
                //余额不足0.005 可能不够授权的费用 给账户转账 转完余额0.01以太 免得麻烦
                //补足0.01要转的数量
                BigInteger tranEthNum = Convert.toWei("10",Convert.Unit.FINNEY).toBigInteger().subtract(ethBalance);
                String hash = this.ethTran(from,tranEthNum);

                //添加上链记录
                //ethRecord = new EthRecordEntity();......
            }
            if(allow.compareTo(new BigInteger("0")) == 1){
                //已经授权过 需要先重置为0
                TransactionReceipt app = contract.approve(EthConstant.OUT_ADDRESS, new BigInteger("0")).send();
                log.info(app.toString());
                //添加上链记录
                //ethRecord = new EthRecordEntity();......
            }
            TransactionReceipt app = contract.approve(EthConstant.OUT_ADDRESS
                    , Convert.toWei("1",Convert.Unit.GWEI).toBigInteger()).send();//一次授权一千个
            log.info(app.toString());
            //添加上链记录
           //ethRecord = new EthRecordEntity();......
        }
        TransactionReceipt send = outContract.transferFrom(from,to, Convert.toWei(num, Convert.Unit.MWEI).toBigInteger()).send();
        log.info(send.toString());
        //添加上链记录
      	//ethRecord = new EthRecordEntity();......
        return null;
    }
```
## 14.大概说一下sendAsync()和send()及使用send()时的处理方法
字面意思 一个同步一个异步
同步就是要等待处理完才继续执行，如先approve再进行transferFrom，如果使用异步就可能出问题，还没授权成功就进行交易，这样首先不会成功，而且gas price * gas limit会被全吞，本文使用的全都是send()，这样会带来一个问题，接口等待时间太长，响应无效了，处理方式如下
调用上链操作时异步执行

        Async.run(() -> 
                proxyTranUsdt(address,address,num) 
        );

## 15.分享一下我的上链记录class

```javascript
@Data
public class EthRecordEntity implements Serializable {

    @ApiModelProperty(value = "主键id")
    private String id;

    @ApiModelProperty(value = "Transaction Hash")
    private String tranHash;

    @ApiModelProperty(value = "from账户")
    private String fromAddress;

    @ApiModelProperty(value = "to账户")
    private String toAddress;

    @ApiModelProperty(value = "币种类型 USDT/Ether")
    private String kind;

    @ApiModelProperty(value = "function approve/transfer")
    private String functionName;

    @ApiModelProperty(value = "交易数量 BigInteger")
    private String num;

    @ApiModelProperty(value = "交易数量 处理后的个量")
    private String numStr;

    @ApiModelProperty(value = "gas price")
    private String gasPrice;

    @ApiModelProperty(value = "gas limit")
    private String gasLimit;

    @ApiModelProperty(value = "使用的gas量")
    private String gasUsed;

    @ApiModelProperty(value = "块高度号")
    private String blockNumber;

    @ApiModelProperty(value = "创建时间")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date createTime;

```
