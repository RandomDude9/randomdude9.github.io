---
published: false
---
## Deploying SharePoint Image Rendtions

SharePoint Image Renditions are a useful tool to save bandwidth and reduce image loading times. They allow you to specify the dimensions of an image - SharePoint will then serve the image with those dimensions. 

To use image renditions you will have to have blob caching enabled on your web app. There are plenty of resources online on how to do that so I won't cover it here. This post will focus purely on methods for deploying your Image Renditions

Deploying image renditions via code can be challenging. In my case, I wanted to deploy image renditions declaratively; and if an image rendition already existed, I wanted to override it. 

Initially I tried to use a PNP Provisioning Template. The provisioning template schema for image renditions is not very clear. After succesfully defining the image renditions according to the schema and testing, it became clear that the PNP provisiong template would be additive for the image renditions. i.e. it would create a new Image Rendition with the same name rather than overriding an existing image rendition. So this did not fit my use case. 

Second, I used the Files element of the PNP provisiong template schema to reference a file to upload to the Master Page Gallery (image renditions are located there in the PublishingImageRenditions.xml file). Editing this file and then uploading it via pnp provisioning templates worked well. The image renditions created/edited would be exactly as defined in the file. However, when I ran the Apply-PnPProvisioningTemplate cmdlet an error would be thrown, despite the outcome seeming to be what was intended (I should have captured a screenshot for later reference). This was confusing but I did not have time to investigate and resolve. Something for another day. I needed full confidence in the deployment, so even though it seemed to be working I looked for another solution. 

An example of the xml is below:

```    
<ImageRendition>
      <Height>75</Height>
      <Id>5</Id>
      <Name>browse</Name>
      <Version>4</Version>
      <Width>-1</Width>
  </ImageRendition>
    ```

I pursued an outcome without errors, and so I ended up writing a powershell script to achieve what I wanted. The script would check the image renditions, and if an image rendition already existed it would update it, otherwise it would create that image rendition. Example of script below:


Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
