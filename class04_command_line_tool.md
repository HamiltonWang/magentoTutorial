# Class 04: 使用/建構 Command line

你應該會覺得有點奇怪，為何這個章節會教導使用 command line (後續簡稱 cmd)?

沒錯，M2 的確有很多重要的功能。但是為了實際+學習，你可能剛好為了個專案在製作所以來學這個，然而你如果選了這個學習課程，表示你是個後端工程師。因為我的開端就是從後端程式開始教導，而不是前端（當然也是因為前端的課程也太多了）。

當你努力建立了個資料庫（不是用 M2 setup script），或是已經有了個現有的資料庫，或是你手動直接去資料庫建立了個 table, 那你勢必要倒資料進去。這時你最需要的是從某處抓取資料然後匯入，這時其實建個 UI 也不是不行，但是可能很花時間，所以用 cmd  是個簡單的方式也是許多工程師採用的方式。而且，這樣也方便做 Unit Test。

> It is the shortest path, the nut and bolts of a software.

讓我們開始吧。 Let’s get started.





## Step 1: 設定 registration.php 和 module.php

#### registration.php
檔案位置: app/code/Aiart/Hotel/registration.php

    <?php
    /**
    * Copyright © AI Art Inc.. All rights reserved.
    * See COPYING.txt for license details.
    */
    \Magento\Framework\Component\ComponentRegistrar::register(
        \Magento\Framework\Component\ComponentRegistrar::MODULE,
        'Aiart_Hotel',
        __DIR__
    );

#### Module.php
檔案位置: app/code/Aiart/Hotel/etc/module.xml

    <?xml version="1.0"?>
    <config 
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
        <module name="Aiart_Hotel" setup_version="1.0.0" />
    </config>



## Step 2: 在 di.xml 定義 command

在 di.xml 檔案中你必須使用 `type` `Magento\Framework\Console\CommandList` 來讓程式知道你要用 cmd.

檔案位置: app/code/Aiart/Hotel/etc/di.xml

JavaScript10 lines

    <?xml version="1.0"?>
    <config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <type name="Magento\Framework\Console\CommandList">
        <arguments>
            <argument name="commands" xsi:type="array">
                <item name="createCategory" xsi:type="object">Aiart\Hotel\Console\CreateCategory</item>
            </argument>
        </arguments>
    </type>
    </config>

* 請注意 `<item>` 是用來宣告 cmd 的名稱和他對應的執行檔案 e.g. `Aiart\Hotel\Console\CreateCategory`。

 

## Step 3: 製作你的實際要執行的程式 Create the implementation class

在步驟 2 的 di.xml 中 `item` 裡的 `CreateCategry` 實際執行程式的檔案位置是 `app/code/Aiart/Hotel/Console/CreateCategory.php`。

他是放在 `Console` 檔案夾中。


    <?php
        namespace Aiart\Hotel\Console;
        use Symfony\Component\Console\Command\Command;
        use Symfony\Component\Console\Input\InputInterface;
        use Symfony\Component\Console\Output\OutputInterface;
        class CreateCategory extends Command
        {
            protected function configure()
            {
                $this->setName('aiart:category:create');
                $this->setDescription('Create Category');
                
                parent::configure();
            }
            
            protected function execute(InputInterface $input, OutputInterface $output)
            {
                $output->writeln("Testing..1,2,3, ready to launch...");
            }
        }

主要的程式函式是 `execute()` 和 `configure()`。
`execute()` 是執行程式， `configure()` 是宣告他的名稱、敘述、參數等。

> `$this->setName('aiart:category:create');` 是到時執行的指令。

### 實際測試
#### 3-1 清掉你的 Cache

M2 版本 2.1
    sudo rm -rf ./var/cache/* ./var/page_cache/* ./var/di/* ./var/generation/* pub/static/* var/view_preprocessed/*
    sudo php ./bin/magento cache:clean
    sudo php ./bin/magento cache:flush

M2 版本 2.2 後
    sudo rm -rf ./var/cache/* ./var/page_cache/* ./var/di/* ./generated/code/*  ./generated/metadata/* ./var/session/*
    sudo rm -rf pub/static/* var/view_preprocessed/*
    sudo php ./bin/magento cache:clean && sudo php ./bin/magento cache:flush

#### 3-2 安裝

    php ./bin/magento setup:upgrade
    php ./bin/magento setup:di:compile



把模組列出

    php ./bin/magento --list



#### 3-3 看是否有安裝成功

檢查一次是否模組是啟動的

> 如果你還沒建構模組，請參考前一章節，因為你建構的任何東西都必須依附在某個自建模組下

    bin/magento module:status
    
如果沒有啟動，請啟動他

    php ./bin/magento module:enable Aiart_Hotel.

#### 3-4 執行

    php bin/magento aiart:category:create

## Step 4: 加入參數

我們需要多點功能
檔案位置：`app/code/Aiart/Hotel/Console/CreateCategory.php`


    <?php
        namespace Aiart\Hotel\Console;
        use Symfony\Component\Console\Command\Command;
        use Symfony\Component\Console\Input\InputInterface;
        use Symfony\Component\Console\Output\OutputInterface;
        class CreateCategory extends Command
        {
            protected function configure()
            {
                $this->setName('aiart:multiply');
                $this->setDescription('Multiply your number');
                // Next line will add a new required parameter to our script
                $this->addArgument('number', InputArgument::REQUIRED, __('Type a number'));
 
                parent::configure();
            }
            
            protected function execute(InputInterface $input, OutputInterface $output)
            {
                $output->writeln("請輸入一個數字:");
                $output->writeln($input->getArgument('number') * 2);
            }
        }

這只是把你輸入的數字乘 2 而已。

#### 4-1 執行看看

    php bin/magento aiart:multiply

他會出現 “Not enough arguments” 的錯誤訊息

但你如果有加入參數

    php bin/magento aiart:multiply 2

他就會成功執行

    請輸入一個數字:
    4


## 進階參考範例

這是個很棒的實際範例
https://www.thirdandgrove.com/create-magento-module-command-import-products

為了怕以後網頁不見來保留資料，我把程式碼保留在 class04_real_example.md


