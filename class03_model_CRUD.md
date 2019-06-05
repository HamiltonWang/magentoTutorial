# 什麼是 Model?

Model 其實有點複雜而且很多步驟和做法，我們先從理解開始。

## 基本概論

他其實有 3 個基本檔案。

1. `Model` - 把商業邏輯寫在這，簡單來說，就是透過資料庫存取來製作你需要的 function
2. `ResourceModel` - CRUD
3. `Collection` - 其實他比以上 2 位更高層的功能，是個綜合前 2 個的類似像 ORM 的 Library，而且自己有 Cache 功能。
4. `Repository` - 這是 Magento 的一個標準，其實就是多加個 Interace 讓對外規格固定。

其他：`Factory` - 在 Model 中快速的存取資料庫資料，但沒有 Cache 功能。

我們先討論前 3 個，第四個是對 external lobrary 如 REST API 比較有幫助。

結論：
功能越強大，越多 code。

## 實作一下
假設我們想要以下的設定

    Namespace: Prince
    Module: Hello
    Database Table: tb_hello
    ID: id

請自己在資料庫新增一個 table

p.s. 因為 Magento 是屬於有經驗的工程師的套件工具，所以太基本的東西我們基本上就不過度記載

### 1. 製作一個 Model: Hello.php Model
檔案位置：`app/code/Prince/Helloworld/Model/Hello.php`

    <?php

    namespace Prince\Helloworld\Model;

    use Magento\Framework\Model\AbstractModel;

    class Hello extends AbstractModel
    {
        /**
        * Define resource model
        */
        protected function _construct()
        {
            $this->_init('Prince\Helloworld\Model\ResourceModel\Hello');
        }
    }

### 2.  新增一個 ResourceModel: Hello.php
檔案位置：`app/code/Prince/Helloworld/Model/ResourceModel/Hello.php`

    <?php

    namespace Prince\Helloworld\Model\ResourceModel;

    use Magento\Framework\Model\ResourceModel\Db\AbstractDb;

    class Hello extends AbstractDb
    {
        /**
        * Define main table
        */
        protected function _construct()
        {
            $this->_init('tb_hello', 'id'); //hello is table of module
        }
    }


### 3.  新增一個 Collection：Collection.php、
檔案位置：`app/code/Prince/Helloworld/Model/ResourceModel/Hello/Collection.php`


    <?php

    namespace Prince\Helloworld\Model\ResourceModel\Hello;

    use Magento\Framework\Model\ResourceModel\Db\Collection\AbstractCollection;

    class Collection extends AbstractCollection
    {
        /**
        * Define model & resource model
        */
        protected function _construct()
        {
            $this->_init(
                'Prince\Helloworld\Model\Hello',
                'Prince\Helloworld\Model\ResourceModel\Hello'
            );
        }
    }

你現在可以在你前一個章節製作的 Controller 來測試你的 Collection 
或是另外製作個 `app/code/Prince/Helloworld/Controller/Index/Index.php`

程式範例：

    <?php

    namespace Prince\Helloworld\Controller\Index;

    use Magento\Framework\App\Action\Action;
    use Magento\Framework\App\Action\Context;
    use Prince\Helloworld\Model\HelloFactory;

    class Index extends Action
    {
        protected $_modelHelloFactory;

        public function __construct(
            Context $context, 
            HelloFactory $modelHelloFactory
        ) {
            parent::__construct($context);
            $this->_modelHelloFactory = $modelHelloFactory;
        }

        public function execute()
        {

            $resultPage = $this->_modelHelloFactory->create();
            $collection = $resultPage->getCollection(); //Get Collection of module data
            var_dump($collection->getData());
            exit;

        }
    }


### Factory 是什麼？

其實這非常常用，而且應該是最常的用法。
每次 Magento 會 compile 讓他自己的程式碼，compile 之後，他會依照每個 Model 各自產生一個名為 `<class-type>Factory` 的物件。用這 Factory Object 可以快速存取資料。

> Factory 是程式自己產生的



##### 範例如下：
你可以看到我們用 Dependency Injection 將 `EmployeeFactory` 和 `EmployeeFactory` 置入到 constructor。

    namespace Foggyline\Office\Controller\Test;
    class Crud extends \Foggyline\Office\Controller\Test
    {
        protected $employeeFactory;
        protected $departmentFactory;
        public function __construct(
            \Magento\Framework\App\Action\Context $context,
            \Foggyline\Office\Model\EmployeeFactory $employeeFactory,
            \Foggyline\Office\Model\EmployeeFactory $departmentFactory
        )
        {
            $this->employeeFactory = $employeeFactory;
            $this->departmentFactory = $departmentFactory;
            return parent::__construct($context);
        }
    } 



然後就可以使用了

##### creating new entities
    $employee1 = $this->employeeFactory->create();
    $employee1->setDepartment_id($department1->getId());
    $employee1->setEmail('goran@mail.loc');
    $employee1->setSalary(3800.00);
    $employee1->setVatNumber('GB123451234');
    $employee1->setNote('Note #1');


##### reading based on Id
    $employee = $this->employeeFactory->create();
    $employee->load(25);

##### Update based on Id
    $department = $this->departmentFactory->create();
    $department->load(28);
    $department->setName('Finance #2');
    $department->save();

##### Delete based on Id
    $employee = $this->employeeFactory->create();
    $employee->load(25);
    $employee->delete();

#### Collection：用 Factory 取得 Collection
Collection 的好處是他有 Caching, 有 Select filter 等能力。後續章節會特別提到。

    $collection = $this->employeeFactory->create()
                        ->getCollection();

    foreach ($collection as $employee) {
        \Zend_Debug::dump($employee->toArray(), '$employee');
    } 

    // 另一種比較直觀的做法
    foreach($collection as $employee){
		var_dump($employee->getData());
    }


##### 再多一個範例
這是 Magento 自己的 CMS Block Fatory

    function __construct ( \Magento\Cms\Model\BlockFactory $blockFactory) {
        $this->blockFactory = $blockFactory;
    }

也就是說 `Magento\Cms\Model\BlockFactory` 是個  Factory 類別，而且是由 `class Magento\Cms\Model\Block` 產生出來的。

### EVA 是什麼？

先不用管他，不然腦袋會爆炸，前面學得都會忘記或搞混。
之後會提到。現在只要知道它是管多國語言的就好。

