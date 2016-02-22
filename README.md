# ArcObject .Net with VS 2013 #

Please take this repository as a memo to walk you through the AO setup, debug and development...

If you can't do, you teach :)


## Prerequisites: Installation##
## Lesson 1: Setup Environment and Debug ##
OK, let's start up our first AO project in VS2013! On menu bar: FILE -> New Project, then open the tree structure inside the left pane on the opening dialog, find Visual C# -> ArcGIS -> Desktop Add-ins -> ArcMap Add-in.

In the ArGIS Add-Ins Wizard, make sure to check "Button" option under Add-in Types. This will create a template of a button control for ArcMap.
You should be able to find a cs file with name "Button1.cs". Please replace this file with the on at [here](https://github.com/hellocomrade/arcobject/blob/master/lesson1/Button1.cs).
Before we build the button control, open Soultion Explorer of your VS 2013, right-click on the project name and choose properties. Double Check 2 places:

1. Target framework under Application section should be ".NET Framework 4";
2. Under Debug section, make sure the "Start external program" is set as your ArcMap program;

Still under Solution Explorer, right click on the References and choose "Add ArcGIS References...". We need the following two references in order to build this solution. They are not included by this default template:

1. ESRI.ArcGIS.Geometry
2. ESRI.ArcGIS.Carto

Now you are ready to build the solution! Before you do that, there is still some housekeeping we have to do on ArcMap end. Go to your ArcMap installation folder, usually it is located at: C:\Program Files (x86)\ArcGIS\Desktop10.3\bin, find a file named "ArcMap.exe.config". Open it up with administrator privilege. It is an XML file and search the following section located at the top of the file:
```xml
<startup>
    <!--<supportedRuntime version="v4.0.30319"/>-->
    <supportedRuntime version="v2.0.50727"/>
</startup>
```
By default, ArcMap is configured to run against .Net Framework Runtime 2.0, which is a conflict with the button control we are going to build in VS 2013. Comment out that line and uncomment the line above to enable .Net Framework Runtime 4.0 for ArcMap. OK, now you should be able to build it without any issue. In order to make sure we are on the same page for the following section, please check your "config.esriaddinx" file. You can find this file under Solution Explorer. Here is what I have (I did remove unrelated sections)
```xml
<ESRI.Configuration xmlns="http://schemas.esri.com/Desktop/AddIns" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Name>ArcMapAddin1</Name>
  ......
  <Targets>
    <Target name="Desktop" version="10.3" />
  </Targets>
  <AddIn language="CLR4.0" library="ArcMapAddin1.dll" namespace="ArcMapAddin1">
    <ArcMap>
      <Commands>
        <Button id="ArcMapAddin1_Button1" class="Button1" message="Add-in command generated by Visual Studio project wizard." caption="My Button" tip="Add-in command tooltip." category="Add-In Controls" image="Images\Button1.png" />
      </Commands>
    </ArcMap>
  </AddIn>
</ESRI.Configuration>
```
It gives us a good summary on the tool we just built. Its name is "ArcMapAddin1" and its target is against ArcMap 10.3 running against .Net Framework Common Lauguage Runtime 4.0. This button control was given the caption as "My Button" and categorized under "Add-In Controls". Keep these in mind, you will need them later.

If we click the "Start" button with green arrow in VS2013, you should be able to see ArcMap is starting. VS2013 will do all the dirty jobs for us: invoke ArcMap 10.3 and attach the debugger to its process, load all necessary symbols from various assemblies to  facilitate debug...We are then ready to debug!

You can put a breakpoint on any line of source in Button1.cs, but nothing happens, right? It's because this control is a UI component and requires the user to click on it to invoke any action. So, where is our button? It is not on the UI! Well, it is really inconvenient and ESRI can do a better job by automatically adding the button onto the toolbar during debug. 

Anyway, we will have to do this manaully. You will need to go to Customize -> Add-In Manager, you should be able to see our button under "My Add-Ins", thanks to ESRI! The name should match the name in config.esriaddinx, the XML file that I ask you to pay attention.

Then, click on the button "Customize...", on Toolbars tab, you can create a new toolbar to hold our work. I created one called "Monkeybar", you can name it whatever, just make sure it won't duplicate the existing ones in ArcMap. Switch to Commands tab, the left pane lists all available categories, remember what I told you to remember? The category name is "Add-In Controls"! If you click on it, on the right side, you should be able to see "My Button" as the command. 

Now you can drag "My Button" onto your new toolbar! Next step will be adding some layers onto the map. The quickest way to do it is by adding a basemap which will bring in the coordinate system as well. I chose ESRI World Topo. It has a web Mercator type of projection. You can find my mxd file  [here](https://github.com/hellocomrade/arcobject/blob/master/lesson1/lesson1.mxd) 

Now if you click the button (By default, it is with a blue round icon), you should be able to see four red round dots showing on the four corners of the map. You may need to zoom to the full extent to see them. If you put a break point at line [59](https://github.com/hellocomrade/ArcObject/blob/master/lesson1/Button1.cs#L59) of Button1.cs and click our button in ArcMap, the execution will be suspended and you can do step by step debug on Button1.cs source inside VS2013.

You may notice that most of logics of Button1.cs are outside OnClick function. What if you want to debug the code, say, at line [33](https://github.com/hellocomrade/ArcObject/blob/master/lesson1/Button1.cs#L33)? Well, that function is called "the default contructor of the class" and only invoked once when this button is clicked the every first time. In order to debug the codes in there, we will have to termintate current debug session by clicking the red square button in VS 2013, then put a new breakpoint at line [33](https://github.com/hellocomrade/ArcObject/blob/master/lesson1/Button1.cs#L33) and start over again.

BTW, can anyone tell me what the line [36](https://github.com/hellocomrade/ArcObject/blob/master/lesson1/Button1.cs#L36) means? What is the section on the right side of += sign for? If you don't know, it is time to go over C# before you move onto future lessons.

## Lesson 2: Get the Heavy Work Done Outside ArcMap ##
When Arcpy was released, one of the exciting things is that GISers are allowed to get the data processing work done without having ArcMap opening. AO allows you to do so from day one, you just need a little bit more programming effort:)

In this session, we are going to develop a Windows Command Line application that is able to run without ArcMap involved (This is not completely true: if you don't have ArcEngine license, you still have to have ArcMap installed on the system in order to gain the license access). Anyway, let's close eyes and pretent ArcMap is off the radar here.

In order to make a fun case, I decided to play with some real data from [Great Lakes Restoration Database](http://habitat.glc.org/). Through its [Map Interface](http://habitat.glc.org/glrd/viewer.php), you can download all GLRD projects in csv format. We will take this dump as the source data. Each row of this file contains a project record with lat/lon, which we can convert it into a point feature class. If you are a GISer, you probably have done this hundreds of times, you may ask: what the fun part of that? Allright, let's add some fun: what if I only want to extract the project for the state of Michigan? Well, if you still have trouble to find the place to download this csv file, please click [here](http://habitat.glc.org/glrd/glc-search.php?download=1&all=1).

Let's go back to VS2013. FILE -> Project, on the left pane inside New Project window: Installed -> Templates -> Visual C# -> ArcGIS, if you click on "Extending ArcObjects", the central pane will display all available template, please choose "Console Application (Desktop)". You should see "ArcGIS Project Wizard" showing up, on the first page, we will keep everything as default and click "Next". Then, you will need to choose the license type for your ESRI products so the program knows what components can be built into your program. I chose "Advanced" then click Finish to get the template loaded.

If you open "Solution Explorer", you should see two cs files created by ESRI. Please replace "Program.cs" with [this](https://github.com/hellocomrade/ArcObject/blob/master/lesson2/Program.cs). Again, before you build the program, you will have to add some extra refereces. Remebmer the 1st step of "ArcGIS Project Wizard"? I convinced you to skip it, you actually could load extra references there.

1.ESRI.ArcGIS.Version;
2.ESRI.ArcGIS.Geodatabase;
3.ESRI.ArcGIS.Geometry

Now, you should be able to build the program. This tool is supposed to run under command line with two arguments:

C:\DesktopConsoleApplication1.exe glrd.csv michigan

The first argument is the path to the csv file you just downloaded, the second one is the State name. If you fail to feed these two arguments, the program will terminate immediately. Then, you may ask: how could I set this up insdie VS2013? In Solution Explorer, right click on the project, then go to "properties", under Debug, put the arguments inside "Command line arguments". By doing so, everytime you debug the program, the stuff you put there will be fed to it.

This is a "standalone" app and we will put our focus on the [code](https://github.com/hellocomrade/ArcObject/blob/master/lesson2/Program.cs). There are two tasks done in the code:

1. Parse an Excel csv file using SQL query;
2. Create a file Geodatabase and populate it with point geometry and attributes;
 
The first task is necessary if we have to filter the content first. For the case I set up, we'd like to find out only the projects in a single Great Lakes State. We achieved this at Line [33](https://github.com/hellocomrade/ArcObject/blob/master/lesson2/Program.cs#L33) by using OleDbCommand against Excel. Microsoft offers a very neat approach to manipluate Excel data using SQL statement. I highly recommend it to your programming work.

If we find positive number of projects for a state, we can go ahead to put them inside a geodatabase. Please pay attention on the flow of this process:

1. Get the factory that is able to create file-based geodatabase, see line [135](https://github.com/hellocomrade/ArcObject/blob/master/lesson2/Program.cs#L135).  
2. Create new feature class at line [147](https://github.com/hellocomrade/ArcObject/blob/master/lesson2/Program.cs#L147).
3. Loop the project records, create new feature and then insert them into the feature class through FeatureBuffer and FeatureCursor,
   which are recommended for bulk insertion. See line [156](https://github.com/hellocomrade/ArcObject/blob/master/lesson2/Program.cs#L156). Once the loop is done, flush the curosr to write all records back to disk.

During the feature class creation, you will have to define:

1. GeometryType;
2. Spatial Reference;
3. Fields as attributes;

This is a pretty standard procedure. I am surprised that ESRI doesn't have a snippet or code sample for this. I only find [this](https://github.com/Esri/developer-support/blob/master/arcobjects-net/display-XY-data-from-CSV/DisplayXYData.cs) at ESRI github account. Be aware that sample uses "esriDataSourcesFile.TextFileWorkspaceFactory", which freed the programmer from dealing with csv file directly. However, the freedom always comes from the understanding of the system. With the approach I introduced, you now have more flexibility to deal with Excel file.

Before we wrap up, let's go back the default template ESRI offers to us:
```c#
//ESRI License Initializer generated code.
m_AOLicenseInitializer.InitializeApplication(new esriLicenseProductCode[] { esriLicenseProductCode.esriLicenseProductCodeAdvanced },
                                        new esriLicenseExtensionCode[] { esriLicenseExtensionCode.esriLicenseExtensionCodeNetwork,                                           esriLicenseExtensionCode.esriLicenseExtensionCodeSpatialAnalyst });
                    ....................
//ESRI License Initializer generated code.
//Do not make any call to ArcObjects after ShutDownApplication()
m_AOLicenseInitializer.ShutdownApplication();
```
These are the routines for initializing ESRI components including licenese, then shut everything down at the end. Your code that involves ESRI stuff should be written in between.

Before we dismiss, could you please take a look on the signature of function [ParseCSV](https://github.com/hellocomrade/ArcObject/blob/master/lesson2/Program.cs#L22), what does List&lt;dynamic&gt; mean? If you read the code carefully, what's the type of the stuff we throw into the List at line [38](https://github.com/hellocomrade/ArcObject/blob/master/lesson2/Program.cs#L22)? Again, if you have no idea, your C# knowloedge needs to be updated!

OK, we are all done. Until next time, may the force be with you!

## Lesson 3: ArcObjects or ArcPy, To be or Not to be ##

First of all, I don't intend to create a holy war here. Both of them are great! I only want to illustrate the difference between AO and AP under a particular case. Hopefully, no matter you are an AO or AP supporter, this could be helpful for you to pick the right tool for the right task.

Let's describe the task first: if you are familiar with Graph as a data structure in the context of computer science, you may know it's a foundmental structure to conduct any kind of operations, such as network tracing, shortest path, maximum flow, etc. Given a prepocessed dataset [NHD Plus](http://www.horizon-systems.com/nhdplus/) for Great Lakes in file geodatabase format, we'd like to create the graph in its [adjancent list](https://en.wikipedia.org/wiki/Adjacency_list) form. By examining the geodatabase carefully, we found there are two feature classes that are particularly useful for this task: NHD_Flowline and Hydro_Net_Junctions. As you may know, NHD Plus is a dataset that was processed and built with a Stream Network using ESRI technology. However, ESRI geodatabase is proprietary so we can't take advantage of existing data to build our adjacency list (I actually exaggreate a bit here coz ESRI does provide an open source C++ library for manipulating file based geodatabase).

By using these two feature classes, we could get our work done using ArcPy in a pretty straightforward way. Here is the [code](https://github.com/hellocomrade/arcpy_nhd_tracing/blob/master/adj.py). You may notice that script used shp file instead of geodatabase. I didn't lie, I do have a version using Geodatabase [here](https://github.com/GreatLakesCommission/brcs-models/blob/master/test/arcpy/trajectory_db.py). If you work for my employer, you should be able to view it. If you set this script up on your desktop with ArcMap 10.3 installed and NHD Plus for Great Lakes ready (let me know if you need this 'NHDPlusV21_GL_04.gdb' used in the test), you may find it could take you up to 6 to 12 hours to complete. I bet you would also notice the bottleneck is [here](https://github.com/hellocomrade/arcpy_nhd_tracing/blob/master/adj.py#L39) and [here](https://github.com/hellocomrade/arcpy_nhd_tracing/blob/master/adj.py#L56). 

Can we make some improvement there? Unfortunatelly, there is really not much we could do since ESRI only exposed limited functionalities to ArcPy, mainly Geoprocessing using whatever inside the toolbox of ArcMap. In other words, all your ArcPy scripts do one thing: a combination of whatever number of tools ESRI offers in ArcMap. You do have 64-bit capacity but not much help for this case, which is more likely to be CPU bound.

Let's flip the coin and check the other side: ArcObjects .Net, this evil successor of ArcObject for Visual Basic provides a thin layer wrapper on top of ArcObjects COMs and supposed to be able to access every aspect of ESRI technologies, well, if you have proper license purchased. In order to have a fair play, we will stick with the same methodology that ArcPy takes. We believe we could have a better performed C# script using AO based upon the fact AO offers a finer granularity than AP in terms of programming. By breaking those two bottlenecks and go a layer deeper inside AO, we may make a difference.

First thing first, let's get the boring part explained first: Here are two POCOs to represent the node and its succssors:

```c#
    sealed class DownstreamNode
    {
        private int incomingEdgeId;
        
        public int IncomingEdgeId
        {
            get { return incomingEdgeId; }
            //set { incomingEdgeId = value; }
        }
        private int id;

        public int Id
        {
            get { return id; }
            //set { id = value; }
        }

        private IEnumerable<double> velocities = null;

        public IEnumerable<double> Velocities
        {
            get { return velocities; }
            //set { velocities = value; }
        }
        public DownstreamNode(int id, int eid, IEnumerable<double> vecs)
        {
            this.id = id;
            this.incomingEdgeId = eid;
            this.velocities = vecs;
        }

    }
    sealed class Vertex
    {
        private int id;

        public int Id
        {
            get { return id; }
            //set { id = value; }
        }
        private int incomingStreamCount;

        public int IncomingStreamCount
        {
            get { return incomingStreamCount; }
            //set { incomingStreamCount = value; }
        }
        List<DownstreamNode> downstreams;

        internal List<DownstreamNode> Downstreams
        {
            get { return downstreams; }
            //set { downstreams = value; }
        }
        public Vertex(int id, int incomingCnt, List<DownstreamNode> ds)
        {
            this.id = id;
            this.incomingStreamCount = incomingCnt;
            this.downstreams = ds;
        }
    }
```

Vertex as the node is used to track the node Id, the number of incoming streams and most important: the downstream nodes stored in a List. DownstreamNode class contains node ID, incoming stream ID and most important: the velocity data in a list (our original thought is to use these velocity to calculate travel time along streams).

In the C# script, we need to open the file-based geodatabase and get the references to feature classes we'd like to examine:

```c#
    Type factoryType = Type.GetTypeFromProgID("esriDataSourcesGDB.FileGDBWorkspaceFactory");
    IWorkspaceFactory workspaceFactory = (IWorkspaceFactory)Activator.CreateInstance(factoryType);
    IWorkspace workspace = workspaceFactory.OpenFromFile(@"C:\Users\hellocomrade\NHDPlusV21_GL_04.gdb", 0);
    IFeatureWorkspace iFtrWs = workspace as IFeatureWorkspace;
    IFeatureClass fcLine = iFtrWs.OpenFeatureClass("NHD_Flowline");
    IFeatureClass fcPoint = iFtrWs.OpenFeatureClass("Hydro_Net_Junctions");
```

Since we now have feature classes in hand, we are ready to tackle the spatial queries, which ususally done in ArcPy by three lines of code:

```python
    arcpy.MakeFeatureLayer_management("junctionpoint.shp",VertexLyr,'"FID"={0}'.format(row[0]))
    arcpy.MakeFeatureLayer_management("flowline.shp",FlowlineLyr)
    arcpy.SelectLayerByLocation_management(FlowlineLyr,"INTERSECT",VertexLyr)
```

The third line "arcpy.SelectLayerByLocation_management" is the key. Since we are going to have to loop through every junction point against NHD_Flowline layer (one point against all polylines), we suspect "SelectLayerByLocation_management" may be inefficient due to the design. "SelectLayerByLocation_management" is probably programmed this way:

1. Build spatial index on NHD_Flowline;
2. Search the index using given point and find all candidates residing on a sub-tree of polylines;
3. Linear scanning all candidates against given point using geometric operators to rule out false positive;
4. Return resultset;

It's possible the first step, the index, could be rebuilt every time we invoke "arcpy.SelectLayerByLocation_management", which is a waste. What if we build it once and use the same index for all points? I was told that AO spatial query should be done using:

```c#
    featureClassRef.search(spatialQueryFilter, null);
```

I am not sure how it was implemented, but it really looks close to "arcpy.SelectLayerByLocation_management". So, I decide to go a different route: "IFeatureIndex" and "IIndexQuery2", which seems promising by giving us finer scale control. Here is my code:

```c#
   IGeoDataset geoLineDS = (IGeoDataset)fcLine;
   ISpatialReference srLine = geoLineDS.SpatialReference;
   IFeatureIndex lineIdx = new FeatureIndexClass();
   lineIdx.FeatureClass = fcLine;
   lineIdx.Index(null, geoLineDS.Extent);
   IIndexQuery2 queryLineByIdx = lineIdx as IIndexQuery2;
```

We create a FeatureIndex coclass and ask it to create the spatial index against the given feature class (it appears that FeatureIndex actually creates index on the file system, I thought it could take advantage of the index inside geodatabase.) and then we cast it to IIndexQuery2, which has a method called: [IntersectedFeatures](http://resources.arcgis.com/EN/HELP/ARCOBJECTS-NET/COMPONENTHELP/index.html#/IntersectedFeatures_Method/0012000006rz000000/). If we are right, this will perform the above steps from 2 to 4.

We can easily put codes together as an ESRI console application using C#, well if you followed my previous tutorials. One thing that is worth to mention is to calculate the geodesic length of a polyline with lon/lat coordinates. In ArcPy, again, only needs one line of code:

```python
cursor_row_item.getLength('Geodesic','Feet')
```

You may think it's supposed to be this simple. Well, hold your thought, see how this was done in AO, given polyLine is a IPolyline:

```c#
    double lineLen = 0.0;
    IGeometryServer2 geoOperator = new GeometryServerClass();
    ILinearUnit unit = new LinearUnitClass();
    PolylineArray tmpArr = new PolylineArrayClass();
    tmpArr.Add(polyLine);
    IDoubleArray lengths = geoOperator.GetLengthsGeodesic(srLine, tmpArr as IPolylineArray, unit);
    if (0 != lengths.Count)
        lineLen = lengths.get_Element(0) * 3.28084;
```
Sorry, don't ask me how come a default LinearUnitClass instance could work here. It's messy enough. I held my impluse to put my own Haversine code there coz I trust ESRI has a better solution near North/South Pole...You got to believe it! :) Again, I was told by an AO guru that this geodesic length on polyline thing should be done on the segments level using a differnt method. I didn't dig that much: I actually intentionally made my C# code a bit messy, I even tried to use Linq but C#'s Linq is pale comparing with Python's list comprehension. Because I know: C# on CLR against COM on Windows platform is definitely going to be faster than Python, not to mention ArcPy is on top of wrapper's wrapper of wrappers...

If you could compile the [code](https://github.com/hellocomrade/ArcObject/blob/master/lesson3/Program.cs) successfully, don't be so rush to run it, make a cup of coffee and have your favorite book in hands, the last ArcPy script took hours! Well, on my laptop, I did have time have a cup of coffee and read some news online, however, surprisingly, This AO version was completed in 15 mintues! My math is bad, can someone tell me how many times this one is faster than the AP version?

Again, I don't mean to prove AO is better than AP or convince you to switch to AO. I'd like to give everyone an opportunity to update your knowledgebase and think outside of the box! It's possible that you lose the track of the sense of performance while you are enjoying the fewer lines of code of ArcPy... I like ArcPy, I really do, but in a lot of scenarios, I feel like I was wearing a T-shrit for a teenager. So, my advice is: if you do care the performance or you sense ArcPy has been too slow for you, you may consider using ArcObject, which usually won't disappoint you, at least not this time for our testing case. :)

See you next time!


## Lesson 4: Inside ArcPy ##

I know, I kown, we are supposed to discuss ArcObject, not ArcPy. However, previously, on "ArcObject .Net with VS 2013"...(all right, we are not a TV series:), I was shocked by the performance gain by swithcing from ArcPy to ArcObject. I thought it might be worth doing some investigation on ArcPy, its implementation and its capability. If you don't plan to learn ArcPy, please ignore this chapter.

ArcPy was first introduced in 2010 with the realse of ArcGIS 10 (well, it's really jsut 9.4, but naming it 10 made it sound newer...). In fact, there were "arcgisscripting" back to ArcGIS 9.2, it supported not only Python, but other scripting languages, such as VBScript, Perl, as well. However, soon after Microsoft stopped support on VB, VBScript, Jscript, ESRI made the dicision to drop all scripts but Python. It was meant to provide scripting ability on geoprocessing tasks only. Geoprocessing toolbox was debuted in ArcMap 9.2. It was designed to simplify certain data processing works:

1. Analysis toolbox
2. Cartography toolbox
3. Conversion toolbox
4. Data Management toolbox
5. Editing toolbox
6. Geocoding toolbox
7. Linear Referencing toolbox
8. Multidimension toolbox
9. Spatial Statistics toolbox
 
In ArcPy, every geoprocessing function in ArcMap is defined as a function inside ArcPy modules. You may generate the contex hull of a point feature class by the following two lines of code:

```python
arcpy.env.workspace="C:\Users\hellocomrade\Documents\ArcGIS\GIS Data"
arcpy.MinimumBoundingGeometry_management("Marinas.shp","Marinas_CH.shp","CONVEX_HULL","ALL")
```

The function "MinimumBoundingGeometry_management" is equivalent to the tool of "Minimum Bounding Geometry", which can be found ArcMap-> Toolbox -> Data Management. ArcPy simply automates this tool by allowing user to send parameters of this tool in a programatic manner. After a second look, you may notice all functions follow the samae [naming convention](http://resources.arcgis.com/en/help/main/10.2/index.html#/Understanding_tool_syntax/002t00000011000000/):

toolname + '_' + toolbox alias

If you are not sure the alias of a particular toolbox, right click on the toolbox then property.

According to ESRI blog, here is the purpose of ArcPy:

"ArcPy is a site-package that builds on (and is a successor to) the successful arcgisscripting module. Its goal is to create the cornerstone for a useful and productive way to perform geographic data analysis, data conversion, data management, and map automation with Python."

In the first edition of ArcPy, other than geoprocessing, it also provides couple other modulese: Mapping (arcpy.mapping), Spatial Analysis (arcpy.sa) and Geostatistical Analyst (arcpy.ga). In 10.1, Data Access (arcpy.da) and Time (arcpy.time) were introduced. It also includes some other libraries/utilites.

As we all know, all ESRI production customization can be done through ArcObject and ArcObject was developed based upon Microsoft COM against Windows system. For example, the means to excute geoprocessing tools is through GPDispatch coclass ([coclass definition](https://msdn.microsoft.com/en-us/library/24z8966k.aspx)). It only allows user to send in arugments by string and consume whatever result the interface returns. Therefore, ESRI call it a "coarse-grained object", see [here](http://edndoc.esri.com/arcobjects/9.2/ComponentHelp/esriGeoprocessing/GpDispatch.htm). ArcPy, however, implements its geoprocessing functionalities on top of this interface. As some of the ESRI document indicates, ArcPy is a coarser-grained model as well.

"Arcpy.mapping was built for the professional GIS analyst (as well as for developers). Traditionally, the scenarios listed above had to be done using ArcObjects and it often proved to be a very difficult programming environment to learn for the average GIS professional. Arcpy.mapping is a courser-grained object model, meaning that the functions are designed in a way that a single arcpy.mapping function can replace many lines of ArcObjects code."

Therefore, you could easliy get some amazing thing done through ArcPy in only couple lines of code:

```python
mxd = arcpy.mapping.MapDocument("C:/Project/Watersheds.mxd")  
arcpy.mapping.ExportToPDF(mxd, "C:/Project/Output/Watersheds.pdf")
```

The above codes will render all features in the mxd, using the configuration defined in the mxd, to a pdf file.

You may want to ask: should we consider ArcPy as a replacement of ArcObject? Still, according to ESRI:

"Arcpy.mapping is not a replacement for ArcObjects but rather an alternative for the different scenarios it supports. ArcObjects is still necessary for finer-grain development and application customization, whereas arcpy.mapping is intended for automating the contents of existing map documents and layer file."

You may also want to know: if ArcPy is able to call ArcObject directly? The answer is NO. But, I am sure, ArcPy is still somehow communicate with ArcObject indirectly, implicitly. As always, "curiosity killed the cat". I am going to open the hood of ArcPy a little bit and see what's in there. On my desktop, ArcPy was installed at:

C:\Program Files (x86)\ArcGIS\Desktop10.2\arcpy\arcpy

In there, you will find a classic python module [package structure](https://docs.python.org/2/tutorial/modules.html#packages). There is a subfolder there called "arcobjects", are you convinced? I am sure you are not. So, let's open some py files. "MapDocument" class is defined in _mapping.py. The declaration of the class is like:

```python
class MapDocument(mixins.MapDocumentMethods,_BaseArcObject):  
```

If you are familiar with C# or Java, you could consider "mixins" as an interface. There is no explicit constructor defined in MapDocument, therefore, according to Python's Method Resolution Order ([MRO](https://www.python.org/download/releases/2.3/mro/), depth-first before 2.3 and C3 algorithm after, you could examine the behavior of MRO by class.mro()), its base classes' constructors will be invoked:

```python
class MapDocumentMethods(object):  
    def __init__(self, mxd):  
        """MapDocument(mxd_path) 
 
           Provides a reference to a map document ( .mxd ) stored on disk or to 
           the map document that is currently loaded within the ArcMap 
           application (using the CURRENT keyword) 
 
             mxd_path(String): 
           A string that includes the full path and file name of an existing map 
           document ( .mxd ) or a string that contains the keyword CURRENT."""  
        assert (os.path.isfile(mxd) or (mxd.lower() == "current")), gp.getIDMessage(89004, "Invalid MXD filename")  
        super(MapDocumentMethods, self).__init__(mxd)
......
```

It appears MapDocumentMethods invokes its base class's __init__ directly if super() actually returns its base class. However, since we are dealing with multiple inheritance here, super(MapDocumentMethods, self).__init__(mxd) actually invokdes the constructor of MapDocument's second base class: "_BaseArcObject". If you'd like to understand more about the fancy behaviors of super, please click [here](https://rhettinger.wordpress.com/2011/05/26/super-considered-super/). Let's take a look on _BaseArcObject:

```python
class _BaseArcObject(object):  
    _arc_object = None  
    def __init__(self, *args, **kwargs):  
        """Wrapper for ArcGIS scripting Arc Objects -- 
           Create a new object instance if no reference is passed in."""  
        super(_BaseArcObject, self).__init__()  
        self._arc_object = gp._gp.CreateObject(self.__class__.__name__,  
                    *((arg._arc_object if hasattr(arg, '_arc_object') else arg)  
                        for arg in args))  
        for attr, value in kwargs.iteritems():  
            setattr(self._arc_object, attr, value)  
        self._go()  
```

It has a constructor that is able to take Variable-length input arguments. It also defines a static member variable, "_arc_object". "gp._gp.CreateObject" function is actually defined in name space "geoprocessing", remember we mentioned that geoprocessing is the first module introduced for ArcPy? For gp, it's an instance of class Geoprocessor:

```python
class Geoprocessor(object):  
    """Represents a geoprocessing object in ArcGIS"""  
    def __init__(self):  
        """Geoprocessor()"""  
        self._gp = arcgisscripting.create(10.0) 
```

Now, we have reached the origin: ArcPy communicate with ArcObject through "arcgisscripting", who is the predecessor of ArcPy. You will not find arcgisscripting inside ArcPy folder, it actually resides at "C:\Program Files (x86)\ArcGIS\Desktop10.2\bin", with the name of "arcgisscripting.pyd", which is actually a DLL.

As we discussed in the previous and this chapter, because of interoperability with COM, you should expect some performance loss if you use ArcPy not ArcObject. How much loss you may get is really depending on the tasks. Here is a benchmark I fuond on Internet against ArcGIS 9.3. Let's hope ESRI is making a better job now.

![benchmark_arcpy](https://github.com/hellocomrade/ArcObject/blob/master/lesson4/20140712032424890.png)

Again, This chapter is meant to help you understand what made of ArcPy. Both ArcObject and ArcPy are great tools for different scenarios. If you'd like to know more about ArcPy, I recommend ESRI's repos on github [here](https://github.com/Esri/solutions-geoprocessing-toolbox). Let me know if you can find any expensive geoprocessing function called inside a huge loop. :)

Enjoy!
