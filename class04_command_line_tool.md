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


## Step 2: Define command in di.xml
In the di.xml file, you have to use a type Magento\Framework\Console\CommandList to tell the program that you want to use command line.

File: app/code/Aiart/Hotel/etc/di.xml

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
This is to assign  InsertCategory command class. This class requires a execute  function and we will write code in it.

 

Step 3: Create the implementation class
The actual implementation is carried out in the following file as we define previously in di.xml.

File: app/code/Aiart/Hotel/Console/CreateCategory.php

PHP22 lines

<?php
namespace Aiart\Hotel\Console;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
class CreateCategory extends Command
{
   protected function configure()
   {
       $this->setName('Category:create');
       $this->setDescription('Create Category');
       
       parent::configure();
   }
   
   protected function execute(InputInterface $input, OutputInterface $output)
   {
       $output->writeln("Testing..1,2,3, ready to launch...");
   }
}
The main 2 functions are execute() and configure(). First is to write your implementation code and the later is to setup the command line instruction such as name, description and arguments.

Let’s see if you have done correctly.

Please flush your cache

PowerShell1 lines

php ./bin/magento setup:upgrade
Let’s see by typing the following command

Batch File1 lines

php ./bin/magento --list
check that the module is active bin/magento module:status if not, it means you haven’t enable your module. Run php ./bin/magento module:enable Aiart_Hotel.



Step 3: Add in arguments
