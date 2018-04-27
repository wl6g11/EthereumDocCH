.. _mining:

********************************************************************************
挖矿
********************************************************************************

简介
================================================================================

挖矿这个词源于对加密货币与黄金的类比。黄金或贵金属很稀有，电子代币也是，增加总量的唯一方法就是挖矿。以太坊也是这样，发行的唯一办法就是挖矿。但是不像其他例子，挖矿也是通过在区块链中创建、验证、发行和传播区块来保护网络的方法。

- 挖以太币 = 保护网络 = 验证计算

什么是挖矿?
--------------------------------------------------------------------------------
以太坊，和所有区块链技术一样，使用激励驱动的安全模式。共识基于选择具有最高总难度的区块。矿工创造区块，其他人检测有效性。区块只有在包含特定 **难度** 的 **工作量** (POW)时才有效，还有其他合格性条件。请注意到以太坊Serenity里程碑，可能就会被取代（参考 :ref:`proof of stake model <POS vs POW>` ）。

以太坊区块链在很多方面与比特币区块链类似，但也有些不同。在区块链架构方面，以太坊和比特币之间最主要的的区别是，不像比特币，以太坊区块不仅包含交易列表也包含最近状态（merkle patricia trie结构的根hash编码在状态中更精确）除此之外，另外两个值，区块数和难度，也储存在区块中。

使用的工作量证明算法叫 `Ethash <https://github.com/ethereum/wiki/wiki/Ethash>`_ （ `Dagger-Hashimoto algorithm <https://github.com/ethereum/wiki/wiki/Dagger-Hashimoto>`_ 的改良版本），包括找到算法的 **随机数** 输入以使结果低于特定的难度阈值。工作量证明算法的意义在于，要找到这样一个随机数，没有比列举可能性更好的策略，而解决方法的验证琐碎又廉价。由于输出有均匀分布（是散表功能应用的结果），我们可以保证，平均而言，需要找到这样一个随机数的时间取决于难度阈值。这使得只通过操纵难度来控制找到新区块的时间成为可能。

正如协议中所描述的，难度动态调整的方式是每15秒整个网络会产生一个区块。我们说网络用 **15秒区块时间** 生产一个区块链。这个"心跳"基本上主要强调系统状态同步，保证不可能维持一个分叉（允许double spend）或被恶意分子重写历史，除非攻击者有半数以上的网络挖矿能力（即所谓的 **51%** 攻击）。

任何参与到网络的节点都可能是矿工，预期的挖矿收益和他们的（相对）挖矿能力或者说 *hashrate* 成正比，比如被网络总散表率标准化的，每秒尝试的随机数数量。

Ethash工作量证明是 **内存难解** 的，这使它能 **抵抗ASIC** 。内存难解性由工作量证明算法实现，需要选择依靠随机数和区块标题的固定资源的子集合。这个资源（几十亿字节大小的数据）叫做 **DAG** 。每3000个区块的 `DAG <https://github.com/ethereum/wiki/wiki/Ethash-DAG>`_ 完全不同，125小时的窗口叫做 **epoch**（大约5.2天），需要一点时间来生成。由于DAG只由区块高度决定，它可以被事先生成，如果没有被事先生成，客户端需要等到进程最后来生产区块。如果客户端没有预生成并提前缓存DAG，网络可能会在每个epoch过渡经历大规模区块延迟。注意不必要生成DAG以验证工作量证明，它可以在低CPU和小内存的状态下被验证。

在特殊情况下，从零开始创建节点的时候，只有在为现存epoch创建DAG的时候才会开始挖矿。

挖矿奖励
--------------------------------------------------------------------------------

获奖区块的成功工作量证明矿工会获得：

* "获胜"区块的 *静态区块奖* ，包含5.0个以太币
* 区块内支出的gas成本 — 一定数量的以太币，取决于当前gas价格
* 叔伯块的额外奖励，形式是每个叔伯块包含额外的1/32

在区块中执行所有交易所消费的、由获胜矿工提交的gas都由每个交易的发送者支付。已发生的gas成本归到矿工账户作为共识协议的一部分。随着时间变化，这会使数据区块奖变得矮小。

