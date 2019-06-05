# Magento 教學 04：實際範例

## Create a Magento Module Command to Import Products

source from: https://www.thirdandgrove.com/create-magento-module-command-import-products

Technology Development  Jul 08 '16

I have been working with Magento 2 a lot recently and devised a way to import products into the system using the command line. I will be using TAG as the overall folder to hold this custom module. If you wish to use some other name, just keep in mind you will need to update the references accordingly. I would also like to mention that I’m using PHP 7, so there may be some difference if you are using an older version. Finally, I’m going to be throwing a ton of code at you at the beginning, but will be explaining the important pieces towards the end of this post.

First things first, in `magento/app/code` I have created a folder called `TAG`. This will contain our custom module to import products. I am calling this module ProductImport, so I have created a directory for it in `TAG/ProductImport`.

We need to register the module with Magento. Place the following code in `TAG/ProductImport/registration.php`:

    <?php
        \Magento\Framework\Component\ComponentRegistrar::register(
        \Magento\Framework\Component\ComponentRegistrar::MODULE,
        'TAG_ProductImport',
        __DIR__
    );


Next, we need to create the `module.xml` and `di.xml` files.

In `TAG/ProductImport/etc/module.xml` place the following code:

    <?xml version="1.0"?>
        <config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../../../../../lib/internal/Magento/Framework/Module/etc/module.xsd">
            <module name="TAG_ProductImport" setup_version="1.0.0" />
        </config>

Then, in `TAG/ProductImport/etc/di.xml` place the following: 

    <config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../../../../../lib/internal/Magento/Framework/ObjectManager/etc/config.xsd">
        <type name="Magento\Framework\Console\CommandList">
            <arguments>
                <argument name="commands" xsi:type="array">
                    <item name="tag_product_import" xsi:type="object">TAG\ProductImport\Commands\ProductImportCommand</item>
                </argument>
            </arguments>
        </type>
    </config>

The most important part of that last snippet is <item name="tag_product_import" xsi:type="object">TAG\ProductImport\Commands\ProductImportCommand</item>. This is telling Magento that there is a command line option that is stored in TAG\ProductImport\Commands\ProductImportCommand.php, so let’s go ahead and place the following code there:


    <?php
        namespace TAG\ProductImport\Commands;
        use Symfony\Component\Console\Input\{InputInterface, InputArgument, InputOption};
        use Symfony\Component\Console\Output\OutputInterface;
        use Symfony\Component\Console\Command\Command;
        use Magento\Catalog\Api\ProductRepositoryInterface;
        use Magento\Catalog\Model\Product\Attribute\Source\Status;
        use Magento\Framework\App\{ObjectManager, State};
        use TAG\ProductImporter\Setup\InstallData;
        /**
        * ProductImporterCommand
        *
        * Exposed Magento command option that allows the caller to import products.
        */
        class ProductImportCommand extends Command
        {
            /**
            * Constructor
            *
            * @param State $state A Magento app State instance
            *
            * @return void
            */
            public function __construct(State $state, ProductRepositoryInterface $prepo)
            {
                // We cannot use core functions (like saving a product) unless the area
                // code is explicitly set.
                try {
                    $state->setAreaCode('adminhtml');
                    } catch (\Magento\Framework\Exception\LocalizedException $e) {
                // Intentionally left empty.
                }
                parent::__construct();
            }

            /**
            * Configures arguments and display options for this command.
            *
            * @return void
            */
            protected function configure()
            {
                $this->setName('tag:productimport');
                $this->setDescription('Import products into Magento');
                $this->addArgument('import_path', InputArgument::REQUIRED, 'Path to the import file (relative to Magento bin directory).');
                parent::configure();
            }

            /**
            * Execute the command to add product data into the database.
            *
            * @param InputInterface $input An input instance
            * @param OutputInterface $output An output instance
            *
            * @return void
            */
            protected function execute(InputInterface $input, OutputInterface $output)
            {
                $import_path = $input->getArgument('import_path');
                $output->writeln("<info>Loading SKU information from $import_path</info>");
                $product_data = require $import_path;

                if (empty($product_data)) {
                   $output->writeln('<error>No SKU information found.</error>');
                   return;
                }
                $output->writeln('<info>Importing Products...</info>');
                $objectManager = ObjectManager::getInstance();
                foreach ($product_data as $sku_data) {
                $product = $objectManager->create('\Magento\Catalog\Model\Product');
                $product->setSku($sku_data['sku']);
                $product->setName($sku_data['name']);
                $product->setPrice($sku_data['price']);
                $product->setTypeId($sku_data['type']);
                $product->setWebsiteIds([1]);
                $product->setStatus(Status::STATUS_ENABLED);
                $product->setAttributeSetId($product->getDefaultAttributeSetId());
                try {
                    $product->save();
                    $output->writeln('Created product = <comment>' . $sku_data['name'] . '</comment>');
                } catch (Exception $e) {
                    $output->writeln('<error>Unable to create product = ' . $sku_data['name'] . '</error>');
                }
            }
            $output->writeln('<info>Products have been imported!</info>');
        }
    }


First, let’s talk about the constructor. There isn’t a whole lot there except for `$state->setAreaCode`('adminhtml’). However, this code is important for a couple of reasons. First, without setting the area code, Magento will not allow you to use some core methods (including saving a product). You will also notice that it is in a try block. This is because this code can be invoked in multiple ways and the area code may already be set. If this happens, you get a nice looking bunch of red text all over your screen.

The `configure()` function is pretty simple. When you run the `php bin/magento` list, you will see a list of all valid Magento commands and a short description. The configure function is what populates. This code also specifies that we have an argument that MUST be passed in. So, to invoke our command, we would use php bin/magento tag:product import ../some/path/products.php (more on this later).

The execute() method is also fairly simple. We grab the argument that we specified and begin to use it. In our case, we are just calling loading the php from the passed in file path. If there is no data in there, then we show an error message and exit early. On the other hand, if we do find data, then we integrate over the list and begin to create products.

To create a product, we use the `ObjectManager` to create a product instance. We set the name, sku, price, and product type from the data file that was loaded in. We also set status of the product, as well as the Magento websites that need to use the product. Lastly, we save the product and show a message depending on whether the save was a success or a failure.

So now that we have built out this custom module, let’s see it in action!

Create a file called products.php in some directory that you will be able to find again later. In this file, place the following code:

    <?php
        return array(
        array(
        'sku' => '01234',
        'name' => 'Cool Coffee Mug',
        'price' => '9.99',
        'type' => 'simple',
        ),
        array(
        'sku' => '56789',
        'name' => 'DVD Rewinder',
        'price' => '74.99',
        'type' => 'simple',
        ),
    );


This is a small example that only uses two skus, but you can put in as many as you like. 

We run this calling `php bin/magento tag:product import ../some/path/products.php` and we should get 

some output:

Output of adding products to Magento

That’s it! You have successfully added some products to Magento. With a little bit more work you can extend the module to be able to import custom product fields and even upload the image thumbnails for each product. Good luck and happy Magento-ing!

