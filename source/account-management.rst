********************************************************************************
账户管理
********************************************************************************

.. _Accounts:

账户
================================================================================

账户在以太坊中发挥着中心作用。共有两种账户类型：*外部账户* （EOAs）和 *合约账户* 。
我们这里重点讲一下外部账户，以下会简称为 *账户* 。合约账户简称为 *合约* ，在 :ref:`合约章节 <Contracts>` 具体讨论。
把外部账户和合约账户都归入到帐户的一般概念是合理的，因为这些实体都是所谓的 *状态对象* 。
这些实体都有状态：账户有余额，合约既有余额也有合约储存。所有账户的状态正是以太坊网络的状态，以太坊网络和每个区块一起更新，
网络需要达成关于以太坊的共识。对于用户通过交易和以太坊区块链互动来说，账户是必不可少的。

如果我们把以太坊限制为只有外部账户，只允许外部账户之间进行交易，我们就会进入到"代币"系统，"代币"系统不如比特币本身有力，只能用于转移以太币。

账户代表着外部代理人（例如人物角色，挖矿节点 ，或是自动代理人）的身份。账户运用公钥加密图像来签署交易以便以太坊虚拟机可以安全地验证交易发送者身份。

钥匙文件
================================================================================

每个账户都由一对钥匙定义，一个私钥和一个公钥。账户以 *地址* 为索引，地址由公钥衍生而来，取公钥的最后 20个字节。
每对私钥/地址都编码在一个 *钥匙文件* 里。钥匙文件是JSON文本文件，可以用任何文本编辑器打开和浏览。
钥匙文件的关键部分，账户私钥，通常用你创建帐户时设置的密码进行加密。钥匙文件可以在以太坊节点数据目录的 ``keystore`` 子目录下找到。
确保经常给钥匙文件备份！查看 :ref:`backup-and-restore-accounts` 章节了解更多。

创建钥匙和创建帐户是一样的。

* 不必告诉任何人你的操作。
* 不必和区块链同步。
* 不必运行客户端。
* 甚至不必连接到网络。

当然新账户不包含任何以太币。但它将会是你的，你大可放心，没有你的钥匙和密码，没有人能进入。

在以太坊节点间转换整个目录或任何个人钥匙文件都是安全的。

.. Warning:: 请注意万一你从一个不同的节点向另一个节点添加钥匙文件，账户的顺序可能发生改变。确保不要回复或改变手稿中的索引或代码片段。

.. _creating_an_account:

创建账号
================================================================================

.. Warning:: 记住密码并 :ref:`备份钥匙文件 <backup-and-restore-accounts>`。为了从账号发送交易，包括发送以太币，
你必须同时有钥匙文件和密码。确保钥匙文件有个备份并牢记密码，尽可能安全地存储它们。
这里没有逃亡路径，如果钥匙文件丢失或忘记密码，就会丢失所有的以太币。
没有密码不可能进入账号，也没有忘记密码选项。所以一定不要忘记密码。


使用 ``geth account new``
--------------------------------------------------------------------------------

一旦安装了geth客户端，创建账号只需要在终端执行 ``geth account new`` 指令的问题了。

注意不必运行geth客户端或者和区块链同步来使用 ``geth account`` 指令。

.. code-block:: Bash

  $ geth account new

    Your new account is locked with a password. Please give a password. Do not forget this password.
    Passphrase:
    Repeat Passphrase:
    Address: {168bc315a2ee09042d83d7c5811b533620531f67}

对于非交互式使用，你可以提供纯文本密码文件作为 ``--password`` 标志的变量。文件中的数据包含密码的原始字节，后面可选择单独跟着新的一行。

.. code-block:: Bash

  $ geth --password /path/to/password account new

..  Warning:: 用 ``--password`` 标志只是为了测试或在信任的环境中自动操作。不建议将密码保存在文件中或以任何其他方式暴露。如果 ``--password`` 标志的变量是密码文件，要确保文件只对你自己可阅读和列表。你可以在 Mac/Linux系统中通过以下指令实现：

.. code-block:: Bash

  touch /path/to/password
  chmod 600 /path/to/password
  cat > /path/to/password
  >I type my pass

要列出目前在你的 ``keystore`` 文件夹中的钥匙文件的所有账号，使用 ``geth account`` 指令的 ``list`` 子指令：
.. code-block:: Bash

  $ geth account list

  account #0: {a94f5374fce5edbc8e2a8697c15331677e6ebf0b}
  account #1: {c385233b188811c9f355d4caec14df86d6248235}
  account #2: {7f444580bfef4b9bc7e14eb7fb2a029336b07c9d}

