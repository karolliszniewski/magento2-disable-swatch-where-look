# magento2-disable-swatch-where-look

Ah, now I better understand what you're looking for. In Magento 2, when creating a custom module with image upload functionality, the image types are typically defined in several places. Let's look at where to check and modify these definitions:

InstallData/UpgradeData Script
First, check your module's Setup scripts where attributes are defined:

phpCopy// In Setup/InstallData.php or Setup/UpgradeData.php
use Magento\Catalog\Model\Product;

public function install(ModuleDataSetupInterface $setup, ModuleContextInterface $context)
{
    $eavSetup = $this->eavSetupFactory->create(['setup' => $setup]);
    
    // This is where image attribute types are defined
    $eavSetup->addAttribute(
        Product::ENTITY,
        'your_image_attribute',
        [
            'type' => 'varchar',
            'label' => 'Your Image',
            'input' => 'image',  // or 'media_image'
            'frontend' => 'Magento\Catalog\Model\Product\Attribute\Frontend\Image',
            'required' => false,
            'sort_order' => 50,
            'global' => \Magento\Eav\Model\Entity\Attribute\ScopedAttributeInterface::SCOPE_STORE,
            'used_in_product_listing' => true,
            'user_defined' => true,
            'visible' => true,
            'is_used_in_grid' => true,
            'is_visible_in_grid' => true,
            'is_filterable_in_grid' => true,
            // Remove or modify if this was set as swatch
            'is_used_for_swatch' => false,  // This makes it a swatch image
        ]
    );
}

di.xml Configuration
Check your module's etc/di.xml for image type definitions:

xmlCopy<!-- app/code/YourVendor/YourModule/etc/di.xml -->
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <virtualType name="YourCustomImageType" type="Magento\Catalog\Model\Product\Image">
        <arguments>
            <argument name="width" xsi:type="number">100</argument>
            <argument name="height" xsi:type="number">100</argument>
        </arguments>
    </virtualType>
</config>

view.xml Configuration
Check your module's view.xml where image types are configured:

xmlCopy<!-- app/code/YourVendor/YourModule/etc/view.xml -->
<view xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Config/etc/view.xsd">
    <media>
        <images module="Magento_Catalog">
            <image id="your_custom_image_id" type="your_image_attribute">
                <width>100</width>
                <height>100</height>
            </image>
        </images>
    </media>
</view>

Form UI Component
Check your UI component form configuration:

xmlCopy<!-- app/code/YourVendor/YourModule/view/adminhtml/ui_component/your_form.xml -->
<field name="your_image_attribute">
    <argument name="data" xsi:type="array">
        <item name="config" xsi:type="array">
            <item name="dataType" xsi:type="string">string</item>
            <item name="source" xsi:type="string">product</item>
            <item name="label" xsi:type="string" translate="true">Your Image</item>
            <item name="visible" xsi:type="boolean">true</item>
            <item name="formElement" xsi:type="string">fileUploader</item>
            <item name="elementTmpl" xsi:type="string">ui/form/element/uploader/uploader</item>
            <item name="previewTmpl" xsi:type="string">Magento_Catalog/image-preview</item>
            <item name="required" xsi:type="boolean">false</item>
            <item name="uploaderConfig" xsi:type="array">
                <item name="url" xsi:type="url" path="your_route/upload/image"/>
            </item>
        </item>
    </argument>
</field>

Database Tables
Check the catalog_product_entity_media_gallery and catalog_product_entity_media_gallery_value tables in your database:

sqlCopySELECT * FROM catalog_product_entity_media_gallery_value_to_entity WHERE attribute_id = (
    SELECT attribute_id 
    FROM eav_attribute 
    WHERE attribute_code = 'swatch_image'
);
To fix the issue:

First, identify where the swatch_image type was set:

phpCopy// In your module's setup scripts
$eavSetup->updateAttribute(
    Product::ENTITY,
    'your_image_attribute',
    'is_used_for_swatch',
    false  // Set to false to remove swatch functionality
);

If you want to change the image type completely:

phpCopy// In your module's setup scripts
$eavSetup->updateAttribute(
    Product::ENTITY,
    'your_image_attribute',
    [
        'frontend_input' => 'media_image',
        'input' => 'image',
        'frontend' => 'Magento\Catalog\Model\Product\Attribute\Frontend\Image',
        'is_used_for_swatch' => false
    ]
);
To implement this fix:

Create a new setup script (UpgradeData.php) if you don't already have one
Increment your module version in module.xml
Run setup upgrade commands:

bashCopyphp bin/magento setup:upgrade
php bin/magento setup:di:compile
php bin/magento cache:clean
This should help you identify where the swatch_image type was defined and how to modify or remove it. Let me know if you need any clarification or have questions about implementing these changes.
