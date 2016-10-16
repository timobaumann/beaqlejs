
Table of contents

* [Description](#description)
* [Basic Setup](#basic-setup)
* [Test Configuration](#test-configuration)
* [Browser Support](#browser-support)
* [Online Submission](#online-submission)
* [Internals](#internals)
* [Contact](#contact)
* [License](#license)


# Description #

BeaqleJS (**b**rowser based **e**valuation of **a**udio **q**uality and **c**omparative **l**istening **e**nvironment) provides a framework to create browser based listening tests and is purely based on open web standards like HTML5 and Javascript. Therefore, the test runs in any modern web browser and allows an easy distribution of the test environment to a significant amount of participants in combination with simple configuration. Currently it supports ABX and MUSHRA style test procedures but can be easily extended to other test schemes.

To get a better impression about its functionality there are two demo test sites for the ABX and MUSHRA test classes:

* <http://hsu-ant.github.io/beaqlejs/demo/abx/>
* <http://hsu-ant.github.io/beaqlejs/demo/mushra/>

BeaqleJS has been presented at the [Linux Audio Conference 2014](http://lac.linuxaudio.org/2014/) at the ZKM in Karlsruhe, Germany. The original [paper](http://lac.linuxaudio.org/2014/papers/26.pdf) and presentation [slides](http://lac.linuxaudio.org/2014/download/SKraft_BeaqleJS.pdf) are available from the conference archive. However, meanwhile most of the paper content has been updated and merged into this documentation.

If you want to cite BeaqleJS in a publication please use

> S. Kraft, U. Zölzer: "BeaqleJS: HTML5 and JavaScript based Framework for the Subjective Evaluation of Audio Quality", _Linux Audio Conference_, 2014, Karlsruhe, Germany

as a reference or link to our GitHub repository
    
> https://github.com/HSU-ANT/beaqlejs


# Basic Setup #

1. Download the test scripts
    - you can either get a stable release from  
      <https://github.com/HSU-ANT/beaqlejs/releases>
    - or clone the git repository  
      `git clone https://github.com/HSU-ANT/beaqlejs.git`
    - or simply download a zip of the current HEAD revision at  
      <https://github.com/HSU-ANT/beaqlejs/archive/master.zip>

2. Uncomment the line where the desired test class is initialized and loaded in the header of the `index.html` file.

    ```html
    <script type="text/javascript">
      var testHandle;
      window.onload=function() {
        // Uncomment one of the following lines to choose the desired test class
        //testHandle = new MushraTest(TestConfig);  // <= MUSHRA test class
        //testHandle = new AbxTest(TestConfig);     // <= ABX test class
      };
    </script>
    ```

3. Prepare a config file and set its path in the prepared `<script></script>` tag in the header of the `index.html` file.

    ```html
    <!-- load the test config file -->
      <script src="config/YOUR_CONFIG.js" type="text/javascript"></script>
    <!---->
    ```

    Two example config files for the MUSHRA and ABX test class are already supplied in the `config/` folder to serve as a starting point. Detailed information about the different test classes and configuration can be found below.


# Test Configuration #

## General Options ##

The available options can be dividided into a set of general options whitch apply to all test classes and other options, including file declarations, that are specific for a single test class.

```javascript
var TestConfig = {
  "TestName": "My Listening Test",   // <=  Name of the test
  "LoopByDefault": true,             // <=  Enable looped playback by default
  "ShowFileIDs": false,              // <=  Show file IDs for debugging (never 
                                     //     enable this during real test!)
  "ShowResults": false,              // <=  Show table with test results at the end
  "EnableABLoop": true,              // <=  Show controls to loop playback with an 
                                     //     AB range slider
  "EnableOnlineSubmission": false,   // <=  Enable transmission of JSON encoded 
                                     //     results to a web service
  "BeaqleServiceURL": "",            // <=  URL of the web service
  "SupervisorContact": "",           // <=  Email address of supervisor to contact for 
                                     //     help or for submission of results by email
  "RandomizeTestOrder": true,        // <=  Present test sets in a random order
  "MaxTestsPerRun": -1,              // <=  Only run a random subset of all available 
                                     //     tests, set to -1 to disable
  "AudioRoot": "",                   // <=  path to prepend to all audio URLs in the testsets
  "Testsets": [ {...}, {...}, ... ], // <=  Definition of test sets and files, more 
                                     //     details below
}
```

## ABX ##

In an ABX test three items named A, B and X are presented to the listener, whereas X is randomly selected to be either the same as A or B. The listener has to identify which item is hidden behind X, or which one (A or B) is closest to X. If the listener is able to find the correct item, it reveals that there are perceptual differences between A and B. 

A typical application of ABX tests would be the evaluation of the transparency of audio codecs. For example item A could be an unencoded audio snippet and B is the same snippet but encoded with a lossy codec. When the listener is not able to identify if A or B was hidden in X (results are randomly distributed), one can assume that the audio coding was transparent

```javascript
...                                  // <=  General options
"Testsets": [
  { "Name": "Schubert",              // <=  Name of the test set
    "TestID": "id1_1",               // <=  Unique test set ID, necessary for internal
    "Files": {                       // <=  Array with test files
      "A": "audio/schubert_ref.wav", // <=  File A
      "B": "audio/schubert_2.wav",   // <=  File B
      }
  },
  { ... },                           // <=  Next test set starts here
  ....
]
```

## Preference Testing ##

A preference test (or A/B test) is a simplified version of ABX in which the listener is to decide which of two stimuli sounds better. A typical application of preference tests would be the comparison of different speech synthesis methods. Over a range of stimuli and listeners, one method (A or B) might be shown to be better than the other (one might use a sign test for comparison of results for A and B). Configuration is identical to the ABX test; however, no X is presented to the user.

## MUSHRA ##

In a MUSHRA test (ITU-R BS.1116-1) the listener gets presented an item marked as reference together with several anonymous test items. By using a slider for each test item he has to rate how close the items are to the reference on top. Among the test items there is usually also one hidden reference and one, or several, anchor signals to prove the validity of the ratings and the qualification of the participants.

Contrary to ABX tests the MUSHRA procedure allows more detailed evaluations as it is possible to compare more than one algorithm to a reference. Furthermore, the results are on a continuous scale allowing a direct numerical comparison of all algorithms under test.

```javascript
...                                   // <=  General options
"RateMinValue": 0,                    // <=  Minimum rating
"RateMaxValue": 100,                  // <=  Maximum rating
"RateDefaultValue":0,                 // <=  Default rating
"Testsets": [
  { "Name": "Schubert 1",             // <=  Name of the test set
    "TestID": "id1_1",                // <=  Unique test set ID, necessary for 
                                      //     internal referencing
    "Files": {                        // <=  Array with test files
      "Reference": "audio/ref.wav",   // <=  Every MUSHRA test set needs exactly 
                                      //     one(!) file with a "Reference" label
      "label_1": "audio/algo_1.wav",  // <=  Various files to be tested, the labels 
      "label_2": "audio/algo_2.wav",  //     can be freely chosen as desired but
      "label_3": "audio/algo_3.wav",  //     have to be unique inside a test set
      "anchor": "audio/algo_anc.wav", //      ...
      }
  },
  { ... },                            // <=  Next test set starts here
  ....
]
```


# Browser Support #

BeaqleJS in general will run well in any recent web browser out in the wild. The only noteworthy exceptions are the Internet Explorer versions below 9 which still have a market share of a few percent and unfortunately miss the required FileAPI. Participants will get a warning if they open the listening test with one of these old versions.

## Required HTML5 Features ##

* Audio playback using HTML5 is widely supported by all major browsers since many years (except IE below version 9.0). ([list browsers](http://caniuse.com/#feat=audio))

* FileAPI-Blob is necessary to provide the listening test results as a virtual download to be saved on the local harddisk. This API can be expected to be available in every browser of the last years  (except IE below version 9.0). ([list browsers](http://caniuse.com/#feat=blobbuilder))

Optionally:

* WebAudioAPI is used in BeaqleJS for smooth fade in/out at start/stop of playback and at the loop borders. It currently only works reliably with browsers based on the Chromium engine, although it is available in every major browser apart from the Internet Explorer. ([list browsers](http://caniuse.com/#feat=audio-api))

## Codecs ##

Although most browsers today support the HTML5 `<audio>` tag, the supported formats and codecs vary a lot. Unfortunately no browser directly supports FLAC or other lossless compression so far. The only lossless, but also uncompressed, format widely accepted is WAV PCM with 16 bit sample precision. Solely the Internet Explorer is not capable to play back this file type.

Format     |  IE   | Firefox | Chrome |  Opera | Safari
-----------|-------|---------|--------|--------|--------
WAV PCM    |  no   |  > 3.5  |  yes   | > 11.0 | > 3.1
Ogg Vorbis |  no   |  > 3.5  |  yes   | > 10.5 | XiphQT
MP3        | > 9.0 |  > 26*  |  yes   | > 14   | > 3.1
ACC        | > 9.0 |  > 26*  |  yes   | > 14   | > 3.1

(* not on Mac OS X)


# Online Submission #

BeaqleJS can send the test results in JSON format to a web service to collect them in a central place. An examplary server side PHP script which can be used to receive and store the results is included in the `web_service/` subfolder. It only requires a webspace with PHP version 5.

## Setup ##

1. Upload the file `web_service/beaqleJS_Service.php` to a webserver. Create a folder named `results/` next to the PHP script and make sure that the webserver has write permissions on it.

2. Try to execute the script in your browser. For example, point your browser to `http://yourdomain.com/mysubfolder/beaqleJS_Service.php`. The script performs a self-test and checks PHP version and write permission of the `results/` folder.

3. Enable online submission in the BeaqleJS config (`"EnableOnlineSubmission": true`) and set the `BeaqleServiceURL` to `http://yourdomain.com/mysubfolder/beaqleJS_Service.php`.

## Security ##

As with every public web service it is important to be aware about security aspects. In the current implementation there is no possibility to authenticate submitters, therefore everyone can potentially submit spoofed results if he is able to do some basic reverse engineering! However, this is also a common risk with every other public and open survey platform.

There are two provisions to avoid spamming of your sever:

* The results JSON object is limited to a size of 64kB which is a lot for listening test results, but not enough to abuse your server as a hosting facility
* File names in the `results/` folder contain a random string, so it is not possible to access the submitted data without listing the whole directory


# Internals #

The general structure of BeaqleJS can be divided in three blocks (Fig. 1). There is a common HTML5 `index.html` file to hold the main HTML structure with some basic place holder blocks whose content will be dynamically created by the JavaScript backend. The styling is completely independent and done with the help of cascading style sheets (CSS). Style sheets, config files and all necessary JavaScript libraries are loaded in the header of the `index.html`. Most of the descriptive text, like introduction and instructions, are placed in hidden blocks inside this file and their visibility is controlled by the scripts. For the user interface and to simplifiy Document Object Model (DOM) manipulations, the well known [jQuery](https://jquery.com/) and [jQueryUI](http://jqueryui.com/) libraries are used.

The JavaScript backend consists of two main classes. The first one is the `AudioPool` which takes care of audio playback and buffering. It pools a set of HTML5 `<audio>`-tags in a certain AudioPool `<div>`-tag. There are simple functions to add and load a new file, connect  and address it with an ID, manage playback and looping as well as synchronized pause and stop operations.

The `ListeningTest` class provides the main functions of an abstract listening test. This includes the setup and management of basic playback controls (play, pause, looping, time line display, ...), reading of the test configuration as well as storage of the results and also main control over the test sequence.

To create a certain test type the abstract `ListeningTest` class is inherited and specific functions for the actual arrangement of test items or storage and evaluation of the results need to be implemented. Based on this modular approach it is very easy to extend the framework  with additional test types or to create variants of existing ones without the need to reimplement all the necessary basics.

If the test is performed distributed over the internet or on several local computers, the `ListeningTest` main class is also able to send the final ratings to a web service for centralised collection and evaluation.

## Creating a new test class ##

A new test class has to inherit the base functionality from the main `ListeningTest` class. Inheritance in JavaScript is achieved by prototypes and this means to define a new class `MyTest` and then set its prototype to the base class. As this overwrites the constructor it has to be reset to the child constructor afterwards: 

```javascript
// inherit from ListeningTest
function MyTest(TestData) {
  ListeningTest.apply(this, arguments);
}
MyTest.prototype = new ListeningTest();
MyTest.prototype.constructor = MyTest;

// implement the necessary functions
MyTest.prototype.createTestDOM = ...
MyTest.prototype.saveRatings   = ...
MyTest.prototype.readRatings   = ...
MyTest.prototype.formatResults = ...
```

The child class can access the `TestState` and the `TestConfig` structures from the parent:

```javascript
ListeningTest.TestState = {
    // main public members
    'CurrentTest': -1,	
    'TestIsRunning': false,
    'FileMappings': {},
    'Ratings': {},
    'EvalResults': {},
    // ...
    // optionally add own fields
    // ...
}

ListeningTest.TestConfig = {
    "TestName": "Test",
    "LoopByDefault": true,
    "EnableABLoop": true,
    "EnableOnlineSubmission": false,
    "BeaqleServiceURL": "http://...",
    "SupervisorContact": "super@visor.com",
    "Testsets": [
        {
            "Name": "Testset 1",
            "Files": {
                // ...
            }
        },
        {
            "Name": "Testset 2",
            "Files": {
                // ...
            }
        }
        ],
    // ...
    // further test specific settings
    // ...
}
```

The `TestState` should be used to store random file mappings, ratings and other status variables, but can also be dynamically expanded with specific fields required by the child class. The `TestConfig` structure is just a mapping of the BeaqleJS config file into the class namespace. It has to contain at least the fields and structure as given above but it is possible to add additional sections which are required by the child class.

Every new test class has to implement at least four new functions:

* `createTestDOM(TestIdx)` creates the visible layout and HTML structure of the test with the index `TestIdx` based on the test configuration. All the necessary information from the config is available inside the object in `this.TestConfig.*`. Audio files should be appended to the `AudioPool` with `this.addAudio()` and can then be connected to play buttons by unique file IDs. Random mapping of filenames to IDs can be stored in the prepared `this.TestState.FileMappings[TestIdx]` structure.

* `saveRatings(TestIdx)` is used to obtain the ratings from the sliders or buttons in the DOM and to store them in an arbitrary format in the predefined `this.TestState.Ratings[TestIdx].*` object.

* `readRatings(TestIdx)` is intended to read the ratings for \texttt{TestIdx} from `this.TestState.Ratings[TestIdx]` and to reapply them to the sliders or buttons in the DOM. This is primarily used during switching back and forth in the test sequence.

* `formatResults(TestIdx)` is automatically called after the final test in the sequence. It is supposed to evaluate and summarize the ratings and to store the final results in `this.TestState.EvalResults`. It should return a string containing the results formatted in a human readable manner (HTML). This will be presented to the listener after the last test and the `EvalResults` structure may be send to a web service (see [Online Submission](#online-submission)).


# Contact #

<http://hsu-ant.github.io/beaqlejs>

skraft (AT) hsu-hh.de


# License #

The complete sources, html and script files as well as images are released unter the *GPLv3 
license*. A copy of the GPL is provided in the `LICENSE.txt` file.