钥匙文件的文件名格式为 ``UTC--<created_at UTC ISO8601>-<address hex>`` 。账号列出时是按字母顺序排列，但是由于时间戳格式，实际上它是按创建顺序排列。

使用geth控制台
--------------------------------------------------------------------------------

为了用geth创建新账号，我们必须先在控制台模式开启geth（或者可以用 ``geth attach`` 将控制台依附在已经运行着的实例上）：

.. code-block:: Bash

  > geth console 2>> file_to_log_output
  instance: Geth/v1.4.0-unstable/linux/go1.5.1
  coinbase: coinbase: [object Object]
  at block: 865174 (Mon, 18 Jan 2016 02:58:53 GMT)
  datadir: /home/USERNAME/.ethereum

控制台使你能够通过发出指令与本地节点互相作用。比如，试一下这个列出账号的指令：

.. code-block:: Javascript

  > eth.accounts

  {
  code: -32000,
  message: "no keys in store"
  }

这表示你没有账号，你也可以从控制台创建一个账号：

.. code-block:: Javascript

  > personal.newAccount()
  Passphrase:
  Repeat passphrase:
  "0xb2f69ddf70297958e582a0cc98bce43294f1007d"

.. Note:: 记得用一个安全性强、随机生成的密码。

我们刚刚创建了第一个账号。如果我们再次试着列出账号，就可以看到新创建的账号了：

.. code-block:: Javascript

  > eth.accounts
  ["0xb2f69ddf70297958e582a0cc98bce43294f1007d"]


.. _using-mist-ethereum-wallet:

使用Mist以太坊钱包
--------------------------------------------------------------------------------

对于命令行的反面，现在有一个基于GUI的选项可以用来创建账号："官方"Mist以太坊钱包。 Mist以太坊钱包，和它的父项目Mist, 是在以太坊基金会的赞助下开发，因此是"官方"地位。钱包应用有Linux, Mac OS X和Windows可用的版本。

.. Warning:: Mist钱包是试用软件，使用需风险自担。

用GUI Mist以太坊钱包创建账号再容易不过了。事实上，第一个账号在应用安装期间就创建出来了。


1. 根据你的操作程序 `下载钱包应用最新版本  <https://github.com/ethereum/mist/releases>`_ 。由于你实际上会运行一个完整的geth节点，打开钱包应用就会开始同步复制你电脑上的整个以太坊区块链。

2. 解压下载的文件，运行以太坊钱包可执行文件。

.. image:: img/51Downloading.png
   :width: 582px
   :height: 469px
   :scale: 75 %
   :alt: downloading-mist
   :align: center

3. 等待区块链完全同步，按照屏幕上的说明操作，第一个账号就创建出来了。

4. 第一次登录Mist 以太坊钱包，你会看到自己在安装过程中创建的账号。它会被默认命名为主账号（ETHERBASE）。

.. image:: img/51OpeningScreen.png
   :width: 1024px
   :height: 938px
   :scale: 50 %
   :alt: opening-screen
   :align: center

5. 再另外创建账号很容易；只需点击应用主界面上的添加账号，输入所需的密码即可。

.. Note:: Mist钱包仍在开发中，以上列出的具体步骤可能会随着更新有所变更。

在 Mist创建多签名钱包
--------------------------------------------------------------------------------

Mist以太坊钱包有个选项是可以用多签名钱包使钱包里的余额更安全。用多签名钱包的好处是它需要多个账号共同批准才能够从余额中提取大额资金。创建多签名钱包之前，需要创建多个账号。

在Mist创建账号文件很容易。在 `账号` 菜单下点击 `添加账号` 。选择一个安全性高又容易记住的密码（记住没有密码找回选项），确认，账号创建就完成了。创建至少 2个账号。如果你愿意，第二个账号可以在另一台有Mist运行的电脑上创建（理论上这样可以使多签名更加安全）。你只需要第二个账号的公钥（存款地址）来创建多签名钱包（复制/粘贴，不要手动输入）。因为需要第一个账号来创建多签名钱包合约，所以第一个账号必须是在你创建多签名钱包的电脑上。

