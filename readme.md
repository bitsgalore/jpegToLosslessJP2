# Quality loss after compressing JPEG image to lossless JP2

## Summary

Compressing a JPEG image to *lossless* JP2 results in some loss of quality in both ImageMagick and the Aware SDK.


Software versions used: ImageMagick 6.8.9-8 Q16 x64 2014-08-26; Aware v. 3.19.0.0. Tested under Windows 7.

## Example

Starting with image *balloon.jpg*, compress this JPEG to lossless JP2 with Imagemagick:

    convert balloon.jpg balloon_im.jp2

Use [jpylyzer](http://jpylyzer.openpreservation.org/) to double check that the resulting image really is lossless:

    jpylyzer balloon_im.jp2 > balloon_im.xml

Value of *transformation* equals *5-3 reversible*, so that looks as expected.

Now compare both images (pixel values) with ImageMagick's *compare* tool, using:

    compare -metric PSNR balloon.jpg balloon_im.jp2 NUL
  
Result:

     90.0235

Even though this is pretty good, for a truly lossless compression (pixel values are 100% identical) the PSNR should approach infinity (indicated by `1.#INF` in ImageMagick).

Interestingly, if I first convert the JPEG to a TIFF, and then compress the TIFF to JP2 the results are as expected. So:

    convert balloon.jpg balloon_im.tiff
    convert balloon_im.tiff balloon_im_from_tiff.jp2

And then:

    compare -metric PSNR balloon.jpg balloon_im.tiff balloon_im_from_tiff.jp2 NUL

Result:

     1.#INF

So the JP2 is identical to the source JPEG. This makes it all the more puzzling why the result is different when converting from JPEG directly. I repeated the above steps for the Aware JPEG 2000 SDK, which gave me similar results (although the quality loss for the direct conversion was even greater). See below table:

|Encoder|PSNR (direct conversion)|PSNR(conversion via TIFF)|
|:--|:--|:--|
|ImageMagick|90.0235|1.#INF|
|Aware|62.9218|1.#INF|

The only explanation I can come up with is that both ImageMagick and Aware do something wrong while decoding the JPEG-compressed image data. I wasn't able to test this for any other encoders (OpenJPEG and Kakadu don't support JPEG as a source format, and Adobe Photoshop stopped working on my PC altogether).   

## Files

|Name|Description|
|:--|:--|
|balloon.jpg|Source JPEG|
|balloon_im.jp2|JP2, created directly from *balloon.jpg* with ImageMagick|
|balloon_im.tiff|TIFF, created from *balloon.jpg* with ImageMagick|
|balloon_im_from_tiff.jp2|JP2, created from *balloon_im.tiff* with ImageMagick|
|balloon_aware.jp2|JP2, created directly from *balloon.jpg* with Aware|
|balloon_aware_from_tiff.jp2|JP2, created from *balloon_im.tiff* with Aware|
|outputJpylyzer.xml|Jpylyzer output for all JP2s|
