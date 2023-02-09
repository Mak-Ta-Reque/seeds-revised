# SEEDS Revised

[![Build Status](https://travis-ci.org/davidstutz/seeds-revised.svg?branch=master)](https://travis-ci.org/davidstutz/seeds-revised)

**Update:** SEEDS Revised is also part of [davidstutz/superpixel-benchmark](https://github.com/davidstutz/superpixel-benchmark).

**Update:** SEEDS Revised is also available as part of [Superpixels Revisited](https://github.com/davidstutz/superpixels-revisited), a library providing command line tools for seven state-of-the-art superpixel algorithms.

Implementation of the superpixel algorithm called SEEDS [1] described in

    [1] M. van den Bergh, X. Boix, G. Roig, B. de Capitani, L. van Gool.
        SEEDS: Superpixels extracted via energy-driven sampling.
        Proceedings of the European Conference on Computer Vision, pages 13–26, 2012.

If you use this code, please cite [1] and [2]:

    [2] D. Stutz, A. Hermans, B. Leibe.
        Superpixel Segmentation using Depth Information.
        Bachelor thesis, RWTH Aachen University, Aachen, Germany, 2014.

Also consider citing [3]:

	[3] D. Stutz.
		Superpixel Segmentation: An Evaluation.
		Pattern Recognition (J. Gall, P. Gehler, B. Leibe (Eds.)), Lecture Notes in Computer Science, vol. 9358, pages 555 - 562, 2015.
		
**Note:** Evaluation results are now available online (to view or download) at [http://davidstutz.de/projects/superpixelsseeds/](http://davidstutz.de/projects/superpixelsseeds/).

[2] is available online at [http://davidstutz.de/projects/superpixelsseeds/](http://davidstutz.de/projects/superpixelsseeds/).

Note that all results published in [2] are based on an extended version of the Berkeley Segmentation Benchmark [4], the Berkeley Segmentation Dataset [4] and the NYU Depth Dataset [5].

    [4] P. Arbeláez, M. Maire, C. Fowlkes, J. Malik.
        Contour detection and hierarchical image segmentation.
        Transactions on Pattern Analysis and Machine Intelligence, 33(5):898–916, 2011.
    [5] N. Silberman, D. Hoiem, P. Kohli, R. Fergus.
        Indoor segmentation and support inference from RGBD images.
        Proceedings of the European Conference on Computer Vision, pages 746–760, 2012.

The extended version of the Berkeley Segmentation Benchmark is available on GitHub: [https://github.com/davidstutz/extended-berkeley-segmentation-benchmark](https://github.com/davidstutz/extended-berkeley-segmentation-benchmark).

![Example: several superpixel segmentations.](screenshot.png?raw=true "Example: several superpixel segmentations")

## Compile

SEEDS Revised is based on [OpenCV](http://opencv.org/) (see [here](http://docs.opencv.org/doc/tutorials/introduction/linux_install/linux_install.html#linux-installation) how to install OpenCV on Linux) and [Boost](http://www.boost.org/) and easily compiled using [CMake](http://www.cmake.org/):
    
    # Prerequisites assuming OpenCV is already installed.
    sudo apt-get install build-essential cmake libboost-dev-all
    # Install OpenCV using the link above or:
    sudo apt-get install libopencv-dev
    # Clone the repository:
    $ git clone https://github.com/Mak-Ta-Reque/seeds-revised.git
    # Go to the build subfolder to generate the CMake files:
    $ cd seeds-revised/build
    $ cmake ..
    # Compile the library and corresponding Command Line Interface:
    $ make

The binaries will be saved to `seeds-revised/bin`. The command line interface offers the following options:

    $ ../bin/reseeds_cli --help
    Allowed options:
        --help                          produce help message
        --input arg                     the folder to process, may contain several 
                                  images
        --bins arg (=5)                 number of bins used for color histograms
        --neighborhood arg (=1)         neighborhood size used for smoothing prior
        --confidence arg (=0.100000001) minimum confidence used for block update
        --iterations arg (=2)           iterations at each level
        --spatial-weight arg (=0.25)    spatial weight
        --superpixels arg (=400)        desired number of supüerpixels
        --verbose                       show additional information while processing
        --csv                           save segmentation as CSV file
        --contour                       save contour image of segmentation
        --labels                        save label image of segmentation
        --mean                          save mean colored image of segmentation
        --output arg (=output)          specify the output directory (default is 
                                  ./output)

## Usage

The library contains two classes:

* `SEEDSRevised`: the original algorithm as proposed in [1].
* `SEEDSRevisedMeanPixels`: an extension using mean pixel updates as discussed in [1].

Thorough documentation can be found within the code. The following example will demonstrate the basic usage of `SEEDSRevisedMeanPixels`:

    #include <opencv2/opencv.hpp>
    #include "SeedsRevised.h"
    #include "Tools.h"

    // ...

    cv::Mat image = cv::imread(filepath);
    
    // Number of desired superpixels.
    int superpixels = 400;
    
    // Number of bins for color histograms (per channel).
    int numberOfBins = 5;
    
    // Size of neighborhood used for smoothing term, see [1] or [2].
    // 1 will be sufficient, >1 will slow down the algorithm.
    int neighborhoodSize = 1;
    
    // Minimum confidence, that is minimum difference of histogram intersection
    // needed for block updates: 0.1 is the default value.
    float minimumConfidene = 0.1;
    
    // The weighting of spatial smoothing for mean pixel updates - the euclidean
    // distance between pixel coordinates and mean superpixel coordinates is used
    // and weighted according to:
    //  (1 - spatialWeight)*colorDifference + spatialWeight*spatialDifference
    // The higher spatialWeight, the more compact superpixels are generated.
    float spatialWeight = 0.25;
    
    // Instantiate a new object for the given image.
    SEEDSRevisedMeanPixels seeds(image, superpixels, numberOfBins, neighborhoodSize, minimumConfidence, spatialWeight);

    // Initializes histograms and labels.
    seeds.initialize();
    // Runs a given number of block updates and pixel updates.
    seeds.iterate(iterations);
    
    // Save a contour image to the following location:
    std::string storeContours = "./contours.png";

    // bgr color for contours:
    int bgr[] = {0, 0, 204};
    
    // seeds.getLabels() returns a two-dimensional array containing the computed
    // superpixel labels.
    cv::Mat contourImage = Draw::contourImage(seeds.getLabels(), image, bgr);
    cv::imwrite(store, contourImage);

## OpenCV 3 Compatibility

The implementation is compatible with OpenCV 2 and OpenCV 3 and tries to detect the used version automatically. However, as some constants changed in OpenCV3, the code may be slightly adapted when using development releases of OpenCV3. In particular, this relates to the following constants:

    CV_BGR2GRAY
    CV_BGR2Lab

## License

Licenses for source code corresponding to:

D. Stutz. **Superpixel Segmentation using Depth Information.** Bachelor Thesis, RWTH Aachen University, 2014.

D. Stutz. **Superpixel Segmentation: An Evaluation.** Pattern Recognition (J. Gall, P. Gehler, B. Leibe (Eds.)), Lecture Notes in Computer Science, vol. 9358, pages 555 - 562, 2015.

Copyright (c) 2014-2018 David Stutz, RWTH Aachen University

**Please read carefully the following terms and conditions and any accompanying documentation before you download and/or use this software and associated documentation files (the "Software").**

The authors hereby grant you a non-exclusive, non-transferable, free of charge right to copy, modify, merge, publish, distribute, and sublicense the Software for the sole purpose of performing non-commercial scientific research, non-commercial education, or non-commercial artistic projects.

Any other use, in particular any use for commercial purposes, is prohibited. This includes, without limitation, incorporation in a commercial product, use in a commercial service, or production of other artefacts for commercial purposes.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

You understand and agree that the authors are under no obligation to provide either maintenance services, update services, notices of latent defects, or corrections of defects with regard to the Software. The authors nevertheless reserve the right to update, modify, or discontinue the Software at any time.

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software. You agree to cite the corresponding papers (see above) in documents and papers that report on research using the Software.
