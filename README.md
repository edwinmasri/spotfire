# Integrating Plotly and Spotfire Visualizations

* [Overview](#overview)
* [Compatibility](#compatibility)
* [Spotfire Customization](#spotfire)
* [Required Scripts](#requiredscripts)
* [Constants](#constants)
* [Fields](#fields)
* [Properties](#properties)
* [Customization Attributes](#customizationattributes)
* [Callbacks](#callbacks)
* [Open the Analysis](#openanalysis)
* [Helper Functions](#helperfunctions)
* [Passing/Receiving Parameters](#parameters)
* [Plotly Customization](#plotly)
* [Walkthrough - Example - ControlChartHeatMap](#puttingitalltogether)
* [Known Issues](#knownissues)
* [Source Files](#sourcefiles)
* [JSViz Extension - Integrating with Plotly](#jsviz)
* [Overview](#jsviz_overview)
* [Required Scripts](#jsviz_requiredscripts)
* [Walkthrough - Example - spotfire_plotly.js](#jsviz_walkthrough)
* [Known Issues](#jsviz_knownissues)
* [Source Files](#jsviz_sourcefiles)
* [Troubleshooting](#jsviz_troubleshooting)

<a name="overview"/>
### Overview 

Spotfire is a business intelligence tool developed by TIBCO that is used primarily as an enterprise tool to visualize and analyze sets of data. Plotly is an online analytics and data visualization tool available for both public and enterprise use that is extremely robust with graphing libraries for Python, R, MATLAB, Perl, Julia, Arduino, and REST. Both the Spotfire Web Player and Plotly allow customizations through their APIs and interactive visualizations and dashboards can be created using the two. 

This documentation and associated examples assume that the Spotfire Web Player has been installed and configured to build mash-ups using the Spotfire API. More information on Spotfire and the Web Player can be found at the Spotfire Technology Network. For more information about Plotly, visit the API site.  

<a name="compatibility"/>
### Compatibility 

This code has been tested using Spotfire 7 using both Chrome and Firefox. Plotly.js was used for the Plotly portion of the integration.

<a name="spotfire"/>
##Spotfire - Customizing Visualizations

While this documentation assumes that the Spotfire Web Player has been set up, it is good to first review that the API has been configured to work within Spotfire. 

Within the Web.config file in the Spotfire Web Player directory - the default Web Player directory is at `C:\Program Files\TIBCO\Spotfire Web Player\7.0.0\webroot` - check to make sure that the JavaScript API is enabled. Without this attribute set to `true`, you will not have access to Spotfire’s API and not be able to build customized visualizations or be able to pass and receive parameters. 

Detailed information on how to set up mashups using the Spotfire Web Player is available on the [Spotfire Technology Network](http://stn.spotfire.com/stn/Tutorials/HowToCreateMapMashup.aspx)

An example walkthrough, as well as code, is available within this repo as well. The files needed to follow this documentation reside in the `SpotfirePlotlyDemo` folder. To begin, open the `ControlChartHeatMap.html` file with an editor. This is the main driver of the integration between Spotfire and Plotly.

<a name="requiredscripts"/>
### Required Scripts

At the top of the `ControlChartHeatMap.html` file you will find the required scripts within the `<script>` tags. An important script is the Spotfire API script which allows access to the Spotfire Web Player’s API so that the visualization can be customized. 

The line to include the API script looks like this:

`<script type=”text/javascript” src=”http://<server>/SpotfireWeb/GetJavaScriptApi.ashx?Version=1.0”></script>`

where `<server>` is the server location of the Spotfire Web Player.

<a name="constants"/>
### Constants

The constants within the code are variables that will be used throughout the code. The Spotfire server URL, the path in the Spotfire Library to the analysis file and other properties are set here. 

The constants include:

`c_ServerUrl`  		The Spotfire Web Player URL 
`c_AnalysisPath`  		The path within the Spotfire Library to the Spotfire analysis
`c_parameters` 		Parameters to set up upon initialization, e.g. setting the filters
`c_markingColumns` 	Columns within the analysis file which can be marked/passed to Plotly
`c_tableName`  		The underlying data table for the Spotfire visualization
`c_markingName`  		The name of the marking scheme
`c_filteringScheme` 	The name of the filtering scheme associated with the visualization’s page
`c_startPage` 		The name of the page within the analysis file that should be opened when initialized

Note: Not all of these constants are required in your code. For example, `c_startPage` and `c_filteringScheme` will be set to default values. But `c_ServerURL` and `c_AnalysisPath` are two required constants for setting up the visualization properly.

<a name="fields"/>
### Fields

Fields include variables needed to invoke the custom Spotfire analysis file, as well as the slider that is used to toggle between a full-size Spotfire Web Player and full-size Plotly visualization view. 

`slider` 		At the top of the `PlotlyDemo.html` page there is a slider that allows a user to increase the size of either the Spotfire visualization or the Plotly visualization markings 
`customization`  	Needed to invoke a custom Spotfire Web Player analysis
`app` 			The actual Web Player app, which is created after the page has loaded completely. This groups in all other settings to create the visualization.

<a name="properties"/>
### Properties

Properties are specific to the Spotfire Web Player. In our examples, we use property information to grab data from the Web Player so we can later be passed and used within the Plotly visualization. 

`Columns` 	A mapping of columns that we can later grab information from. This is useful to pass information in our integration. DOM Event Handlers

Within this section of code we set up each component of the customization. The slider, the layout/size of each div container, and various attributes that we want to set when the analysis file loads. 

First, `window.onload` is called to specify that these handlers should be set up when the page is loaded. Each DOM element that we want to manipulate and invoke is set within this section. 

<a name="customizationattributes"/>
### Customization Attributes

`showClose` 			Toggle whether to show the user the close option; options are `true`/`false`
`showAnalysisInfo` 	Toggle whether to show user analysis info about this file; options are `true`/`false`
`showToolBar` 		Toggle whether to show the Spotfire tool bar; options are `true`/`false`

<a name="callbacks"/>
### Callbacks

Within PlotlyDemo.html we have three callback function set up: one for when an error occurs, one for when the analysis file has completely opened, and one for a marking (e.g. a selection within the Spotfire visualization) occurs. 

More specifically, the callbacks within the code include:

`errorCallback`		When an error occurs, this method specifies what steps to take. In the example we display an alert to the user to show them adescription of the error. 

`openedCallback`		When the analysis has been opened, we want to set some properties. In the example, we set the active page and then set up a listener for markings. This listener is useful for interacting with the Plotly visualization and is described in the following callback. 

`markingCallback`		The markingCallback method sets the marked value (from Properties section) and loads the Plotly visualization. This way we can use both the marked value, and the Plotly visualization loads based on the marking. 

<a name="openanalysis"/>
### Open the Analysis

After all constants, fields and callbacks have been setup, we can now open our analysis. 

This can be done with the following line of code:

`app.open(c_analysisPath, <div_container>, c_parameters);`

Where `c_analysisPath` is the location of the analysis file in the Library - set within the [Constants](#constants) section - the `div_container` is the div we want the analysis to load into, and `c_parameters` are the analysis parameters we specified earlier in our code.

<a name="helperfunctions"/>
### Helper Functions

Within this example code, helper functions are utilized to set the layout of both the Web Player and Plotly visualization.

These helper functions include:

`layoutPlotly` 		Set up the layout of the Plotly visualization.
`layoutWebPlayer` 		Set up the layout of the Web Player visualization. 

<a name="parameters"/>
### Passing/Receiving Parameters 

Since the parent container is accessbile to both the Web Player visualiation and the Plotly visualization, all parameters are accessible from each. 

<a name="plotly">
## Plotly - Customized Visualization

The Plotly.js API code should be used as it is standalone and does not rely on Plotly’s servers. All required JavaScript files are included within the package, and visualizations can be hosted directly on the Spotfire Web Player server. The Plotly API allows for visualizations to be quickly setup and is not as complicated as setting up the Spotfire Web Player. The example included within PlotlyHeatmapDemo.html shows that a Plotly visualization can be set up in about 60 lines of code. 

<a name="includedscripts"\>
### Included Scripts

D3.js scripts are needed to properly use Plotly. Additionally...

<a name="creating"\>
### Creating the Visualization

Creating a visualization is simple with Plotly. Within the example, we first load data into the `data` variable. The call to render the heatmap is a single line of code:

`Plotly.plot(‘plotlyWrapper’,data,layout);`

where `plotlyWrapper` is the div container name, `data` is the variable with our data in it and `layout` specifies the layout of the Plotly visualization. 

<a name="parameters_plotly"/>
### Passing/Receiving Parameters

Since the `plotlyWrapper` is a div located on the same page as the Spotfire Web Player visualization, parameters can be quickly retrieved and passed to be used between the two applications. 

Within the Spotfire Web Player, the marked value is stored in a variable called `markedSpotfireValue`. this variable can be accessed within the Plotly visualization as well. For example, `console.log(“Marked value: “ + markedSpotfireValue)` will display the markedValue within the console. 

<a name="puttingitalltogether"\>
## Walkthrough - Example - ControlChartHeatmap

In this first example we will take a look at a basic mashup of Spotfire and Plotly. This will be done using a control chart - a line chart, essentially - in Spotfire and a heatmap within Plotly. Selecting a point within Spotfire will trigger a new heatmap to load within Plotly. 

Within the SpotfirePlotlyDemo folder you will find:

* Spotfire control chart analysis file - `SpotfireControlChartDemo.dxp`
* Plotly heat map visualization - `PlotlyHeatmapDemo.html`
* Custom (Spotfire via JS) code to tie everything together - `ControlChartHeatmap.html`
* JavaScript, CSS and image files for components i.e. `js`, `css`, `img` sub-folders

2. The first step is to upload the package onto the Web Player server, unzip its contents and place the `SpotfirePlotlyDemo` folder in the Web Player's webroot directory. On a Windows machine, the Spotfire Web Player's webroot directory is, by default, located at `C:\Program Files\TIBCO\Spotfire Web Player\7.0.0\webroot` but you should verify that this is the correct location.

i. If the server admin is not available, you can check the Web Player's directory by opening the IIS Manager (under `Administrator Tools`), selecting the site (e.g. `SpotfireWeb`) and then `Explore` from the menu.

![alt text](http://i.imgur.com/nV7B7BT.png)

ii. Once the SpotfirePlotlyDemo folder has been copied into the Web Player directory, a restart of the site (e.g. the Spotfire Web site in the IIS Manager, not the entire server) should be done.

![alt text](http://i.imgur.com/yyfaFzt.png)

3. Open the SpotfireControlChartDemo file with the Spotfire desktop client. Once it has loaded:

  i. Choose `File->Save As->Library Item`
 
  ii. A prompt showing a directory within the Spotfire Library appears.

  ![alt text](http://i.imgur.com/xiqiq0n.png)

  iii. The file should be saved to `Plotly/SpotfireControlChartDemo` 

  iv. If the Plotly folder does not exist, create it by navigating first to the root folder of the entire Web Player's Library and selecting New Folder…
  
  ![alt text](http://i.imgur.com/5tOdPSJ.png)
  
  v. A prompt appears. Enter Plotly as the name of the folder in the Name field.

  ![alt text](http://i.imgur.com/GBgBQVG.png)
  
  vi. Note: If it's not possible to use this path and you must save somewhere else in the Library, that is fine. You will just need to make sure the path is updated within the ControlChartHeatmap.html file. 

The `c_analysisPath` parameter is currently set to `/Plotly/SpotfireControlChartDemo` and should be updated if the Spotfire analysis was saved in another location.

vi. Note: If it's not possible to use this path and you must save somewhere else in the Library, that is fine. You will just need to make sure the path is updated within the `ControlChartHeatmap.html` file. 

The `c_analysisPath` parameter is currently set to `/Plotly/SpotfireControlChartDemo` and should be updated if the Spotfire analysis was saved in another location.

4. Now ensure that you are able to open the file we just saved to the Library, but from a browser instead of using the Spotfire desktop client.

i. To do this, first navigate to the Web Player's address within a browser.  The Web Player URL will be similar to:

`http://dev-server-name/SpotfireWeb`

ii. Enter your credentials if prompted and check off the 'Remember Me' box. 

iii. When you are logged into the Spotfire Web Player, select Browse Library in the upper-right corner. 

iv. Navigate to the path where the file was saved (which was done in step 3) and open the file. If you are unable to open the file for any reason, permissions may first need to be set so that you are able to access it.

![alt text](http://i.imgur.com/uqu3UKD.png)

5. Now let's go back into the Web Player server. From within the Web Player directory on the server - where you copied theSpotfirePlotlyDemo folder to in step 2 - open the ControlChartHeatmap.html file. We need to change the hard-coded location of the Web Player server. Find and replace all instances of `http://<localhost>/SpotfireWeb` with the domain of the Web Player server. There should be three times where this occurs within the file.

i. For instance, if the Web Player is accessed through 
`http://dev-server-name/SpotfireWeb` then `dev-server-name` is the domain that should replace `<localhost>` within the html file

6. Assuming everything is ok, you should now be able to access the mash-up.

 i. It will be accessible at
 `<Web Player Root>/SpotfirePlotlyDemo/ControlChartHeatmap.html` 

    e.g. 
`http://dev-server-name/SpotfireWeb/SpotfirePlotlyDemo/ControlChartHeatmap.html`

7. If an error occurs at any point, check the debugging console (Ctrl+Shift+I in Chrome on a Windows machine). 

i. In addition to any errors that the Web Player may give, I have also included console logging within the code that will help with debugging if needed. 

![alt text](http://i.imgur.com/qr6B34T.png)

8. When the mash-up loads successfully, you will see the Web Player analysis file on the left and the Plotly visualization on the right. 

![alt text](http://i.imgur.com/9QIcl1Q.png)

i. Selecting any marking on the Spotfire chart (left) will cause the Plotly visualization (right) to update. 

ii. Hovering over cells within the heatmap will give more information about values. 

![alt text](http://i.imgur.com/YqdwnOn.png)

iii. You will also notice two arrows at the top of the page. By selecting an arrow, a user can enlarge the page's area of either a Spotfire or Plotly visualization, depending on which one you select. 

![alt text](http://i.imgur.com/gPjxuOg.png)

iv. Additionally, there are options at the top of the Spotfire visualization in the left frame.

![alt text](http://i.imgur.com/gAkHkpy.png)

v. Selecting the Filter icon will toggle whether the filter panel is shown or not. Selecting items within the  
filter panel will allow users to look at a subset of the data within the visualization.

![alt text](http://i.imgur.com/uPejsel.png)

![alt text](http://i.imgur.com/JHhiJTN.png)


vi. Other options include Twitter sharing, exporting and general help. 

vii. By hovering over a blank area on the Plotly visualization, an options toolbar similar to Spotfire's will appear. 

![alt text](http://i.imgur.com/vSdqyHN.png)

<a name="sourcefiles" />
## Source Files

The example code is located within this Github repository in the [examples/SpotfirePlotlyDemo](https://github.com/plotly/spotfire/tree/master/examples/SpotfirePlotlyDemo) directory.

<a name="knownissues" />
## Known Issues

### Cross-domain Scripting

Cross-domain scripting should be taken into consideration when setting up a Web Player/Plotly integration. The examples within this document assume that Plotly visualizations have been set up within a folder in the Web Player directory, or a virtual folder that is accessible within the same URL as the Web Player. 

If the Plotly visualizations are located on a different server within the same network, then the document.domain attribute will need to be set. 

For example, if the Spotfire Web Player is installed and set up on webplayer.company-domain.com and Plotly visualizations exist on plotly.company-domain.com then the document.domain attribute within the code for both visualizations should be set to:

`document.domain = company-domain.com`

If the Plotly visualization files are set up on a completely different server than other methods will need to be used in order to allow cross-domain scripting. Cross-origin Resource Sharing (CORS) or using the postMessage function are two possible solutions in these instances. 

<a name="Troubleshooting" />
## Troubleshooting

Google Chrome includes a console that is helpful for debugging. Similarly, Firefox has Firebug. These debugging tools are useful when attempting to figure out issues with a custom mashup. Inserting `console.log(...);` with the variable or object inserted between the parentheses throughout your code will help with debugging any issues.

<a name="jsviz" />
## JSViz Extension - Integration with Plotly  

<a name="jsviz_overview"/>
### Overview

It is possible to also render Plotly charts with the Spotfire JSViz
extension. The JSViz extension is a plugin that allows users to create
visualizations using JavaScript libraries such as D3, but still allows
them to integrate with the Spotfire platform. 

Since Plotly uses D3, all visualizations can be rendered within the
JSViz visualization space and allow for Spotfire interactivity. There
are some required js files and other code before a Plotly visualization
can be displayed within Spotfire. This section will walk you through how
to do so and provides an example Plotly rendering within Spotfire which shows the following:

![alt text](http://i.imgur.com/bLPOO0k.png)

Within the JSViz extension file, there are a number of examples, tutorials and documentation. This documentation includes information on the steps needed to properly render a visualizations not included within Spotfire and using JS libraries such as D3.js.

### Desktop Client Implementation

The implementation of the custom extension is different for the Spotfire Desktop Client and the Spotfire Web Player, and both can be implemented within a single js file if needed. 

In the Desktop Client, a Microsoft Web Browser control is used as the container for the JavaScript files containing the display code. The actualy JavaScript files can either be hosted on a Web Server instance or embedded within the DXP file itself.

To set up a visualization within JSVIz, the following steps should take place:

1. The Spotfire Professional client starts up and loads the JS Visualization extension. As the extension is part of the memory space of the Spotfire Professional executable, it has access to the internal components of Spotfire such as the Data Engine, the Scripting Engine etc. 
2. A visualization that uses the JS Visualization extension is created or a DXP file is loaded containing an existing instance. 
3. The view for the JS Visualization is created. This is an embedded instance of the Microsoft Web Browser control. 
4. The Web Browser control loads the HTML or JavaScript page containing the visualization code from the external Web Server. This code contains two special hooks: a. A JavaScript callback function that takes a JSON data string and populates the visualization from the data. b. A handler for the “spotfireready” event that registers the above callback. 
5. Once the Web Browser control completes loading the HTML or JavaScript page a callback is made that triggers the injection of some JavaScript code into the DOM of the Web Browser Control. This injected code first creates a DOM object called “Spotfire” which is an alias for the JavaScript window.external object. 
6. The injected code then creates and fires an event called “spotfireready”. This event is captured within the visualization code (4b above) and registers the data callback function (4a above) using the method Spotfire.registerSelectionCallback. 
7. Now, whenever the data or configuration change, the plugin calls the callback defined in 4a to render the JavaScript visualization. 

### Web Player Implementation

When deployed within the Web Player, the JavaScript or HTML visualization code is hosted inside an IFrame created by the Web Player framework. Once again, the actual files are hosted on a Web Server instance. 

Unlike other visualizations in the Web Player, which are rendered as image files, the JavaScript drawing code is executed within the users Web Browser. Whenever the framework detects that a change has been made to the underlying data, for instance due to marking or filtering, it refreshes the contents of the IFrame. 

The sequence of events is as follows: 

1. The Web Player starts up and loads the JS Visualization extension. As the extension is part of the memory space of the Spotfire Web Player executable, it has access to the internal components of Spotfire such as the Data Engine, the Scripting Engine etc. 
2. The user opens a DXP file containing a visualization that uses the JS Visualization extension. 
3. The Web Player server creates an IFrame on the HTML page that will host the JS Visualization instance. The web player then writes the source location of all the required JavaScript files and writes the body HTML for the visualization page to the user’s browser. 
4. The IFrame requests its content from the Web Player server which routes the request through to the JS Visualization extension. The extension returns the body HTML for the visualization page to the user’s browser. Within the HTML will be tags referring to the location of the JavaScript and CSS files that make up the visualization. These can either point to external Web Server addresses or back to the JavaScript Visualization extension for embedded content. 
5. The IFrame reads the JavaScript and CSS files from the specified locations and initializes any JavaScript loaded. 
6. When the <DIV> tag that will host the JavaScript visualization is written to the document, an HTTP POST Request is made to the Web Player Server to retrieve the data for the visualization in JSON format. The returned data is then used to render the visualization. 
7. Whenever the user changes a marking or filter that affects the underlying data for the visualization the Web Player Server refreshes the IFrame contents which causes either step 6 to be repeated, or in some cases steps 4 through 6 above to be repeated. 

When the user marks some data points in the visualization or executes a script, these events are sent to the Web Player via a HTTP request which routes them to the JS Visualization Extension 

<a name="requiredscripts"/>
### Required Scripts

The JSViz extension should first be installed in order to render Plotly
charts within Spotfire. In order for a Plotly.js visualization render correctly, 
the following JavaScript files should be included:

* [d3.min.js](https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.6/d3.min.js)
* [jquery-2.1.4.js](https://code.jquery.com/jquery-2.1.4.js)
* plotly.min.js
* typedarray.js
* [underscore.min.js](https://cdnjs.cloudflare.com/ajax/libs/underscore.js/1.8.3/underscore.js)

The last required js file - which renders the visualization itself - includes Spotfire functions and 'hooks.' These functions and hooks are documented within the JSViz documentation itself and the next sections will walk you through one such example.  

### Required Functions and divs

#### renderCore()

The renderCore method is the main component to render the visualization. 

#### js_chart div

js_chart is the main id that is used to append visualization components to. Within spotfire_plotly.js, and in the renderCore function, sub-divs are first created for reach of the six visualizations to display. These are inline divs and their styles are adjusted accordingly as well. 


<a name="jsviz_walkthrough"/>
## Walkthrough - Example - spotfire_plotly.js

In this section we will guide you through how to create a Plotly chart which is rendered within Spotfire. The example plot will consist of three columns: heatmaps on the top row, and histograms on the bottom row. The data which drives the example integration is random and not associated with any 'real' dataset. 

![alt text](http://i.imgur.com/bLPOO0k.png)



### spotfire_plotly.js

This is the 'last' of the required files - mentioned in the Required Scripts section of this markdown file - is the spotfire_plotly.js file. This js file is the driver that renders the Plotly visualization within Spotfire. 

There are two main types of code/functions/etc within this file: 

1. JSViz and Spotfire code, which are explained within the JSViz documentation and 
2. Plotly code, which are explained both here and, in general, in the Plotly.js documentation

As mentioned above, both renderCore and the js_chart div can be found in this script. DIVs are first created and appeded to js_chart and then the Plotly visualizations are rendered. Functions that are used by Plotly include makeHeatmap and makeHistogram

Other functions within this file are used by the extension to hook into the Spotfire framework. This includes functions for marking, as well as access to the Spotfire data itself. 

<a name="jsviz_sourcefiles" />
## Source Files

The example code is located within this Github repository in the [examples/PlotlyJSViz](https://github.com/plotly/spotfire/tree/master/examples/PlotlyJSViz) directory. 

<a name="jsviz_knownissues" />
## Known Issues

<a name="jsviz_troubleshooting" />
## Troubleshooting

...