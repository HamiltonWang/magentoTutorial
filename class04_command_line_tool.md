Why use command line?

Because it is the shortest path, the nut and bolts of a software.

Using JSON and other API is cool, but in the end, if you want to build a solid software. It is better to use command line as foundation. You can unit test them, setup cron jobs, provide access for other programming languages. There is no need to major framework to do unit test or cross-language binding if everything is done internally within organization or team.

Let’s get started.

Step 1: Define registration.php and module.php
File: app/code/Aiart/Hotel/registration.php

PHP11 lines

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
File: app/code/Aiart/Hotel/etc/module.xml

XML5 lines

<?xml version="1.0"?>
<config 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="Aiart_Hotel" setup_version="1.0.0" />
</config>
Step 2: Define command in di.xml
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
