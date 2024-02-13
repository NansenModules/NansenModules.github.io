Most of the data management and organization in Nansen relies on the concepts listed in the sections below. The following image shows a test project with multiple sessions as it appears in the Nansen App. The user can select sessions in the app and run different methods and community toolboxes on their data. By implementing [DataLocationModels](#Data-Locations) and [VariableModels](#Variables), Nansen can run these methods and toolboxes regardless of how the data is saved and organised on the file system.

<img src="https://github.com/ehennestad/ehennestad.github.io/blob/main/images/nansen_annotated2.png?raw=true" alt="Nansen App" width="100%"/>


## Projects
A user creates one or several projects to manage collections of experimental sessions. An experimental session is defined as one continuous recording of data from a single or multiple recording systems. A project should ideally consist of a fairly homogeneous set of sessions, i.e same type of recordings and similar protocols used, but in the end it is up to the user to decide how to organize a project. To manage projects, run `nansen.ProjectManager`

The name of the project will be used to create a namespace (or [package folder](https://www.mathworks.com/help/matlab/matlab_oop/scoping-classes-with-packages.html)) in Matlab, so it is required that the name is a valid variable name (i.e consist of letters and underscores) and recommended that it is short and concise. 

Users can create their own table variables and session methods (more on this below) and these will be added to the project's namespace. In this way, session methods and table variables can be adapted to meet project-specific demands, while also retaining the same names across different projects.

## Data Locations
A typical experiment might include data from different recording systems as well as different stages of processing. Each of these constitutes a **Data Location**. For example, when doing two-photon experiments, one computer is typically dedicated to the image acquisition whereas another computer is controlling the behavioural setup and recording behavior-related variables. These would be represented as different data locations, i.e "Rawdata_twophoton" and "Rawdata_behavior". 

In Nansen, the user will use the `nansen.setup` application to create a DataLocationModel for each project. The DataLocationModel is configured so that Nansen can locate folders containing session data (session folders) from all the different Data Locations and match them to specific sessions. This includes specifying the root directory where data is located, as well as the subfolder organization of data. Once a DataLocationModel is properly configured, all the session folders from different Data Locations are accessible from a `Session` object (more about session objects below).

## Variables
When working with Nansen, a key principle is that the user should not have to interact directly with files, but instead with data variables. A very short example workflow could look like this:

```
twoPhotonImageStack = mySession.loadData('TwoPhotonSeries')
motionCorrectedStack = nansen.wrapper.normcorre.Processor(twoPhotonImageStack)
```
(Note: To run motion correction in Nansen, it is better to run it from the menu in the Nansen app)

In order to make this work the user has to create a VariableModel. This is initially done during the initial project setup, but the VariableModel can be updated interactively at any time. The VariableModel maps a file (or a set of files) to a data variable and contains instructions on how to read and write data from the file. For the example above to work, there need to be a variable called "TwoPhotonSeries" and the model needs to know where to find that variable and how to read it. This is achieved by linking the variable to a file by specifying a pattern which should uniquely identify the file containing two-photon data and by specifying a **file adapter** 
to use for reading the data. 

Nansen provides file adapters for some common microscope formats (PrairieView, SciScan, ScanImage, more to come) and therefore it does not matter how the data is stored on the drive. ScanImage saves data to tiff stacks, SciScan saves data to binary files, PrairieView saves data to individual tiff files, etc, and the job of the file adapter's is to load the data into a standard data type that can be passed on to a processing method.

In practice, this means that opening a TwoPhotonSeries for any session, regardless of which recording system is used, the following command is used: `twoPhotonImageStack = mySession.loadData('TwoPhotonSeries')`.

Some data variables are predefined in Nansen, e.g variables related to two-photon data, rois and extracted fluorescence series, but the user can easily add any experimental files as custom **Variables**. Custom file adapters (m-files) can be dragged and dropped into Nansen, and a hopeful scenario is that users can share file adapters and thereby spend less time writing custom code to load data into their specific pipelines, but instead write session methods for working on standardised data variables.

## Session table and `Session` objects
Once a DataLocationModel is created, Nansen can detect folders with experimental data from sessions and create a table of sessions. Each session is represented using a `Session` object. All the variables that are specified in the VariableModel is then accessible on a `Data` property of the session object. 

In the nansen app, all sessions are listed in a session table, where the user can sort sessions or filter sessions according to different criteria. Sessions can also be added to collections, which is useful when working with a subset of sessions. The user can also create custom table (or session info) variables which are added to the table. These **table variables** can either get their values from manual inputs, or the user can write custom functions for retrieving values that are session specific. Furthermore, custom table variables (m-files) can easily be shared with others and included to projects.

Also, the session table provides an interface to running processing / analysis methods on sessions and building processing pipelines. Nansen comes with many methods for processing two photon data, and these can be combined into custom pipelines. The user can select sessions in the app and run methods or pipelines for these sessions. The user can also create custom **session methods** which is instantly integrated into the app and can be shared across projects and with other users.
