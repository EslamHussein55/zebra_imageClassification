<p align="center"><img width=30% src="https://github.com/darabigdata/IDWBotswana/blob/master/media/00000022.jpg"></p>


# Challenge: Google Image Web-scraping and Classification

In this challenge you will learn how to web-scrape images from Google and use them to train/test a Machine Leaning (ML) model. The aim is to come up with a image classification problem (cats vs dogs, people vs trees, Trump vs an orange cheeto etc), web-scrape the images and then use ML for the classification.

### What's in the repo?

* **Classify_Zebra_Images.ipynb**
    * *A jupyter notebook that implements simple machine learning to identify images of zebras*
* **urls.txt**
    * *A list of urls of pictures of zebras from Google Image Search*
* **download_images.py**
    * *Code to download images from a list of urls*
* **images/zebra**
    * *A directory of ~400 pictures that contain zebras*
* **images/others**
    * *A directory of ~400 pictures that don't contain zebras*



------

## Simple Web-scraping and Classification Tutorial

This tutorial is devided into three parts:
1. Image Webscraping: Obtaining a text file of the image urls that will make up the training and test datasets
2. Download all the images: Using python to download and perfom simple checks on the images
3. Image classification: Use Scikit learn to perform image classification



### Step 1: Image Webscraping

For this challenge you're going to need to build your own library of training data. For image data one of the best places to get this kind of data is Google Images. So in this tutorial we'll use Google Images to web-scrape a database of images. The instructions for this step are a simplified version of the excellent blog post [here](https://www.pyimagesearch.com/2017/12/04/how-to-create-a-deep-learning-dataset-using-google-images/). To follow this tutorial you'll need to use Google Chrome, but there are also many nice ways of scraping data from the web using the Python requests library and the BeautifulSoup library, e.g. [here](https://allofyourbases.com/2017/10/08/web-scraping-youtube-in-python/).

1. Open Chrome and navigate to google image search. Now enter your search, e.g. "Zebra"
2. Open the Developer console: either use CTRL+SHIFT+I or go to 'View' --> 'Developer' --> 'Javascript Console'. 
3. The next step is to start scrolling! Keep scrolling until you have found all relevant images to your query.
4. Next is to grab all the urls of the images in your scroll. In the console enter the following commands:

```javascript
/**
 * simulate a right-click event so we can grab the image URL using the
 * context menu alleviating the need to navigate to another page
 *
 * attributed to @jmiserez: http://pyimg.co/9qe7y
 *
 * @param   {object}  element  DOM Element
 *
 * @return  {void}
 */
function simulateRightClick( element ) {
    var event1 = new MouseEvent( 'mousedown', {
        bubbles: true,
        cancelable: false,
        view: window,
        button: 2,
        buttons: 2,
        clientX: element.getBoundingClientRect().x,
        clientY: element.getBoundingClientRect().y
    } );
    element.dispatchEvent( event1 );
    var event2 = new MouseEvent( 'mouseup', {
        bubbles: true,
        cancelable: false,
        view: window,
        button: 2,
        buttons: 0,
        clientX: element.getBoundingClientRect().x,
        clientY: element.getBoundingClientRect().y
    } );
    element.dispatchEvent( event2 );
    var event3 = new MouseEvent( 'contextmenu', {
        bubbles: true,
        cancelable: false,
        view: window,
        button: 2,
        buttons: 0,
        clientX: element.getBoundingClientRect().x,
        clientY: element.getBoundingClientRect().y
    } );
    element.dispatchEvent( event3 );
}
```

```javascript
/**
 * grabs a URL Parameter from a query string because Google Images
 * stores the full image URL in a query parameter
 *
 * @param   {string}  queryString  The Query String
 * @param   {string}  key          The key to grab a value for
 *
 * @return  {string}               value
 */
function getURLParam( queryString, key ) {
    var vars = queryString.replace( /^\?/, '' ).split( '&' );
    for ( let i = 0; i < vars.length; i++ ) {
        let pair = vars[ i ].split( '=' );
        if ( pair[0] == key ) {
            return pair[1];
        }
    }
    return false;
}
```

```javascript
/**
 * Generate and automatically download a txt file from the URL contents
 *
 * @param   {string}  contents  The contents to download
 *
 * @return  {void}
 */
function createDownload( contents ) {
    var hiddenElement = document.createElement( 'a' );
    hiddenElement.href = 'data:attachment/text,' + encodeURI( contents );
    hiddenElement.target = '_blank';
    hiddenElement.download = 'zebra.txt';
    hiddenElement.click();
}
```

```javascript
/**
 * grab all URLs va a Promise that resolves once all URLs have been
 * acquired
 *
 * @return  {object}  Promise object
 */
function grabUrls() {
    var urls = [];
    return new Promise( function( resolve, reject ) {
        var count = document.querySelectorAll(
        	'.isv-r a:first-of-type' ).length,
            index = 0;
        Array.prototype.forEach.call( document.querySelectorAll(
        	'.isv-r a:first-of-type' ), function( element ) {
            // using the right click menu Google will generate the
            // full-size URL; won't work in Internet Explorer
            // (http://pyimg.co/byukr)
            simulateRightClick( element.querySelector( ':scope img' ) );
            // Wait for it to appear on the <a> element
            var interval = setInterval( function() {
                if ( element.href.trim() !== '' ) {
                    clearInterval( interval );
                    // extract the full-size version of the image
                    let googleUrl = element.href.replace( /.*(\?)/, '$1' ),
                        fullImageUrl = decodeURIComponent(
                        	getURLParam( googleUrl, 'imgurl' ) );
                    if ( fullImageUrl !== 'false' ) {
                        urls.push( fullImageUrl );
                    }
                    // sometimes the URL returns a "false" string and
                    // we still want to count those so our Promise
                    // resolves
                    index++;
                    if ( index == ( count - 1 ) ) {
                        resolve( urls );
                    }
                }
            }, 10 );
        } );
    } );
}
```

```javascript
/**
 * Call the main function to grab the URLs and initiate the download
 */
grabUrls().then( function( urls ) {
    urls = urls.join( '\n' );
    createDownload( urls );
} );
```
The last step will download a text file named: 'urls.txt'.

### Step 2: Download all the images

This repo includes a [script](https://github.com/EslamHussein55/zebra_imageClassification/blob/master/download_images.py) to download all the images listed in the url file. You can also always write your own!

The script provided here takes two arguments: (1) the url list file, and (2) the location of the directory where you want the images to be stored. You can run it like this:

```bash
> python download_images.py --urls urls.txt --output images/zebra
```


For classification we're also going to need a set of images that don't contain our target class, i.e. images that are NOT of zebras. Therefore, we randomly selected 400 images from [**Caltech-256 Dataset**](https://authors.library.caltech.edu/7694/).


### Step 3: Image classification

There are many different approaches to image classification. One heavily used method is Convolutional Neural Networks (CNNs) and there's a good example of how to implement a CNN using the [keras library](https://keras.io/) in [this blog](https://www.pyimagesearch.com/2017/12/11/image-classification-with-keras-and-deep-learning/) and [this blog](https://www.pyimagesearch.com/2018/04/16/keras-and-convolutional-neural-networks-cnns/).

In this repo we've provided an example [classifier](https://github.com/EslamHussein55/zebra_imageClassification/blob/master/Classify_Zebra_Images.ipynb) that uses a combination of [Gabor filters](http://scikit-image.org/docs/dev/auto_examples/features_detection/plot_gabor.html) and [Support Vector Machines (SVMs)](https://medium.com/machine-learning-101/chapter-2-svm-support-vector-machine-theory-f0812effc72). 

-----

This tutorial is based on a [JBCA hack challenge](https://github.com/hrampadarath/JBCA_Hack_Night_Dec/tree/master/google_images_webscraping)