既然已经创建了账号，保持安全并进行备份（如果不备份，电脑系统崩溃，余额就会丢失）。点击菜单顶端的 `备份` 。选择 `keystore` 文件夹，反向点击 / 选择 `复制`（不要选择 `剪切` ，否则结果会很糟糕）。回到桌面，在空白区域反击，选择"粘贴"。你可能会想把这个"keystore"文件夹重命名为"以太坊－keystore－备份－年－月－日"，这样以后就能很快辨认出来。这时候你就能把文件夹内容添加到压缩文件里（如果是在线备份，最好用另外一个安全性高又容易记住的密码对档案进行密码保护 ），复制到U盘，刻录到CD/DVD ，或者上传到在线存储设备（ Dropbox/Google Drive等）。

你现在应该添加大约不到 0.02以太币到第一个账号里（那个用来创建多签名钱包的账号）。这是创建多签名钱包所需的交易费用。另外再需要1以太币（或者更多），因为 Mist现在需要这样做来确保钱包合约交易有足够的"gas"来正常执行……所以对新人来说，总共需要不到 1.02以太币。

创建多签名钱包的时候，你会进入到附属在它上面所有账号的完整地址。我推荐把每个地址复制/粘贴到简单的文本编辑器上（notepad / kedit等），到Mist每个账号的详情页以后，从右侧按键栏里选择"复制地址"选项。不要手动输入地址，或者冒着输入错误的风险，你可能会把交易发送到错误的地址，因此丢失余额。

我们现在准备好了创建多签名钱包。在"钱包合约"下，选择"增加钱包合约"。起个名字，选择第一个账号持有人，选择"多签名钱包合约"。你会看到出现这样的文字：

"这是由X个持有人共同控制的联合账号。每天最多可以发送X个以太币。任何超过每日限额的交易都需要 X个持有人确认。"