*叔伯块* 是稳定的区块，比如说，和包含先前区块（最多回6个区块）的父区块。有效的叔伯块会受到奖励以中和网络滞后给挖矿奖励带来的影响，因而提升安全性（这叫做GHOST协议）。叔伯块由成功工作量证明矿工形成的区块中所包含的叔伯块接收7/8的数据区块奖励（=4.375以太币）。每个区块最多允许2个叔伯块。

    * `Uncles ELI5 on reddit <https://www.reddit.com/r/ethereum/comments/3c9jbf/wtf_are_uncles_and_why_do_they_matter/>`_
    * `Forum thread explaining uncles <https://forum.ethereum.org/discussion/2262/eli5-whats-an-uncle-in-ethereum-mining>`_

挖矿的成功取决于设定的区块难度。区块难度动态调整每个区块，以规定网络散列能力来创造12秒区块时间。找到区块的机会因此由与难度相关的散列率产生。

Ethash DAG
--------------------------------------------------------------------------------
Ethash将 *DAG*（有向非循环图）用于工作量证明算法，这是为每个 *epoch* 生成，例如，每3000个区块（125个小时，大约5.2天）。DAG要花很长时间生成。如果客户端只是按需要生成它，那么在找到新epoch第一个区块之前，每个epoch过渡都要等待很长时间。然而，DAG只取决于区块数量，所以可以预先计算来避免在每个epoch过渡过长的等待时间。 ``Geth`` 和 ``ethminer`` 执行自动的DAG生成，每次维持2个DAG以便epoch过渡流畅。挖矿从控制台操控的时候，自动DAG生成会被打开和关闭。如果 ``geth`` 用 ``--mine`` 选项启动的时候，也会默认打开。注意客户端分享DAG资源，如果你运行任何客户端的多个实例，确保自动的DAG生成只在一个实例中打开。

为任意epoch生成DAG：

.. code-block:: bash

    geth makedag <block number> <outputdir>

例如 ``geth makedag 360000 ~/.ethash`` 。请注意ethash为DAG使用 ``~/.ethash`` (Mac/Linux) 或 ``~/AppData/Ethash`` (Windows)，这样它可以在不同的客户端实现以及多个运行实例中分享。

算法
================================================================================

我们的算法， `Ethash <https://github.com/ethereum/wiki/wiki/Ethash>`_ （之前被称为Dagger-Hashimoto），是基于一个大的、瞬时的、任意生成的、形成DAG（Dagger-part）的资料组规定，尝试解决它一个特定的约束，部分通过区块标题散列来决定。

它被设计用于在一个只有慢CPU的环境中来散列快速验证时间，但在被提供大量高带宽内存时，为挖矿提供大量的加速。大量内存需求意味着大规模矿工获得相对少的超线性利益。高带宽需求意味着从堆在很多超速处理单元、分享同样内存的加速在每个单独的单元给出很少的利益。没有节点验证的利益因而阻碍中心化，这在挖矿中很重要。

外部挖矿应用和以太坊工作规定和报送的后台程序之间的交流通过JSON-RPC API发生。提供两个RPC功能； ``eth_getWork`` 和 ``eth_submitWork`` 。

这些被正式记录在维基百科文章 `miner <https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console#miner>`_ 的 `JSON-RPC API <https://github.com/ethereum/wiki/wiki/JSON-RPC>`_ 条目下。

为了挖矿你需要一个完全同步的、能够挖矿的以太坊客户端和至少一个以太坊账户。这个账户用于发送挖矿奖励，通常被称为 **货币基** 或 **以太基** 。查看 ":ref:`creating_an_account`" 章节，学习如何创建帐户。

.. warning:: 开始挖矿前，确保区块链和主链完全同步，否则就不能在主链上挖矿。

CPU 挖矿
================================================================================

你可以用电脑的中央处理器（CPU）挖以太币。自从GPU矿工的效率高出两个数量级，它就不再盈利了。然而你可以用CPU挖掘在Morden测试网或私有链上挖矿，以便创建你测试合约和交易所需要的以太币， 而无需花费实时网络上的真实以太币。

.. note:: 测试网以太币除了用于测试目的外没有其他价值（查看 :ref:`test-networks` ）。

使用geth
--------------------------------------------------------------------------------

用 ``geth`` 启动以太坊节点时，并不是默认挖掘。在CPU挖掘模式开启，你会用 ``--mine`` `命令行选项 <https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options>`_ 。 ``--minerthreads`` 参数可以用于设置平行于挖掘线程的数量（默认为处理器核心的总数）。

``geth --mine --minerthreads=4``

你也可以在执行期间用 `console <https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console#adminminerstart>`_ 开启或停止CPU挖掘。 ``miner.start`` 取一个矿工线程数量的可选参数。

