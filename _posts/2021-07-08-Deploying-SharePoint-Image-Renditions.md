---
published: false
---
## Deploying SharePoint Image Rendtions

SharePoint Image Rendtions are a usefull tool to save bandwidth and reduce image loading times. They allow you to specify the dimensions of an image, and SharePoint will then serve the image with those dimensions. 

To use image renditions you will have to have blob caching enabled on your web app. 

Deploying image renditions via code can be challenging. in my case, I wanted to deploy image renditions declaratively; and if an image rendition already existed, I wanted to override it. 

Initially I tried to use a PNP Provisioning Template. The provisioning template schema for image renditions is not that clear. After succesfully defining the image renditions according to the schema and testing, it became clear that the PNP provisiong template would be additive for the image renditions. i.e. it would create a new image rendition with the same name rather than overriding an existing image rendition. So this did not fit my use case. 

Second, I used the Files element of the PNP provisiong template schema to reference a file to upload to the master pages gallery (image renditions are located there in the PublishingImageRenditions.xml file). Editing this file and then uploading it via pnp provisioning templates worked well. The image renditions created/edited would be exactly as defined in the file. However, when I ran apply-pnpprovisioningtemplate an error would be thrown, despite the outcome seeming to be what was intended. This was confusing but I did not have time to investigate. Something for another day. 

I pursued an outcome without errors, and so I ended up writing a powershell script to achieve what I wanted. The script would check the image renditions, and if an image rendition already existed it would update it, otherwise it would create that image rendition. Example of script below:


Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
