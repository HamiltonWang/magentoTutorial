## 1. 把 Magento cache 解除
到 Admin → System → Cache Management → 選擇 cache types 然後把它為解除

Configuration
Layouts
Collections Data
你可以把所有的選項都點掉，但是這樣網頁會變得太慢。


## 2. 把 Magento 設定成開發模式（developer mode）
請執行以下指令

    php bin/magento deploy:mode:set developer


## 3. 新增模組的必要檔案和檔案夾
### 3-1. 新增檔案夾

    app/code/Aiart
    app/code/Aiart/Hotel



第一個是 namespace 然後第2個是 module 的檔案夾
一個 namespace 裡面可以有多個 module, 一般來說，namespae 是你的公司名稱。



### 3-2. 建立 module.xml 檔案
檔案位置：

    app/code/Aiart/Hotel/etc

    etc/module.xml

範例內容：


    <?xml version="1.0"?>
    <config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
        <module name="Aiart_Hotel" setup_version="1.0.0" />
    </config>

### 3-3. 加入 registration.php 檔案
檔案位置: app/code/Aiart/Hotel/registration.php

這是為了註冊的用途，讓你的 Magento 知道你有這個模組

內容範例：

    <?php \Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'Aiart_Hotel',
    __DIR__
    );


### 3-4. 安裝你的模組 module
install your module by issuing command

    php bin/magento module:status
    php bin/magento module:enable Aiart_Hotel
    php bin/magento setup:upgrade

如果你要確認模組已經被安裝，請到

    Admin → Stores → Configuration → Advanced → Advanced
   
來看看是否有在這顯示出來。

在 Magento 所有商業邏輯應該寫在 module 裡面。所以不同的 module 應該有不同的功能，然後交叉呼叫。


## 如何寫個 controller

Controller 的 url 一般是這樣的形式

    http://example.com/route_name/controller/action

請注意 `route_name` 是放在`routes.xml`特別的唯一名稱。
`controller` 必須和 controller 裡面的檔案夾名稱相同，待會會有範例來解釋。
`action` 是個`類別 class` 裡的 `execute` method 

接下來為範例


### 4-1. 定義你 routes.xml
檔案: app/code/Aiart/Hotel/etc/frontend/routes.xml

內容範例：

    <?xml version="1.0" ?>
    <config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
        <router id="standard">
            <route frontName="hotel" id="hotel">
                <module name="Aiart_Hotel"/>
            </route>
        </router>
    </config>

請注意 `frontName` 就是 url 中的第一部份

`router` 的標籤 就是在模組中中 `controller` 是如何跟模組相關聯的。

舉例來說：

在 Magento 2 的 URL’s 是這麼表示的:

    <frontName>/<controler_folder_name>/<controller_class_name>

預設是`index`, 也就是如下
    
    hotel/index/index


但為了要清楚一點，我們使用以下 URL 來當解說範例

    hotel/index/search 
    


### 4-2. 新增一個 controller 檔案 e.g. Search.php
檔案位置：app/code/Aiart/Hotel/Controller/Index/Search.php

內容範例：

    <?php namespace Aiart\Hotel\Controller\Index; 
    use Magento\Framework\App\Action\Context; 
    use Magento\Framework\View\Result\PageFactory;
    
    class Search extends \Magento\Framework\App\Action\Action { 
        protected $_resultPageFactory; 
        public function __construct(Context $context, PageFactory $resultPageFactory) { 
            $this->_resultPageFactory = $resultPageFactory;
            parent::__construct($context);
        }
    
        public function execute()
        {
            $resultPage = $this->_resultPageFactory->create();
            return $resultPage;
        }
    }

每個 `action` 在自己的類別中都會有一個 `execute()` 的方法。
注意一下我之前說的
檔案夾位置的 `Controller/Index/Search.php` 的 `Index` 檔案夾的 `Search.php` 就是 Search 類別，然後 `execute`就是預設的執行方法。


接下來把暫存清除

    php bin/magento cache:clean


接下來你應該在以下網址可以看到你執行的結果

    http://example.com/hotel/index/search
    
    
    

### 5. 新增一個 Block
什麼是 `Blcok`?
他是網頁的處理邏輯，你可以用來計算資料或是把一些資料從資料庫拉出來後顯示。

請在 `app/code/Aiart/Hotel/Block` 檔案夾中加入 `Index.php` 然後使用以下範例。


    <?php
    namespace Aiart\Hotel\Block;
    class Index extends \Magento\Framework\View\Element\Template
    {
        public function getHelloWorldTxt()
        { 
            return 'Hello world!';
        }
    }


### 6. 加入 Layout 檔案
這是告訴 Magento 如何將 block 和 template 連結使用。
URL 格式 {module_root}/view/{area}/layout/


    app/code/Aiart/Hotel/view/frontend/layout/hotel_index_index.xml

範例


    <?xml version="1.0"?>
    <page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" layout="1column" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
        <referenceContainer name="content">
            <block class="Aiart\Hotel\Block\Index" name="aiart_index_index" template="Aiart_Hotel::index.phtml" />
        </referenceContainer>
    </page>


##Layout 檔案

### 7. 新增一個 template 檔案
這是告訴 Magento 如何排版

檔案位置: app/code/Aiart/Hotel/view/frontend/templates/index.phtml

因為上面有在 `layout` 檔案將 `template` 和 `block` 關聯在一起，所以在 phtml  中可以直接使用 `block` 的 function  

    <p>
    <h1><?php echo $this->getHelloWorldTxt(); ?></h1>
    <h2>Welcome to my site</h2>
    </p>

## 8. 清除暫存

    php bin/magento cache:clean

    php bin/magento cache:flush

## 9. 檢查你的結果

        http://www.example.com/hotel/index/search

if at step 4-2 yo use your class name as Index then your url would be http://example.com/hotel/index/index
in short it can be found at
http://example.com/hotel/



so it is done!

another quick way is to use a module generator
http://www.silksoftware.com/magento-module-creator/magento2x.php

you can use it when you understand the concept and try it by yourself manually and then you can use the generator. Enjoy!