.. code-block:: Javascript

    > miner.start(8)
    true
    > miner.stop()
    true

注意挖掘真实以太币只有在你与网络同步时才有意义（由于你是在共识区块顶部挖矿）。因此以太区块链下载器/同步器会延迟挖掘直到同步完成，此后挖掘自动开始，除非你用 ``miner.stop()`` 取消挖矿。

为了赚取以太币，你必须有 **etherbase** （或 **coinbase** ）地址集。这个etherbase默认为你的第一个账户。如果你没有etherbase地址， ``geth --mine``就不会开启。

你可以在命令行重新设置etherbase：

.. code-block:: bash

    geth --etherbase 1 --mine  2>> geth.log // 1 is index: second account by creation order OR
    geth --etherbase '0xa4d8e9cae4d04b093aac82e6cd355b6b963fb7ff' --mine 2>> geth.log

你也可以在控制台重新设置etherbase：

.. code-block:: javascript

    miner.setEtherbase(eth.accounts[2])

注意你的etherbase不必是本地账户地址，只要是现存的就可以。

有一个给你挖掘过的区块 `添加额外数据 <https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console#minersetextra>`_ （只有32字节）的选项。按照惯例，它被解释为统一码字符串，你可以设置短期虚荣标签。

.. code-block:: javascript

    miner.setExtra("ΞTHΞЯSPHΞЯΞ")
    ...
    debug.printBlock(131805)
    BLOCK(be465b020fdbedc4063756f0912b5a89bbb4735bd1d1df84363e05ade0195cb1): Size: 531.00 B TD: 643485290485 {
    NoNonce: ee48752c3a0bfe3d85339451a5f3f411c21c8170353e450985e1faab0a9ac4cc
    Header:
    [
    ...
            Coinbase:           a4d8e9cae4d04b093aac82e6cd355b6b963fb7ff
            Number:             131805
            Extra:              ΞTHΞЯSPHΞЯΞ
    ...
    }

你可以用 `miner.hashrate <https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console#adminminerhashrate>`_ 检查算力，结果用H/s表示（每秒hash操作）。

.. code-block:: javascript

    > miner.hashrate
    712000

成功挖掘一些区块以后，你可以检查etherbase账户中的以太币余额。现在假定你的etherbase是个本地账户：

.. code-block:: javascript

    > eth.getBalance(eth.coinbase).toNumber();
    '34698870000000'

为了花费你赚的gas来交易，你需要解锁账户。

.. code-block:: javascript

    > personal.unlockAccount(eth.coinbase)
    Password
    true

你可以在控制台上用以下代码片段，检查哪个区块被特殊的矿工（地址）挖掘过：

.. code-block:: javascript

    function minedBlocks(lastn, addr) {
      addrs = [];
      if (!addr) {
        addr = eth.coinbase
      }
      limit = eth.blockNumber - lastn
      for (i = eth.blockNumber; i >= limit; i--) {
        if (eth.getBlock(i).miner == addr) {
          addrs.push(i)
        }
      }
      return addrs
    }
    // scans the last 1000 blocks and returns the blocknumbers of blocks mined by your coinbase
    // (more precisely blocks the mining reward for which is sent to your coinbase).
    minedBlocks(1000, eth.coinbase);
    //[352708, 352655, 352559]

请注意，发现一个区块但是不能把它变成典型链会经常发生。这意味着你在当地把挖过的区块包括在内，当前的状态会显示归于你账户的挖矿奖励，然而不久后，会发现更好的链，我们转换到不包含你区块的链，因而不会记入任何挖矿奖励。因此很有可能矿工监控coinbase余额的时候会发现，它发生了相当程度的浮动。

GPU挖矿
================================================================================

硬件
--------------------------------------------------------------------------------

算法是内存难解的，为了使DAG适合内存，每个GPU需要1-2GB内存，如果你得到错误提示： ``Error GPU mining. GPU memory fragmentation?`` 说明你没有足够的内存。GPU挖矿软件是基于OpenCL实现的，AMD GPU会比同一水准的NVIDIA GPU“更快”。ASIC和FPGA相对低效因而被阻拦。要给芯片集成平台获取openCL，尝试：

- `AMD SDK openCL <http://developer.amd.com/tools-and-sdks/opencl-zone/amd-accelerated-parallel-processing-app-sdk>`_
- `NVIDIA CUDA openCL <https://developer.nvidia.com/cuda-downloads>`_

Ubuntu Linux设置
-------------------------

对于这个快速指南，你会需要Ubuntu 14.04或15.04以及fglrx图像驱动器。你也可以使用NVidia驱动器和其他平台，但是你必须要找到自己的方式来获得有效的OpenCL安装，比如 `Genoil's ethminer fork <http://cryptomining-blog.com/tag/ethminer/>`_ 。

如果你在用15.04，转到"软件与更新 > 额外的驱动器"设置为"从fglrx为AMD图形加速器使用视频驱动器"。

如果你在用14.04，转到"软件与更新 > 额外的驱动器"设置为"从fglrx为AMD图形加速器使用视频驱动器"。很遗憾，对于一些人来说，这种方法可能不管用，因为Ubuntu 14.04.02中有个已知的程序错误会阻止你转换到GPU挖矿所必须的专属图形驱动器。

所以，如果你遇到这个程序错误，先到"软件与更新 > 更新"选择"预发行的可靠更新提议"。然后，回到"软件与更新 > 额外的驱动器"设置为"从fglrx为AMD图形加速器使用视频驱动器"。重启之后，值得检查一下现在确实正确安装了驱动器（例如通过再到"额外驱动器"）。

不管做什么，如果你在用14.04.02，一旦安装之后，就不要改变驱动器或者驱动器配置。例如，aticonfig –-initial的使用（尤其是-f, –-force选项）会"破坏"你的设置。如果你偶然改变了配置，会需要卸载驱动器，重启，再次安装驱动器并重启。

Mac设置
-------------------------------

.. code-block:: bash

 wget http://developer.download.nvidia.com/compute/cuda/7_0/Prod/local_installers/cuda_7.0.29_mac.pkg
 sudo installer -pkg ~/Desktop/cuda_7.0.29_mac.pkg -target /
 brew update
 brew tap ethereum/ethereum
 brew reinstall cpp-ethereum --with-gpu-mining --devel --headless --build-from-source

查看冷却状态：

.. code-block:: bash

  aticonfig --adapter=0 --od-gettemperature

Windows设置
-------------------------------

`下载最新的Eth++安装包 <https://github.com/ethereum-mining/ethminer/releases>`_ ，在安装界面的"选择组件"页面选择ethminer。

..  image:: img/eth_miner_setup.png
..   :height: 513px
..   :width: 399 px
   :alt: ethereum-ethminer-set-upfdg

用geth使用ethminer
-------------------------------

.. code-block:: bash

    geth account new // Set-up ethereum account if you do not have one
    geth --rpc --rpccorsdomain localhost 2>> geth.log &
    ethminer -G  // -G for GPU, -M for benchmark
    tail -f geth.log

``ethminer`` 在端口8545（geth的默认RPC端口）和geth沟通。你可以通过给 ``geth`` ``--rpcport`` 选项来改变这种情况。ethminer会在任何端口发现geth。注意你需要用 ``--rpccorsdomain localhos`` 设置CORS标题。你也可以用 ``--Fhttp://127.0.0.1:3301`` 在 ``ethminer`` 设置端口。如果你想要在同一个电脑上挖几个实例，设置端口是必需的，尽管有些没有意义。如果你在私有链上测试，我们推荐你用CPU挖掘代替。

.. note:: 你 **不需要** 把 ``--mine`` 选项给 ``geth`` ，或者在控制台开启挖矿，除非你想要在GPU挖掘顶端做CPU挖掘。

如果 ``ethminer`` 的默认无效，试试用 ``--opencl-device X`` 来规定OpenCL装置，其中X是{0, 1, 2,…}。用 ``-M`` （基础测试程序）运行 ``ethminer`` 时，你会看到如下的提示：

.. code-block:: bash

    Benchmarking on platform: { "platform": "NVIDIA CUDA", "device": "GeForce GTX 750 Ti", "version": "OpenCL 1.1 CUDA" }


    Benchmarking on platform: { "platform": "Apple", "device": "Intel(R) Xeon(R) CPU E5-1620 v2 @ 3.70GHz", "version": "OpenCL 1.2 " }

调试 ``geth`` :

.. code-block:: bash

    geth  --rpccorsdomain "localhost" --verbosity 6 2>> geth.log

调试挖矿:

.. code-block:: bash

    make -DCMAKE_BUILD_TYPE=Debug -DETHASHCL=1 -DGUI=0
    gdb --args ethminer -G -M

..  note:: GPU挖矿时，散列率信息在 ``geth`` 上不可用。

用 ``ethminer`` 检查散列率， ``miner.hashrate`` 总会报告0。

用eth使用ethminer
--------------------------------------------------

在单独的GPU上挖矿
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

为了在单独的GPU上挖矿，只需要用以下参数运行eth：

.. code-block:: bash

 eth -v 1 -a 0xcadb3223d4eebcaa7b40ec5722967ced01cfc8f2 --client-name "OPTIONALNAMEHERE" -x 50 -m on -G

- ``-v 1`` 将冗长的信息设置为1。不要被信息刷屏。
- ``-a YOURWALLETADDRESS`` 设置挖矿奖励会去的coinbase。以上地址只是一个例子。这一参数十分重要，确保不要在钱包地址出错，否则会接收不到以太币支出。
- ``--client-name "OPTIONAL"`` 设置可选择的客户端名称，在网络上确定身份。
- ``-x 50`` 请求大量的端点。帮助在开始找到端点。
- ``-m on`` 在挖矿开启的状态下实际启动。
- ``-G`` 打开GPU挖掘。

客户端运行时，你可以用geth附属或 `ethconsole <https://github.com/ethereum/ethereum-console>`_ 和它互动。

在多个GPU上挖矿
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

用多个GPU和eth挖矿与用geth和多个GPU挖矿十分相似。确保eth节点和正确设置的coinbase地址一起运行：

.. code-block:: bash

   eth -v 1 -a 0xcadb3223d4eebcaa7b40ec5722967ced01cfc8f2 --client-name "OPTIONALNAMEHERE" -x 50 -j

注意我们也添加了-j参数以使客户端有可用的JSON-RPC服务器与ethminer实例沟通。此外由于ethminer可以为我们挖矿，我们移除了与挖矿相关的参数。每个GPU都会执行一个不同的ethminer实例：

.. code-block:: bash

   ethminer --no-precompute -G --opencl-device X

X是索引号码，与你想ethminer用{0, 1, 2,…}的OpenCL装置一致。为了轻松获取OpenCL装置列表，你可以执行 ``ethminer --list-devices`` ，它会提供一个OpenCL可以检测到的所有装置，以及每个装置的一些附加信息。

下面是一个示例输出：

.. code-block:: console

 [0] GeForce GTX 770
     CL_DEVICE_TYPE: GPU
     CL_DEVICE_GLOBAL_MEM_SIZE: 4286345216
     CL_DEVICE_MAX_MEM_ALLOC_SIZE: 1071586304
     CL_DEVICE_MAX_WORK_GROUP_SIZE: 1024

最终 ``--no-precompute`` 参数请求ethiminers不要提前创建下一个epoch的DAG。尽管不推荐这样，因为每次epoch过渡的时候，你都会有一个挖矿中断。

基准测试程序
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

挖矿能力通常以内存带宽衡量。我们的实现写在OpenCL上，很典型地在NVidia上被AMD GPU支持得更好。实验证据确认了在价格方面，AMD GPU比对应的NVidia挖矿表现更好。

用基准程序测试单一装置设置，你可以在基准测试程序模式下通过-M使用ethminer。

.. code-block:: bash

   ethminer -G -M

如果你有很多装置，你会喜欢分别用基准程序测试，可以用 ``–opencl-device`` 选项，与之前章节相似:

.. code-block:: bash

 ethminer -G -M --opencl-device X

用 ethminer ``--list-devices`` 来列出可能的数字替代X {0, 1, 2,…}。

开始在Windows上挖矿，首先要 `下载geth windows binary <https://build.ethereum.org/builds/Windows%20Go%20master%20branch/>`_ 。

* 解压缩Geth (单击右键选择打开)，启用命令提示符。用cd导航到Geth数据文件夹的位置(例如 ``cd /`` 到 ``C:`` 盘)。
* 输入 ``geth --rpc`` 开启geth。

进入以后，以太坊区块链会开始下载。有时候防火墙肯能会阻止同步进程（阻止时会有提示）。如果被阻止，点击"允许进入"。

* 首先 `下载安装ethminer <http://cryptomining-blog.com/tag/ethminer-cuda-download/>`_ ， C++挖矿软件 (防火墙或Windows本身可能会有反应，允许进入)。
* 打开另一个命令提示符 (保持第一个运行！)输入 ``cd/Program\ Files/Ethereum(++)/release`` 改变目录。
* 确保 ``geth`` 完成区块链同步。如果同步不再进行，就可以在命令提示符输入 ``ethminer -G`` 开启挖矿进程。

此时可能会出现一些问题。如果有错误发生， 可以输入 ``Ctrl+C`` 来中断矿工。如果错误显示(提示）"内存不足"，就说明没有足够的GPU内存来挖以太币。

矿池挖矿
================================================================================

矿池挖矿是旨在通过联合参与矿工的挖矿力来解决预期收益问题的合作社（挖矿的矿工的算力来解决预期收益问题的合作组织）。作为回报，通常收取0-5%的挖矿奖励。挖矿池从中央账户用工作量证明提交区块并按照参与人贡献的挖矿力比例来重新分配奖励。

.. warning:: 大多数挖矿池包含第三方，中心组件，意味着他们是不需信任的。换言之，挖矿池操作人可以把你的收入拿走。谨慎操作。有很多具备开源数据库、不需信任的、去中心化的挖矿池。

.. warning:: 挖矿池只会外包工作量证明运算，他们不会使区块生效或运行虚拟机来检查执行交易带来的状态过渡。 这能有效地使挖矿池在安全方面像单个节点一样表现，他们的增长会造成`51% 攻击 <https://learncryptography.com/cryptocurrency/51-attack>`_ 的中心化威胁。确保遵守网络能力分配，不要让挖矿池长得太大。

矿池
--------------------------------------------------------------------

* `coinotron`_
* `nanopool`_
* `ethpool`_ - Predictable solo mining, unconventional payout scheme, affiliated with `etherchain\.org`_.
* `supernova`_
* `coinmine.pl`_
* `eth.pp.ua`_
* `talkether`_ - Unconventional payout scheme, partially decentralized
* `weipool`_
* `ethereumpool`_
* `pooleum`_
* `alphapool`_
* `cryptopool`_
* `unitedminers`_
* `2miners`_
* `dwarfpool`_ - Try to avoid this (currently over 50% of the network)
* `laintimes <http://pool.laintimes.com/>`_ - Discontinued

.. _Ethpool: https://github.com/etherchain-org/ethpool-core
.. _Ethpool source: https://github.com/etherchain-org/ethpool-core
.. _ethereumpool: https://ethereumpool.co/
.. _nanopool: http://eth.nanopool.org/
.. _pooleum: http://www.pooleum.com
.. _alphapool: http://www.alphapool.xyz/
.. _dwarfpool: http://dwarfpool.com/eth
.. _talkether: http://talkether.org/
.. _weipool: http://weipool.org/
.. _supernova: https://eth.suprnova.cc/
.. _coinmine.pl: https://www2.coinmine.pl/eth/
.. _eth.pp.ua:  https://eth.pp.ua/
.. _coinotron: https://www.coinotron.com/
.. _etherchain.org: https://etherchain.org/
.. _unitedminers: http://eth.unitedminers.cloud/
.. _cryptopool: http://ethereum.cryptopool.online/
.. _2miners: https://2miners.com/


挖矿资源
=======================================================

* `Top miners of last 24h on etherchain <https://etherchain.org/statistics/miners>`_
* `pool hashrate distribution for august 2015 <http://cryptomining-blog.com/5607-the-current-state-of-ethereum-mining-pools/>`_
* `Unmaintained list of pools on Forum <https://forum.ethereum.org/discussion/3659/list-of-pools>`_
* `Mining profitability calculator on cryptocompare <https://www.cryptocompare.com/mining/calculator/eth>`_
* `Mining profitability calculator on cryptowizzard <http://cryptowizzard.github.io/eth-mining-calculator/>`_
* `Mining profitability calculator on etherscan <http://etherscan.io/ether-mining-calculator/>`_
* `Mining profitability calculator on In The Ether <http://ethereum-mining-calculator.com/>`_
* `Mining difficulty chart on etherscan <http://etherscan.io/charts/difficulty>`_


.. _POS vs POW:

POS vs POW
-----------------------------

* https://www.reddit.com/r/ethereum/comments/38db1z/eli5_the_difference_between_pos_and_pow/
* https://blog.ethereum.org/2014/11/25/proof-stake-learned-love-weak-subjectivity/
* https://www.reddit.com/r/ethereum/comments/42o8oy/can_someone_explain_the_switch_to_pos_how_and_when/
