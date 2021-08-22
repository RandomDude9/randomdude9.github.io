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

I pursued an outcome without errors, and so I ended up writing a powershell script to achieve what I wanted. The script would check the image renditions, and if an image rendition already existed it would update it, otherwise it would create that image rendition. Example of script below to be run from the CA server:

```
#checks if an image rendition exists with the given name. If it exists it updates the image rendition with the $height. If it does not exist it will create a new image rendtion with the $height
function Add-ImageRendition {
  param (
    $ImageRenditionName,
    $Height # I only use a height parameter, leaving the width blank to maintain the image's orgiginal aspect ratio
  )
  
  add-pssnapin microsoft.sharepoint.powershell #move these setup lines outside of the function
  $site = Get-SPSite $SiteUrl
  $imageRenditions =  [Microsoft.SharePoint.Publishing.SiteImageRenditions]::GetRenditions($site)

  $existingRendition = $imageRenditions | where-Object {$_.Name -eq $ImageRenditionName};

  if($existingRendition -is [array]){
    Write-Host "Add-ImageRendition found multiple renditions with the name $ImageRenditionName. Existing..."
    #put some logic here if there are multiple image renditions with the same name. I knew this wouldn't happen in my situation
    return
  }

  if($existingRendition -eq $null){ #create the new image rendition if it doesn't already exist
    $rendition = New-Object Microsoft.SharePoint.Publishing.ImageRendition
    $rendition.Name = $ImageRenditionName
    $rendition.Height = $Height
    $imageRenditions.Add($rendition)
  }else { #change the image renditions height property if it already exists
    $existingRendition.Height = $Height
  }  
  
  $imageRenditions.Update() #update the properties
}

#example calling the function
Add-ImageRendition -ImageRenditionName "Sample" -Height 75
```

