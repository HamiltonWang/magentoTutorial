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

Controller 長這樣

    http://example.com/route_name/controller/action

請注意 `route_name` 是放在`routes.xml`特別的唯一名稱。
`controller` is the folder inside Controller folder.
action is a class with execute method to process request.

### 4-1. define router by adding routes.xml
file: app/code/Aiart/Hotel/etc/frontend/routes.xml

XML9 lines


    <?xml version="1.0" ?>
    <config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
        <router id="standard">
            <route frontName="hotel" id="hotel">
                <module name="Aiart_Hotel"/>
            </route>
        </router>
    </config>


the tag ‘s frontName attribute is the first part of the url
the router is connected to a module here, so it is how controller and module is linked.

In Magento 2 URL’s are constructed this way:

    <frontName>/<controler_folder_name>/<controller_class_name>

but the default is index, namely
    
    hotel/index/index

but the following example i will use

    hotel/index/search 
 to illustrate the concept clearer.

### 4-2. create controller file e.g. Index.php
location: app/code/Aiart/Hotel/Controller/Index/Search.php

PHP20 lines

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

every action has its own class which implements the execute() method.

    php bin/magento cache:clean

After finish all steps, the output Hello World should be displayed in your browser when you open the URL.

    http://example.com/hotel/index/search

### 5. Creating a block
Create a Index.php file in the app/code/Aiart/Hotel/Block folder with the following code:

PHP11 lines


    <?php
    namespace Aiart\Hotel\Block;
    class Index extends \Magento\Framework\View\Element\Template
    {
        public function getHelloWorldTxt()
        { 
            return 'Hello world!';
        }
    }


### 6. Create layout file

    app/code/Aiart/Hotel/view/frontend/layout/hotel_index_index.xml

XML7 lines


    <?xml version="1.0"?>
    <page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" layout="1column" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
        <referenceContainer name="content">
            <block class="Aiart\Hotel\Block\Index" name="aiart_index_index" template="Aiart_Hotel::index.phtml" />
        </referenceContainer>
    </page>


##Layout file
### 7. Create template file

File: app/code/Aiart/Hotel/view/frontend/templates/index.phtml

HTML5 lines

    <p>
    <h1><?php echo $this->getHelloWorldTxt(); ?></h1>
    <h2>Welcome to my site</h2>
    </p>

## 8. Flush Magento cache

    php bin/magento cache:clean

    php bin/magento cache:flush

## 9. check your result

http://www.example.com/hotel/index/search

if at step 4-2 yo use your class name as Index then your url would be http://example.com/hotel/index/index
in short it can be found at
http://example.com/hotel/



so it is done!

another quick way is to use a module generator
http://www.silksoftware.com/magento-module-creator/magento2x.php

you can use it when you understand the concept and try it by yourself manually and then you can use the generator. Enjoy!