设置附属在这个多签名钱包上的持有人(账号)数量，每日提款限额(这只要求一个账号提出这些钱款，以及允许多少持有人(账号)批准超过每日限额的提款。

现在加入之前复制/粘贴在文本编辑器中的账号地址，确认所有的设置正确后，点击底部的"创建"按钮。然后需要输入密码发送交易。在"钱包合约"部分，会显示出新的钱包，告诉你"创建"。

钱包创建完成后，就能在屏幕上看到合约地址。选择整个地址，复制/粘贴到文本编辑器的新文件里，保存至桌面，命名为"以太坊-钱包-地址.txt"或其他名称。

现在只需用备份合约文件的方式来备份"以太坊-钱包-地址.txt"，接着就能用这个地址在ETH装载新的多签名钱包。

如果你要从备份中恢复，只需要复制"以太坊–keystore–备份"文件夹里的文件到这个攻略第一部分里提到的"keystore"文件夹。如果是在从未安装过Mist的机器上安装（第一次创建账号的同时就会建立文件夹），可能需要创建"keystore"文件夹。如果要恢复多签名钱包，不要像我们创建之前一样选择"多签名钱包合约"，只选择"导入钱包"就可以了。

故障排查：

* Mist不能同步。一个有用的解决方案是将个人电脑硬件时钟与NTP服务器同步，确保时间无误后重启。

* Mist同步后启动，但出现了白屏。有可能是因为你在基于Linux的操作系统上运行了 "xorg" 视频驱动器（Ubuntu, Linux Mint等），试试安装制造商的视频驱动器。

* 提示"密码错误"。在现在的Mist版本上，这有可能是个错误的提示。重启Mist，问题就能解决（如果你输入的确实是正确密码）。

使用 Eth
--------------------------------------------------------------------------------

Every option related to key management available using geth can be used the same way in eth.
与使用geth的可用钥匙管理相关的每个选项都同样适用于eth。

以下是与"账号"相关的选项：

.. code-block:: Javascript

  > eth account list  // List all keys available in wallet.
  > eth account new   // Create a new key and add it to the wallet.
  > eth account update [<uuid>|<address> , ... ]  // Decrypt and re-encrypt given keys.
  > eth account import [<uuid>|<file>|<secret-hex>] // Import keys from given source and place in wallet.

以下是与"钱包"有关的选项：

.. code-block:: Javascript

  > eth wallet import <file> //Import a presale wallet.

.. Note:: "账号导入"选项只能用于导入一般的钥匙文件。"钱包导入"选项只能用于导入预售钱包。

也可以从综合控制台进入钥匙管理（用内置控制台或者geth附件）：

.. code-block:: Javascript

  > web3.personal
  {
	listAccounts: [],
	getListAccounts: function(callback),
	lockAccount: function(),
	newAccount: function(),
	unlockAccount: function()
  }

使用EthKey （弃用）
--------------------------------------------------------------------------------

Ethkey是C++实现的CLI工具，可以让你和以太坊钱包互动。你可以用它罗列、检查、创建、删除和修改钥匙，以及检查、创建和签署交易。

我们假定你还没有运行过客户端，比如eth或者Aleth系列的任何客户端。如果你运行过，可以略过这一章节。

要创建钱包，用 ``creatwallet`` 指令运行 ``ethkey`` ：

.. code-block:: Bash

  > ethkey createwallet

请输入管理员密码来保护keystore（设一个安全性高的！）：会问你要一个"管理员"密码。这能保护你的隐私，并且它会默认为你任何钥匙的密码。你需要再次输入同一文本来进行确认。

.. Note:: 使用安全性高的、随机生成的密码。

我们可以通过使用列表指令简单地列出钱包内的钥匙：

.. code-block:: Bash

  > ethkey list

  No keys found.

我们还没创建任何钥匙，它也是这样告诉我们的！我们来创建一个吧。

要创建钥匙，我们需要用 ``new`` 指令，需要通过一个名字——这也是我们要给钱包里账号的名字。我们称之为"测试"：

.. code-block:: Bash

  > ethkey new test

输入密码来保护这个账号（或者用管理员密码就不用输入了）。这会促使你输入密码来保护这个钥匙。如果你只点击回车，就会使用默认的"管理员"密码。这意味着，当你想用账号的时候，不必输入钥匙密码（因为它记住了管理员密码）。总体来说，你应该试着为每个钥匙设置一个不同的密码，因为这样能防止一个密码被盗用而导致其他账号也被入侵。然而为了方便你可能会决定让低安全性的账号使用同一个密码。

在这里，我们用一个极富想象力的密码123（永远不要用这么简单的密码，除非是暂时的测试账号）。输入密码后，它就会让你再次输入确认。再次输入123。由于你设置了它的密码，它会让你提供一个密码提示，每次进入的时候都会显示密码提示。提示会储存在钱包里，由管理员密码保护。我们来输入糟糕的密码提示321倒序。

.. code-block:: Bash

  > ethkey new test

  Enter a passphrase with which to secure this account (or nothing to use the master passphrase):
  Please confirm the passphrase by entering it again:
  Enter a hint to help you remember this passphrase: 321 backwards
  Created key 055dde03-47ff-dded-8950-0fe39b1fa101
    Name: test
    Password hint: 321 backwards
    ICAP: XE472EVKU3CGMJF2YQ0J9RO1Y90BC0LDFZ
    Raw hex: 0092e965928626f8880629cec353d3fd7ca5974f

所有正常（或者说直接）的 ICAP地址都以XE开头，这样就很容易辨认。请注意这个钥匙在创建的钥匙后有另外一个标识符，被称为UUID。这是个特有的钥匙标识符，和账号本身毫无关系。知道了它对攻击者发现你在网上的身份毫无帮助。它刚好也是钥匙的文件名，你可以在~/.web3/keys (Mac或Linux)或者$HOME/AppData/Web3/keys (Windows)中发现。让我们通过列出钱包里的钥匙来确认它在正常运行：

.. code-block:: Bash

  > ethkey list
  055dde03-47ff-dded-8950-0fe39b1fa101 0092e965… XE472EVKU3CGMJF2YQ0J9RO1Y90BC0LDFZ  test

它每行会报告一个钥匙（这里总共只有一个钥匙）。在这个例子里，钥匙被储存在055dde... 文件，有个以XE472EVK...开头的ICAP地址。这不容易记住，所以有个专有名称会很有帮助，还是叫test吧。

导入预售钱包
================================================================================


使用Mist以太坊钱包
--------------------------------------------------------------------------------

用GUI Mist以太坊钱包导入预售钱包非常简便。实际上，在应用安装期间你会被问到是否要导入预售钱包。

.. Warning:: Mist钱包是试用软件。使用风险自担。

安装Mist以太坊钱包的说明在 :ref:`Creating an account: Using Mist Ethereum wallet <using-mist-ethereum-wallet>` 章节给出。

只需要把 ``.json`` 预售钱包文件拖放到指定区域，输入密码，导入预售钱包。


.. image:: img/51PresaleImportInstall.png
   :width: 582px
   :height: 469px
   :scale: 75 %
   :alt: presale-import
   :align: center

如果你选择不在应用安装期间导入预售钱包，以后你可以随时导入，只需选择应用菜单栏下方的 ``账号`` 菜单，然后选择 ``导入预售账号`` 。

.. Note:: Mist钱包仍在开发中，以上列出的具体步骤可能会随着更新有所变更。

使用geth
--------------------------------------------------------------------------------

如果你单独安装geth，导入预售钱包可以通过在终端执行以下操作完成：

.. code-block:: Bash

  geth wallet import /path/to/my/presale-wallet.json

这会提示你输入密码。

更新账号
================================================================================

你可以把钥匙文件更新到最新的钥匙文件格式并且/或者升级钥匙文件密码。

使用geth
--------------------------------------------------------------------------------

你可以在命令行用 ``更新`` 子命令更新现在的账号，可以使用账号地址或者索引作为参数。记住账号索引反映了创建顺序（按字母顺序排列的钥匙文件名包含了创建时间）。

.. code-block:: Bash

  geth account update b0047c606f3af7392e073ed13253f8f4710b08b6

或

.. code-block:: Bash

  geth account update 2

例如:

.. code-block:: Bash

  $ geth account update a94f5374fce5edbc8e2a8697c15331677e6ebf0b

  Unlocking account a94f5374fce5edbc8e2a8697c15331677e6ebf0b | Attempt 1/3
  Passphrase:
  0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b
  account 'a94f5374fce5edbc8e2a8697c15331677e6ebf0b' unlocked.
  Please give a new password. Do not forget this password.
  Passphrase:
  Repeat Passphrase:
  0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b

账户以加密的形式储存在最新版本，它会提示你需要一个密码来解锁账户，另一个密码来保存更新的文件。同一个指令还可以用在将弃用格式的账户变成最新版本或者改变账户密码。

对于非交互式使用，密码可以用 ``--password`` 标志详细说明：

.. code-block:: Bash

  geth --password <passwordfile> account update a94f5374fce5edbc8e2a8697c15331677e6ebf0bs

由于只能给出一个密码，所以只能执行格式更新，修改密码只在交互式的情况下才有可能。

.. Note:: 账号更新有个副作用就是会引起账号顺序变化。更新成功后，同一钥匙所有之前的格式/版本都会被移除！

.. _backup-and-restore-accounts:

账号备份和恢复
================================================================================

手动备份/恢复
--------------------------------------------------------------------------------

要从账号发送交易，需要有账号钥匙文件。钥匙文件可以在以太坊节点数据目录的钥匙商店（keystore）子目录下找到。默认数据目录的位置与平台相关：

- Windows: ``%appdata%\Ethereum\keystore``
- Linux: ``~/.ethereum/keystore``
- Mac: ``~/Library/Ethereum/keystore``

要备份钥匙文件（账号），在 ``keystore`` 子目录中复制单独的钥匙文件或复制整个 ``keystore`` 文件夹。

要恢复钥匙文件（账号），将钥匙文件重新复制到 ``keystore`` 子目录，即其原始地址。

导入未加密私钥
--------------------------------------------------------------------------------

导入未加密私钥由 ``geth`` 支持

.. code-block:: Bash

  geth account import /path/to/<keyfile>

这个指令从纯文本文件 ``<keyfile>`` 导入未加密私钥并创建新账号和打印地址。钥匙文件被假定包含未加密私钥作为编码到十六进制的标准EC原始字节。账号以加密的形式储存，会提示你输入密码。你需要记住密码用于以后解锁账号。

下面给出一个例子，详细说明数据目录。如果 ``--datadir`` 标志没有使用，新账户就会被创建在默认数据目录里，即钥匙文件会被放在数据目录的 ``keystore`` 子目录里。

.. code-block:: Bash

  $ geth --datadir /someOtherEthDataDir  account import ./key.prv
  The new account will be encrypted with a passphrase.
  Please enter a passphrase now.
  Passphrase:
  Repeat Passphrase:
  Address: {7f444580bfef4b9bc7e14eb7fb2a029336b07c9d}

对于非交互式使用，密码可以用 ``--password`` 标志详细说明：

.. code-block:: Bash

  geth --password <passwordfile> account import <keyfile>

.. Note:: 因为你可以直接把加密账户复制到另一个以太坊事例中，在节点之间转移账号的时候就不需要这个导入/导出机制了。

.. Warning:: 当你往已存在节点的 ``keystore`` 里复制钥匙的时候，你习惯的账户顺序可能会改变。因此要保证你不依赖于账户顺序，否则就要进行复核并更新脚本中使用的索引。