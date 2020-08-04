<seo metak="sholl,sholl analysis, plugin,arbor,neuron,morphometry,dendrite, neuroanatomy" metad="sholl,sholl analysis, plugin,arbor,neuron,morphometry,dendrite,neuroanatomy" />
<div style="float:right;">
{{ComponentStats:ca.mcgill:Sholl_Analysis}}
<!-- Create a short TOC next to main Contents -->
<div style="padding-top:17em;padding-left:1.5em;">
<div style="font-size:175%;border-bottom:1px dotted #ccc;margin-bottom:0.55em;">Quick links</div>

;Analysis of traced arbors?:Jump to [[#Analysis of Traced Cells|Traced Cells]] or [[#external-traces|Other tracing software]]
;Segmentation issues?:Jump to [[#Cf. Segmentation|Cf. Segmentation]] or [[#Pre-processing|Pre-processing]]
;Having Problems?:Jump to [[#FAQ|FAQs]]
</div>
</div>{{TOC}}
Automated and multithreaded Sholl for direct analysis of fluorescent images and traced morphologies. Features powerful quantifications based on curve fitting. Analysis of data obtained outside of ImageJ is also possible.

== Introduction ==
<span id="CA1CellMask"></span>[[ File:BitmapSholl-CA1mask.png|frame|[[Skeletonize3D|Skeletonized]] hippocampal CA1 cell<ref name="Gross"></ref> (juvenile mouse) in which apical and basal dendrites have been analyzed [[#CA1CellPlot|separately]] and [[#Output_Options|color coded]] according to their Sholl profile. Warmer hues indicate higher number of Intersections (''N''). [[#CriticalRadius|Critical radius]] (''r<sub>c</sub>'') and [[#MeanValueOfFunction|Mean value]] (''N<sub>av</sub>'') are indicated.]]

The Sholl technique<ref name="Sholl"></ref> is used to describe neuronal arbors. This plugin can perform Sholl directly on 2D and 3D grayscale images of isolated neurons. Its internal algorithm to collect data is based upon how Sholl analysis is done by hand — it creates a series of concentric ''shells'' (circles or spheres) around the focus of a neuronal arbor, and counts how many times connected voxels defining the arbor intersect the sampling shells.
The major advantages of this plugin over other implementations are:
* When analyzing images directly, it does not require previous tracing of the arbor (although it can also analyze [[#Analysis of Traced Cells|traced arbors]])
* It combines [[#MethodsTable|curve fitting]] with several [[#Sholl Plots|methods]] to automatically retrieve [[#Metrics|quantitative descriptors]] from sampled data, which allows direct statistical comparisons between arbors
* It allows [[#Multiple Samples and Noise Reduction|continuous and repeated sampling]] around user-defined foci
* It allows [[#Batch Processing|batch processing]]

== Installation ==
The plugin is distributed with Fiji. It installs several commands under {{bc|color=white|Analysis|Sholl|&nbsp;}}. '''As part of an active effort to [[2015-12-22_-_The_road_to_Java_8|modernize ImageJ]] you need to [[How_to_follow_a_3rd_party_update_site#Add_update_sites | subscribe]] to the Java 8 update site to access the latest plugin version''' (this will also allow you to access the newest [[2015-12-22_-_The_road_to_Java_8#Components_which_have_already_migrated|ImageJ capabilities]]). To do so, you can either:
* [[Downloads|Download the latest Fiji release]]. Newer releases come pre-bundled with Java 8, and are already subscribed to the [[User:Java-8|Java-8 update site]].
* If you have downloaded Fiji while ago and want to keep your existing installation, you will have to download Java 8 and make your [[Troubleshooting#Checking_the_Java_version|Fiji installation aware of it]]. Then subscribe to the [[User:Java-8|Java-8 update site]].

== Direct Analysis of Images ==
In this mode (bitmap analysis), the plugin requires a [[#faq:threshold|binary image or a segmented]] [[#faq:image-types|grayscale image]] (2D or 3D) containing a single neuron.

# Segment the neuronal arbor using {{bc|color=white|Image|Adjust|Threshold...}} (shortcut: <span style="display:inline-block;">{{key press|Shift|T}} </span>).
::N.B.: When using multichannel images, you will have to set the its display mode to ''Grayscale'' using {{bc|color=white|Image|Color|Channels Tool...}} ({{key press|Shift|Z}}), because images displayed as ''Composites'' cannot be thresholded.
# Define the center of analysis using a valid [[#Startup_ROI|startup ROI]].
# Run {{bc|color=white|Analysis|Sholl|Sholl Analysis...}}, adjusting the default [[#Parameters|Parameters]] in the dialog prompt.
# Press ''More » [[#Cf._Segmentation|Cf. Segmentation]]'' to visually inspect the two thresholded phases: ''arbor'' and ''background''.
# Problems? Read the  [[#FAQ|FAQs]].
{{Tip
| id = Paper
| tip = This program is described in [http://www.nature.com/nmeth/journal/v11/n10/full/nmeth.3125.html Nature methods]. The manuscript uses ''Sholl Analysis'' to describe and classify morphologically challenging cells and is accompanied by a [http://www.nature.com/nmeth/journal/v11/n10/extref/nmeth.3125-S1.pdf Supplementary Note] that presents the software in greater detail. Please [[#faq:citing|cite it]] when acknowledging the plugin in your published research.}}
=== Startup ROI ===
The center of analysis can be specified using one of three possibilities:
;Straight line: A Straight line from the focus of the arbor to its most distal point using the Straight Line Tool. The advantages of using line selections are twofold: 1) Center of analysis and [[#EndRadius|Ending radius]] are automatically set, and 2) Horizontal/vertical lines (created by holding {{key press|Shift}} while using the Straight Line Selection Tool) can be used to [[#Restrict|restrict analysis to sub-regions]] of the image.
;Single point: A single point marking the focus of the arbor using the Point Selection Tool. With single point selections, only the center of analysis is defined. Thus, this option is suitable for [[#Batch_Processing|batch processing]] of images with different dimensions with undefined [[#EndRadius|Ending radius]].
;Multi-point selection:A Multi-point selection (multi-point counter) in which the first point marks the center of analysis while the remaining points mark (count) the number of primary branches required for the calculation of [[#SchoenenSampled|ramification indices]]). Suitable for cases in which [[#PrimaryBranches|inference from starting radius]] is not effective.
[[File:ShollAnalysisStartupROIs.png|frame|center|Three types of ROIs expected by the plugin when analyzing images directly. Left: Line defining center of analysis (focal point), hemisphere restriction and ending radius. Middle: Single point defining center of analysis. Right: Multi-point selection in which the first point defines the focal point while the remaining points (2 to 5) serve as counters for primary dendrites.]]

=== Cf. Segmentation ===
Press ''More» Cf. Segmentation'' to visually confirm which phase of the segmented image will be sampled. This command highlights foreground from background pixels and is particularly useful when analyzing black and white (binary) images or when using the ''B&W'' lookup table in the Threshold Widget (<span style="border-bottom:1px dotted #ccc;">Image▷ Adjust▷ Threshold...</span> {{key press|Shift|T}}). ''Cf. Segmentation'' allows you to ensure that you are measuring neuronal processes and not the interstitial spaces between them. Here is an example using an axonal arbor of a Drosophila olfactory neuron from the [http://diademchallenge.org DIADEM] dataset<ref name="DIADEM"/>:
{|
|<center><span style="display:inline-block;text-align:center;width:230px">Segmented image</span> <span style="display:inline-block;text-align:center;width:230px">''Cf. Segmentation output''</span> <span style="display:inline-block;text-align:center;width:230px">''Intersections mask''</span></center>
|-
|<center>[[File:CfSegmentation.png|700px|center|]]</center>
|-
|<center>'''Top row:''' Image properly segmented: Arbor is sampled. '''Bottom row:'''  Aberrant segmentation (inverted image): Background is sampled.</center>
Note the reversal of ''[[#Cf._Segmentation|Cf. Segmentation]] output'' and how the ''[[#Output_Options|intersections mask]]'' no longer decorates the axonal processes but the interstitial spaces between them. The consequences of the phase inversion are twofold:  1) the program oversamples (cf. hue ramps on upper left of ''Intersections mask'') and 2) the program detects artifacts induced by the edges of the image (cf. top-right and bottom-right corners of mask where intersections are sampled in the absence of real axons at those locations). Also, note that the initial black and white image would ''look the same'' under an inverted lookup table  ({{bc|color=white|Image|Lookup Tables|Invert LUT}}).
|}
{{Tip
| tip = With binary images, ''Sholl Analysis'' treats  zero intensities as the background, independently of the image lookup table or the state of the ''Black background option'' in <span style="border-bottom:1px dotted #ccc;">Process▷ Binary▷ Options...</span> As with any other [https://imagej.net/docs/guide/146-29.html#infobox:blackBackground ImageJ routine], confusing background with foreground pixels will lead to aberrant results, including: 1) overestimation of branches and 2) artifacts at distances intersecting the boundaries of the image canvas.}}
<span id="Traces"></span>
== Analysis of Traced Cells ==
[[ File:ShollTracingsPrompt.png|400px|right |Main prompt (version 3.6.8), when input is traced data ({{bc|color=white|Analysis|Sholl|Sholl Analysis (Tracings)...}})]]In this mode, the plugin analyzes reconstructed arbors. This is particularly relevant for stainings that do not allow single-cell resolution. The plugin is macro recordable and [[#Batch Processing|batch processing]] is also possible.
# Run {{bc|color=white|Analysis|Sholl|Sholl Analysis (Tracings)...}} and specify a tracing file (a <code>.swc</code>, a <code>.eswc</code> or a [[Simple Neurite Tracer]] <code>.traces</code> file). If you want, you can also specify the image associated with the reconstruction. This will allow the plugin to use the image's metadata to determine spatial units and x,y,z spacing.
# Choose the center of analysis using the drop down menu in the main prompt listing SWC tags (''axon'', ''dendrite'', ''soma'', etc.). Note that if your tracings are not tagged you can do so in [[Simple Neurite Tracer]]
# Adjust the default [[#Parameters|Parameters]]
# Problems? Read the  [[#FAQ|FAQs]]


{{Tip
| id = external-traces
| tip = You can use  {{bc|color=white|Sholl Analysis (Tracings)...}} to analyze reconstructed data from any software capable of [http://www.neuronland.org/NLMorphologyConverter/MorphologyFormats/SWC/Spec.html SWC] export such as [https://sourceforge.net/projects/py3dn/ Py3DN], [http://www.reading.ac.uk/neuromantic/ Neuromantic], [http://research.mssm.edu/cnic/tools-ns.html NeuronStudio], [http://www.neutracing.com neuTube], or [http://www.vaa3d.org/ Vaa3D]), not just [[Simple Neurite Tracer]]. In addition, [http://cng.gmu.edu:8080/Lm/ L-Measure], [http://neuronland.org/NL.html NLMorphologyConverter] or [http://www.neuron.yale.edu/neuron/ Neuron] can also be used to convert several file formats (including proprietary formats from closed-source commercial software such as Neurolucida®, MicroBrightField, Inc.) into SWC.}}
<span id="Importing"></span>

== Analysis of Existing Profiles ==
[[File:BitmapSholl-CA1Compartment.png|frame|right|Linear plot for CA1 cell [[#CA1CellMask|described above]]. Using the soma as center, image was sampled twice using the [[#Restrict|Restrict analysis to hemicircle/hemisphere]] option in order to segregate apical from basal dendrites. For convenience, distances for basal branches were assigned negative values. For clarity, the binary image of the arbor was rotated, scaled and overlaid (in green) over the plot canvas. Note that it is also possible to restrict [[#MethodsTable|curve fitting]] to a sub-range of distances once [[#Importing|data is collected]].]]
This feature is processed by {{bc|color=white|Analysis|Sholl|Sholl Analysis (Existing Profile)...}}. This command can be used to re-analyze data (replot, modify fitting options, etc.) without having to access the initial image or tracing data.  [[#Batch Analysis of Tabular Data|Batch processing]] is also possible. Noteworthy:

* '''Input data''': Any tab or comma delimited text file (.csv, .txt, .xls, .ods) can be used. You can drag & drop these files into the main ImageJ window, import data from the clipboard, or use data from any other table already opened by ImageJ.
* '''Restricting input data''': To restrict measurements to a range of distances ([[#CA1CellPlot|see related example]]), select the range of distances you want analyze. You can click the first row in the range, and then drag the mouse to the last row, or by holding down {{key press|Shift}} while selecting the last row in the range. Then,  in the prompt, activate the ''Restrict analysis to selected rows only'' checkbox.
* '''Calculation of ''Radius step size''''': [[#StepSize|Radius step size]] is calculated from the difference between the first two rows in ''Distance column''. This is mainly relevant when choosing ''Annulus/Spherical shell'' as [[#Normalizer|normalizer]]. 

== Parameters ==
The majority of parameters is shared by all the Analysis commands in the{{bc|color=white|Analysis|Sholl|&nbsp;}} submenu. However, some settings are specific to the type of data used as input: A segmented image, a tracing, or a previously obtained profile. When analyzing images, input values take into account the scale information of the image (which can be set using the {{bc|color=white|Analyze|Set Scale...}} or {{bc|color=white|Image|Properties...}} ({{key press|Shift|P}}), the type of image (2D or 3D), and its [[#Startup ROI|active ROI]].

==== Definition of Shells ====
* <span id="StartRadius"></span>'''Starting radius''' <sup>&nbsp;</sup>The radius of the smallest sampling circle/sphere, i.e., the first distance to be sampled.
* <span id="EndRadius"></span>'''Ending radius''' <sup>&nbsp;</sup>The radius of the largest (last) sampling circle/sphere. It is automatically calculated if a [[#Usage|line ROI is used]]. Note that the specified distance may not be actually sampled, if ''Radius step size'' is not a divisor of ''Ending radius''-''Starting radius''. In this case, the program will choose the largest possible distance smaller than the specified value.<span id="CA1CellPlot"></span>
'''N.B.''' You can clear ''Ending radius'' or set it to ''NaN'' ("Not a Number") to sample the entire image. This is particularly useful when [[#Batch_Processing|batch processing]] images with different dimensions.
* <span id="StepSize"></span>'''Radius step size''' <sup>&nbsp;</sup>The sampling interval between radii of consecutive sampling circles/spheres. This value may be set to zero for continuous (1-voxel increment) measurements.
'''N.B.''' For stacks with anisotropic voxel size, setting ''Radius step size'' to zero, sets the step length to the dimension of the matching isotropic voxel, i.e., the cube root of the product of the voxel dimensions (3D images) or the square root of the product of the pixel dimensions (2D images).

* <span id="Restrict"></span>'''Restrict analysis to hemicircle/hemisphere''' <sup>&nbsp;</sup>This option is only available when an orthogonal radius has been created (by holding {{key|Shift}} while using the <span style="border-bottom:1px dotted #ccc;">Straight Line Selection Tool</span>). It can be used to limit the analysis to sub-compartments of the arbor.
'''N.B.''' For horizontal lines, this option instructs the algorithm to measure intersections at sites equidistant from the center that have y-coordinates above/below the drawn line. For vertical lines, it instructs the plugin to measure intersections at sites equidistant from the center that have x-coordinates to the left/right of the drawn line.
[[ File:BitmapSholl-Prompt_v3.png|320px|right |Main prompt (version 3.4.1), when input is a segmented image ({{bc|color=white|Analysis|Sholl|Sholl Analysis...}})]]

==== Multiple Samples/Noise Reduction ====
* '''Samples per radius ''(2D images only)''''' <sup>&nbsp;</sup>Defines the number of measurements to be performed at each sampling circumference. These measurements are then combined into a single value according to the chosen [[#SamplesIntegration|integration method]]. This strategy, a break from previous approaches, increases the accuracy of non-continuos profiles by diluting out the effect of processes extending tangent to the sampling circumference.
:Visually, this option can be imagined as the "thickness" of the sampling circumference: e.g., for a radius of 100 pixels and a value of 3 ''Samples per radius'', the final number of intersections would integrate the measurements sampled at distances 99, 100 and 101.
[[ File:BitmapSholl-TabularPrompt.png|320px|right |Main prompt (version 3.4.3), when input is tabular data ({{bc|color=white|Analysis|Sholl|Sholl Analysis (Existing Profile)...}})]]
:Note that it would not make sense to increase the number of samples beyond the length (in pixels) of ''Radius step size''. For this reason, this option is limited to a draconian (and arbitrary) maximum of 10 samples.
* <span id="SamplesIntegration"></span>'''Samples integration ''(2D images only)''''' <sup>&nbsp;</sup>The measure of central tendency used to combine intersection counts when multiple ''Samples per radius'' are used. Options are ''Mean'' (the default), ''Median'' or ''Mode''.

* '''Ignore isolated (6-connected) voxels ''(3D images only)''''' <sup>&nbsp;</sup>If checked, single isolated voxels intersecting the surface of sampling spheres are not taken into account, which may allow for smoother profiles on noisy image stacks. However, it should be noted that connectivity in the stack volume may not reflect connectivity on the surface of a digitized sphere. Indeed, in certain contexts, it is possible (though unlikely) to obtain higher intersection counts when this filtering option is active.
:Please keep in mind that this is just a refinement feature, and you should not expect it to mitigate the effects derived from poorly-segmented arbors.

==== Descriptors and Curve Fitting ====
* <span id="EnclosingRadiusCutoff"></span>'''Enclosing radius cutoff''' The number of intersections to be used in the definition of [[#EnclosingRadius|Enclosing radius]].

* <span id="PrimaryBranches"></span>'''Number of primary branches''' The number of primary branches (i.e., those originating directly from cell soma when the center of analysis is the perikaryon) to be used in the calculation of [[#SchoenenSampled|Schoenen ramification indices]]. It is automatically populated using multi-point counts if a multi-point ROI is [[#Startup_ROI|detected at startup]]. Set it to zero (or ''NaN'') to disable calculations of ramification indices.
** '''Infer from starting radius''' If checked, the ''Number of Primary branches'' is inferred from the count of intersections at [[#StartRadius|Starting radius]].

* '''Fit profile and compute descriptors''' <sup>&nbsp;</sup>If checked, data is fitted according to the chosen [[#Choice_of_Methods|method]] and appropriate [[#Metrics|metrics]] calculated automatically. If unchecked, only sampled data is plotted.
** '''Show fitting details''' <sup>&nbsp;</sup>Choose this option to have all of the parameters of the simplex fitting printed to the Log window. The [[wikipedia:Coefficient of determination|coefficient of determination]] (''R<sup>2</sup>'', a measure of goodness of fit) is always stored in the ''Sholl Results'' table even when this option is not selected.
{{Tip
| tip = [[#Complementary_Tools|Complementary Tools]] describes how to extend curve fitting beyond default options.
}}
==== Choice of Methods ====
<span id="Sholl_Methods"></span>The [[#MethodsTable|type of profile(s)]] to be obtained. ''Linear'' (profile without normalization), or normalized profiles: ''Linear-norm'', ''Semi-log'', or ''Log-log''.
* '''Polynomial''' <sup>&nbsp;</sup> Specifies the degree of the [[#MethodsTable|polynomial]] to be fitted to the ''Linear'' profile<ref name="Stulic"></ref>. While the polynomial of best approximation, or "best fit", should be empirically determined for each analyzed cell type, it is possible to ask the plugin to predict the order of the fitting polynomial (or at least try) using the choice ''Best fitting degree''. In this case, the plugin will loop through all the available choices of polynomials, perform each fit in the background and choose the one with the highest coefficient of determination.
* '''Most informative''' <sup>&nbsp;</sup> Select this option when you cannot predict which type of normalized profile best describes the dataset. If chosen, the plugin will use the [[#Dratio|Determination ratio]] to determine which of ''Semi-log'' or ''Log-log'' methods is more appropriate.  ''Linear-norm'' is not performed.<br>The ''Best fitting degree'' and ''Most informative'' choices are obviously more computer-intensive and can be monitored by activating the ''Show fitting details'' checkbox.
* <span id="Normalizer"></span>'''Normalizer''' The property of the sampling shell to be used in the normalization of ''Linear-norm'', ''Semi-log'', and ''Log-log'' profiles. It is [[#MethodsTable|described below]].

==== Output Options ====
[[File:ShollResultAsROIs.png|400px|thumb|right|Intersection points and sampling shells can be retrieved as ROIs using {{bc|color=white|Image|Overlay|To ROI Manager}}. Intersection points are placed at edges of detected clusters of foreground pixels, not their center.]]
* '''Create intersections mask''' <sup>&nbsp;</sup>If checked, a 16/32–bit maximum intensity projection of the analyzed image is generated in which the measured arbor is painted according to its Sholl profile. The type of data (''Raw'', i.e., sampled or ''Fitted'') is displayed in the image subtitle and can be specified in  {{bc|color=white|Analysis|Sholl|Metrics &amp; Options...}} or using the ''Options...'' command in the ''More»'' drop-down menu.<br>NB: The default Lookup Table (LUT) used by the mask can be changed using {{bc|color=white|Image|Lookup Tables|}}. The background color [gray level: 0 (black) to 255 (white)] can also be set in {{bc|color=white|Metrics &amp; Options...}}, or at any later point using {{bc|color=white|Image|Color|Edit Lut...}} WYSIWYG versions (RGB images) of these masks can be otained using by pressing {{key press|Shift|F}} ({{bc|color=white|Image|Overlay|Flatten}}) or by running {{bc|color=white|Analyze|Tools|Calibration Bar...}}
* '''Overlay sampling shells and intersection points'' (2D images only)''''' <sup>&nbsp;</sup>If checked, two sets of ROIS are added to the image overlay: 1) concentric shells matching sampled distances (circular ROIs or composite ROIs when using hemicircles); and 2) Multipoint ROIs at intersection sites between shells and clusters of foreground pixels.
* '''Save results to''' <sup>&nbsp;</sup> If checked, all the results (with the exception of the ''[[#Metrics|Sholl Table]]'') are saved to the specified directory. These include: 1) Sholl plots (saved as PNG images), 2) A table containing detailed data and 3) The Sholl mask. Files are named after the image filename and analysis method. Saving options can be specified in {{bc|color=white|Analysis|Sholl|Metrics &amp; Options...}} (''Options...'' command in the ''More»'' drop-down menu).
** '''Do not display saved files''' If checked, saved files are directly saved to disk and are not displayed. Activate this option when [[#Batch_Processing|batch processing]] files.
{{Tip
| tip = In the dialog prompts of ''Sholl Analysis'', bold headings are clickable URLs pointing to the respective sections of this manual. In addition, relevant tooltips are displayed in the ImageJ status bar when specifying key options.
}}

== Sholl Plots ==
<center>
<span id="ShollPlots"></span>[[File:ShollPlots.png|center|]]
</center>

'''''Linear'', ''Linear-norm'', ''Semi-log'' and ''Log-log'' profiles for the ddaC cell ({{bc|color=white|File|Open Samples|ddaC Neuron}}), version 3.0'''.
Most of the retrieved [[#Metrics_based_on_fitted_data|metrics]] are automatically highlighted by the plugin. ''Linear profile'': [[#MeanValueOfFunction|Mean value]] (horizontal grid line) and [[#Centroid|Centroid]] (colored mark). Logarithmic profiles: The [[#ShollDecay|Sholl regression coefficient]] (also known as Sholl decay) can be retrieved by linear regression using either the full range of data (blue line) or data within percentiles 10–90 (red line).  For this particular cell type, the Semi-log method is more [[#Dratio|informative]] when compared to the Log-log method.

<span id="MethodsTable"></span>
{| style="padding: 60px;" rules="rows"
! align="Center"| &nbsp;
!style="height: 2em;"| Method
! Fit
! Description
|-
| <span id="eq1">(1)</span>&nbsp;&nbsp;
|style="white-space:nowrap"| Linear
|style="white-space:nowrap"|&nbsp;&nbsp; <i>N</i> = a + br + cr<sup>2</sup> + dr<sup>3</sup>  + er<sup>4</sup> + fr<sup>5</sup> + ... + xr<sup>n</sup>&nbsp;&nbsp;
| &nbsp;&nbsp;&nbsp;<br>Outputs a ''N vs Distance'' profile. Data is fitted to a polynomial function<ref name="Stulic"/>. [[#CriticalRadius|Critical radius]], [[#CriticalValue|Critical value]] and [[#MeanValueOfFunction|Mean value of function]] are calculated<br>&nbsp;
|-
| <span id="eq2">(2)</span>&nbsp;&nbsp;
|style="white-space:nowrap"|  Linear-norm
|style="white-space:nowrap"|&nbsp;&nbsp; ''N/S'' = a r<sup>b</sup>
| &nbsp;<br>Outputs a <i>N/S vs Distance</i> profile. Points are fitted to a power function. It is an intermediate representation of the data that can be used to gauge the choice of [[#Normalizer|normalizer]]. Once plotted under a logarithmic scale the Linear-norm curve is similar to the Semi-log profile<br>&nbsp;
|-
| <span id="eq3">(3)</span>&nbsp;&nbsp;
|style="white-space:nowrap"|  Semi-log
|style="white-space:nowrap"|&nbsp;&nbsp; log(''N/S'') = -k r + m
|&nbsp;<br>Outputs a <i>log(N/S) vs Distance</i> profile. A linear regression is fitted to the sampled data. The [[#ShollDecay|Sholl regression coefficient]] (''k'') is calculated<br>&nbsp;
|-
| <span id="eq4">(4)</span>&nbsp;&nbsp;
|style="white-space:nowrap"| Log-log
|style="white-space:nowrap"|&nbsp;&nbsp;  log(''N/S'') = -k × log(r) + m
|&nbsp;<br>Outputs a <i>log(N/S) vs log(Distance)</i> profile. Data is also fitted to a straight line. This is an alternative approach<ref name="Ristanovic"></ref> of obtaining a relevant [[#ShollDecay|regression coefficient]], when the Semi-log method returns a poor fit<br>&nbsp;
|-
|
|}


<span style="display: inline-block; width: 25px">'''''N'''''</span> For 2D images, the <u>N</u>umber of clusters of pixels (8–connected) intersecting the circumference of radius ''r''<br>
<span style="display: inline-block; width: 25px">&nbsp;</span> For 3D images, the <u>N</u>umber of clusters of voxels (26-connected) intersecting the surface of the sphere of radius ''r''

<span style="display: inline-block; width: 25px">'''''r'''''</span> Distance from center of analysis (<u>r</u>adius of Sholl circle/sphere)

<span style="display: inline-block; width: 25px">'''''log'''''</span> Natural logarithm, the logarithm to the base ''e''

<span style="display: inline-block; width: 25px">'''''S'''''</span>The [[#Choice_of_Methods|chosen property ]] of the sampling shell to be used in the normalization of ''Linear-norm'', ''Semi-log'', and ''Log-log'' profiles.<br> <span style="display: inline-block; width: 25px">&nbsp;</span>For 2D images, the ''Perimeter'' of the sampling circumference (2πr) or the ''Area'' of the corresponding circle (πr<sup>2</sup>)<br> <span style="display: inline-block; width: 25px">&nbsp;</span>For 3D images, the ''Surface'' of the sampling sphere (4πr<sup>2</sup>) or its respective ''Volume'' (4/3πr<sup>3</sup>)<br>

'''N.B.''' A third [[#Normalizer|normalization option]] is also available when performing [[#StepSize|non-continuous sampling]]: ''Annulus''/''Spherical shell''. In this case, the normalization is performed against the area of the annulus formed between the circumferences at ''r'' ± ''Radius step size''/2 (2D images), or against the volume between the two spheres at ''r'' ± ''Radius step size''/2 (3D images).<br>

== Metrics ==
Morphometric descriptors and other properties of the arbor are printed to a dedicated table named ''Sholl Results''. Output is fully customizable using {{bc|color=white|Analysis|Sholl|Metrics &amp; Options...}} or using the ''Options...'' command in the ''More»'' drop-down menu. The first columns log analysis parameters: ''Image Directory'', ''filename'' and ''voxel unit'', ''Channel'', ''Lower'' and ''Upper Threshold levels'', ''X,Y'' (in pixels) and ''Z'' (slice number) coordinates of center of analysis, ''Starting'' and ''Ending radius'', ''Radius step'', ''Number of Samples per Radius'', etc. Other parameters are described below.

[[File:BitmapSholl-Table.png|100%|center|Descriptors and metrics are listed in the Sholl Table (v2.4)]]

=== Metrics based on sampled data ===
[[ File:ShollOptionsPrompt.png|350px|right |Metrics &amp; Options prompt (version 3.6.4)]]
;Intersecting radii
:The number of sampling radii intersecting the arbor at least once.

;Sum of intersections (''Sum inters.'')<span id="Sum"></span>
:The sum of all intersections.

;Mean of intersections (''Mean inters.'')<span id="MeanInters"></span>
:''Sum inters.'' divided by ''Intersecting radii''.
: See also [[#MeanValueOfFunction|Mean value]], [[#faq:AUC|FAQ]]

;Median of intersections (''Median Inters.'')
:The median value of sampled intersections.

;Skewness<span id="Skewness"></span>
:The [[wikipedia:Skewness|skewness]] of the sampled data, an indication of how symmetrical the distribution is around its mean. Positive values indicate an asymmetrical distribution with a longer tail to the right. Negative values indicate data with a longer tail to the left. A popular rule of thumb considers that if the skewness is greater than 1.0 (or less than -1.0), the distribution may be considered far from symmetrical.
: See also [[#Skewness-fitted|Skewness (fitted data)]]

;Kurtosis<span id="Kurtosis"></span>
:The [[wikipedia:Kurtosis|kurtosis]] of the sampled data, which quantifies whether the shape of the distribution matches that of a Gaussian distribution, assuming that a Gaussian distribution has a kurtosis of 0. A distribution more peaked than a Gaussian has a positive kurtosis while a negative value indicates a flatter distribution.
: See also [[#Kurtosis-fitted|Kurtosis (fitted data)]]

;<span id="MaxInters"></span>Highest count of intersections (''Max inters.'')
:The maximum value of sampled intersections, i.e., the maximum in a linear [''N'' vs ''Distance''] profile, reflecting the highest number of processes/branches in the arbor.
: See also [[#CriticalValue|Critical value]]
{{Tip
| tip = '''''Max inters.''''': By default only the absolute maximum is noted. However, it is possible to apply peak detection techniques to the profile to retrieve other sites of high density branching. See [[#Complementary_Tools|Complementary Tools]] for more details.
}}
;<span id="MaxIntersRadius"></span>Radius of highest count of intersections (''Max inters. radius'')
:The distance at which the ''Highest count of intersections'' occurred, reflecting sites of highest branch density. Note that if the same maximum occurs multiple times, only the first distance is considered.
: See also [[#CriticalRadius|Critical radius]]

;<span id="SchoenenSampled"></span>Schoenen Ramification index (''Ramification index (sampled)'')
:A measure of ramification<ref name="Schoenen"></ref>: the ratio between ''Max inters.'' and the number of [[#PrimaryBranches|primary branches]]. It is only calculated when [[#PrimaryBranches|primary branches]] is valid and not zero.
: See also [[#SchoenenFitted|Ramification index (fit)]]

;<span id="Centroid">Centroid radius
:The abcissa of the [[wikipedia:Centroid|centroid]] (i.e., the geometric center or barycenter) of the linear profile. It is [[#ShollPlots|highlighted]] on the ''N'' vs ''Distance'' plot.

;Centroid value
:The ordinate of the [[wikipedia:Centroid|centroid]] (i.e., the geometric center or barycenter) of the linear profile. It is [[#ShollPlots|highlighted]] on the ''N'' vs ''Distance'' plot.

;<span id="EnclosingRadius"></span>Enclosing radius
:The last (thus, the widest) of ''Intersecting radii'' to be associated with the number of intersections specified by [[#EnclosingRadiusCutoff|Enclosing radius cutoff]]. For a cutoff of 1 (the default) ''Enclosing radius'' is the widest of ''Intersecting radii''. It reflects the [[wikipedia:Feret diameter|Feret length]] of the arbor.

=== Metrics based on fitted data ===
;<span id="Dratio"></span>Determination ratio
:The ratio of the [[#RegR2|coefficient of determination]] for the semi-log method and that for the log–log method<ref name="Ristanovic"></ref>. If the semi-log method is better relatively to the log–log method, the ''Determination ratio'' becomes larger than 1. It is the parameter used by the plugin to silently predict the normalization method that is the [[#Choice_of_Methods|most informative]]. The prediction can be monitored by activating the [[#Descriptors and Curve Fitting|Show fitting details ]] checkbox.

;<span id="ShollDecay"></span>Sholl regression coefficient (''Regression coefficient'')
:The slope (multiplied by -1) of the linear regression described in [[#eq3|(3)]] and [[#eq4|(4)]], i.e., ''k'', a measure of the rate of decay of the number of branches with distance from the center of analysis. Higher ''k'' values reflect larger changes in the function log(N/S). To optimize the fit, the plugin retrieves a [[#ShollPlots|second linear regression]] centered around the median distance, excluding distances at the edges of the profile. Details of this second fit are also registered on the [[#Metrics|''Sholl table'']] under dedicated columns, e.g., ''Sholl regression coefficient [P10-P90]'', when data within the 10<sup>th</sup>–90<sup>th</sup> percentile is used.

;Regression intercept
:The y-coordinate ''m'' described in [[#eq3|(3)]] and [[#eq4|(4)]].

;<span id="RegR2"></span>Regression R<sup>2</sup> (''Regression R^2'')
:The coefficient of determination of the linear regression described in [[#eq3|(3)]] and [[#eq4|(4)]].

;<span id="CriticalValue"></span>Critical value
:The local maximum of the polynomial fit, i.e, ''N'' at ''Critical radius'' in [[#eq1|(1)]]. Abbreviation: ''N<sub>m</sub>''.
: See also [[#MaxInters|Max inters.]]

;<span id="CriticalRadius"></span>Critical radius
:The distance at which ''Critical value'' occurs. By default (see [[#Advanced Usage|Advanced Usage]]), it is calculated with a precision of 1/1000 of ''Radius step size''. Abbreviation: ''r<sub>c</sub>''.
: See also [[#MaxIntersRadius|Max inters. radius]]
{{Tip
| id = Nomenclature
| tip = '''Nomenclature''': [[#References|Previous authors]] have used different terms to describe the largest value taken by the Sholl profile, including ''Dendrite maximum''. Since the Sholl technique is not restricted to dendritic arbors and can be applied to any tree-like structure such as axonal arbors, mammary ducts or blood vessels (cf. [[#Citations|List of citations]]), ''Sholl Analysis'' introduces the term [[#CriticalRadius|Critical radius]], renaming ''Dendrite maximum'' (''N<sub>m</sub>'') to [[#CriticalValue|Critical value]]. 
}}
;<span id="MeanValueOfFunction"></span>Mean value
:The mean value<ref name="Stulic"></ref> of the fitted polynomial function [[#eq1|(1)]], representing the average of intersections over the whole area occupied by the arbor. Abbreviation ''N<sub>av</sub>''.
:On the Sholl plot, it is [[#ShollPlots|highlighted]] as the height of the rectangle that has the width of ''Enclosing radius'' − ''First intersecting radius'' and the same area of the area under the fitted curve on that discrete interval. It is analogous to [[#MeanInters|Mean inters.]], the arithmetic mean of sampled intersections throughout the arbor (cf. [[#Metrics_based_on_sampled_data|Metrics based on sampled data]]). By default (see [[#Advanced Usage|Advanced Usage]]), it is calculated with a precision of 1/1000 of ''Radius step size''.

;<span id="SchoenenFitted"></span>Schoenen Ramification index (''Ramification index (fit)'')
:Schoenen Ramification index retrieved from fitted profile:  The ration between [[#CriticalValue|Critical value]] and  [[#PrimaryBranches|Number of primary branches]].
: See also [[#SchoenenSampled|Ramification index (sampled)]]

;<span id="Skewness-fitted"></span>Skewness (''Skewness (fit)'')
:The [[#Skewness|skewness]] of the fitted polynomial distribution between [[#StartRadius|Starting radius]] and [[#EndRadius|Ending radius]].

;<span id="Kurtosis-fitted"></span>Kurtosis (''Kurtosis (fit)'')
:The [[#Kurtosis|kurtosis]] of the fitted polynomial distribution between [[#StartRadius|Starting radius]] and [[#EndRadius|Ending radius]].

;Polynomial R<sup>2</sup> (''Polyn. R^2'')
:The coefficient of determination of the polynomial fit described in [[#eq1|(1)]].

== Complementary Tools ==
<span id="Extended_Fitting"></span>[[ File:AnimatedPolyFit.gif|frame|Sampled data from the ddaC cell (<span style="border-bottom:1px dotted #ccc;">File▷ Open Samples▷ ddaC Neuron</span>) being fitted to polynomials of varying degree using a complementary [[BAR]] script.]]

''Sholl Analysis'' tries to be as flexible as possible by providing several options for normalization and curve fitting. However, it cannot offer exhaustive curve fitting options as determining ''best fit models'' requires reasonable choices that are not amenable to full automation. For this reason,  complementary tools for curve fitting can be installed as needed using [[BAR]] by subscribing to its [[BAR#Installation|update site]]. Several [[BAR]] commands complement ''Sholl Analysis''. These include:
;[[#Pre-processing|Segmentation]] tools:
:Thresholding, shape-based masking and edge-detection routines (see [[BAR#List_of_BARs|full BAR list]])

;Data analysis tools:
:'''[[Find Peaks]]:''' Retrieves local maxima under several filtering options such as peak amplitude, peak height and peak width. Can be used to retrieve secondary sites of branch density
:'''{{GitHub|org=tferr|repo=Scripts|path=BAR/src/main/resources/scripts/BAR/Data_Analysis/README.md#fit-polynomial|label=Fit Polynomial}}:''' Fits a polynomial of any degree to sampled data. Features an heuristic algorithm for guessing a polynomial "best fit". Expands the built-in repertoire of polynomial fits up to 50<sup>th</sup> order functions.
:'''{{GitHub|org=tferr|repo=Scripts|path=BAR/src/main/resources/scripts/BAR/Data_Analysis/README.md#create-boxplot|label=Create Boxplot}}:''' Allows direct comparison of metrics between groups or sets of data (specially useful when tagging images with the ''Comment'' field in {{bc|color=white|Analysis|Sholl|Metrics &amp; Options...}})
:'''{{GitHub|org=tferr|repo=Scripts|path=BAR/src/main/resources/scripts/BAR/Data_Analysis/README.md#interactive-plotting|label=Interactive Plotting}}:''' Whole-purpose routine that plots data from imported spreadsheets.

== Pre-processing ==
This section discusses some aspects that should be taken into account when segmenting neuronal arbors to be processed by ''Sholl Analysis''. Since  ''image segmentation'' (i.e., the partitioning of images into analyzable parts) is vulnerable to noise and background fluorescence, it is not possible to generalize universal routines that efficiently binarize grayscale images. This means that any procedure  that tries to appropriately describe the original fluorescence image with a binary mask must be tailored to the characteristics of individual datasets. As mentioned in [[#Complementary_Tools|Complementary Tools]], several routines listed here as distributed through the [[BAR]] {{ListOfUpdateSites|update site}}.
<span id="Noise"></span>
;Noise
:Noise can be mitigated through the usage of processing filters, specially edge-preserving ones. Examples:
:* [[Rolling_Ball_Background_Subtraction|Rolling Ball]] or "Top hat" filters, e.g., <span style="border-bottom:1px dotted #ccc;">Process▷ Subtract Background...</span>
:* Median Filtering (2D/3D), e.g., <span style="border-bottom:1px dotted #ccc;">Process▷ Filters▷</span>, <span style="border-bottom:1px dotted #ccc;">Plugins▷ 3D▷</span>
:* [[Anisotropic_Diffusion_2D|Anisotropic Diffusion]], <span style="border-bottom:1px dotted #ccc;">Plugins▷ Process▷ Anisotropic Diffusion 2D</span>
:* Sobel Edge Detection, e.g., <span style="border-bottom:1px dotted #ccc;">Process▷ Find Edges</span>
:* Shen-Castan Edge Detector ([[BAR]] {{ListOfUpdateSites|update site}}), <span style="border-bottom:1px dotted #ccc;">BAR▷ Segmentation▷</span>
:* Frequency filters, e.g., <span style="border-bottom:1px dotted #ccc;">Process▷ FFT▷ Bandpass Filter...</span> 
<span id="Uneven_Illumination"></span>
;Uneven Illumination
:Uneven illumination problems, typically associated with [http://imagejdocu.tudor.lu/doku.php?id=howto:working:how_to_correct_background_illumination_in_brightfield_microscopy wide field microscopy], do occur in confocal microscopy when signal from deep layers of the tissue is not captured as bright as with superficial layers. This signal attenuation along the Z-axis will generate a shaded gradient across the stack that [[#Automated_Segmentation|histogram-based segmentation]] will need to take into account. While these problems are better tackled during acquisition (e.g., using laser ramping), it is possible to mitigate this effect using histogram-normalization techniques. Examples:
:*[[Bleach Correction]], <span style="border-bottom:1px dotted #ccc;">Image▷ Adjust▷</span>
:*[http://imagejdocu.tudor.lu/doku.php?id=plugin:stacks:attenuation_correction:start Attenuation correction]
<span id="Automated_Segmentation"></span>
;Automated Segmentation
:It is possible to adopt more sophisticated [[:Category:Segmentation|segmentation algorithms]] when [[Auto_Threshold|global thresholding methods]] do not yield satisfactory results. Examples:
:* [[Auto Local Threshold|Local Threshold]], <span style="border-bottom:1px dotted #ccc;">Image▷ Adjust▷</span>
:* [[RATS:_Robust_Automatic_Threshold_Selection|Robust Automatic Threshold Selection]], <span style="border-bottom:1px dotted #ccc;">Plugins▷ Segmentation▷</span>
:* [[Level_Sets|Level Sets]], <span style="border-bottom:1px dotted #ccc;">Plugins▷ Segmentation▷</span>
:* [[Morphological Segmentation]] (IJPB-plugins {{ListOfUpdateSites|update site}}), <span style="border-bottom:1px dotted #ccc;">Plugins▷ Segmentation▷</span>
:* [[Squassh]], split-Bregman Image Segmentation (Segmentation and Quantification of Sub-cellular Shapes, MOSAIC ToolSuite {{ListOfUpdateSites|update site}}), , <span style="border-bottom:1px dotted #ccc;">Plugins▷ Mosaic▷ Segmentation▷</span>
<span id="Semi-Automated_Segmentation"></span>
;Semi-Automated Segmentation
:Object detection and image segmentation in images with poor signal-to-noise will likely require decisions taken by a human operator. This is frequently done using hand-crafted workflows using either ImageJ's built-in tools or external add ons. Examples:
:* [[Lasso selection|Blow/Lasso Tool]], <span style="border-bottom:1px dotted #ccc;">Plugins▷ Segmentation▷</span>
:* Scripts from the [[BAR]] {{ListOfUpdateSites|update site}}, <span style="border-bottom:1px dotted #ccc;">BAR▷ [[BAR#Segmentation|Segmentation]]▷</span>
{{Tip
|tip = For additional image processing tools have a look at the growing list of  {{ListOfUpdateSites|update sites}}. For more information on image processing have a look at [[:Category:Tutorials|tutorials]], [[:Category:Segmentation|segmentation]] and the [https://imagej.net/docs/guide/ ImageJ User Guide].
}}

== Batch Processing ==
{{Learn|scripting}}It is fairly simple to [[Scripting Help|automate]] the analysis of multiple images using any of the scripting languages supported by ImageJ and Fiji ([[Introduction into Macro Programming|ImageJ Macro Language]], [[Beanshell_Scripting|Beanshell]], [[Javascript_Scripting|Javascript]], [[JRuby_Scripting|JRuby]], [[Jython_Scripting|Jython]], [[Clojure_Scripting|Clojure]], ...). This section provides some examples.

=== Batch Analysis of Images ===
Any macro or script must allow the Sholl Analysis plugin to access the ROI marking the center of analysis. One could instruct ImageJ to read the coordinates of pre-existing ROIs from a text file, store a list of line selections in the ROI Manager, or write a morphology-based routine that detects the center of the arbor. However, marking the center of analysis is probably something that you will want to do manually. Here is a workflow:

#Place all the .tif images to be processed in a single folder.
#Select the <span style="border-bottom:1px dotted #ccc;">Point Selection Tool</span> in the main ImageJ window. With 3D images, make sure ''Set stack positions'' is active in the <span style="border-bottom:1px dotted #ccc;">Image▷ Overlay▷ Overlay Options...</span> prompt.
#Open the first image and press {{key press|Shift|T}} to activate the Threshold widget (<span style="border-bottom:1px dotted #ccc;">Image▷ Adjust▷ Threshold...</span>).
#Adjust threshold levels. Press the ''Apply'' button of the Threshold widget to create a binary image.
#Select the z-slice containing the center of analysis. Click over the center with the <span style="border-bottom:1px dotted #ccc;">Point Selection Tool</span> and press {{key press|B}} (shortcut for <span style="border-bottom:1px dotted #ccc;">Image▷ Overlay▷ Add Selection...</span>). This will add the point ROI to the image overlay. Save the image as .TIFF by pressing {{key press|S}} (<span style="border-bottom:1px dotted #ccc;">File▷ Save As...▷ Tiff...</span>).
#Repeat the last 2 steps until all images are marked, using {{key press|Shift|O}} (shortcut for <span style="border-bottom:1px dotted #ccc;">File▷ Open Next</span>) to iterate through all the images.
{{Tip
| id = FileFormats
| tip = When working with ROIs, it is critical that you work with .tif files because only this format keeps track of image overlays. The <span style="border-bottom:1px dotted #ccc;">Process▷ Batch▷ Convert...</span> command allows bulk conversion between image formats.
}}
Now that all the images are marked, we just need to ask ImageJ to generate some lines of code. We will open the Macro Recorder (<span style="border-bottom:1px dotted #ccc;">Plugins▷ Macros▷ Record...</span>) and run ''Sholl Analysis'' on one of the images to find out how to call the plugin with suitable parameters. In this example, we will use the ImageJ macro language. The single line of code that appears in the recorder window will look something like this:
<source lang="java">
// Recording Sholl Analysis version 3.4.3
// Visit https://imagej.net/Sholl_Analysis#Batch_Processing for scripting examples
run("Sholl Analysis...", "starting=10 ending=400 radius_step=0 infer fit linear polynomial=[8th degree] semi-log normalizer=Volume create save do");
</source>

Now, we just need to assemble a working macro to be pasted in the <span style="border-bottom:1px dotted #ccc;">Process▷ Batch▷ Macro...</span> prompt:
<source lang="java">
// Get the number of ROIs of the image overlay
nROIs = Overlay.size;

// We cannot proceed if no ROI is available (safety check)
if (nROIs==0)
   exit("No ROI was found. Cannot proceed.");

// Select the last ROI of the overlay. Because we have activated the "Set stack positions" option,
// this will automatically activate the Z-slice in which the ROI was created (3D images)
Overlay.activateSelection(nROIs-1);

// We now call the plugin as detailed by the Macro Recorder. We'll set 'Ending radius' to a non-numeric
// value (NaN, "Not a Value") to make sure the maximum length for each individual image is used
run("Sholl Analysis...", "starting=10 ending=NaN radius_step=0 infer fit linear polynomial=[8th degree] semi-log normalizer=Volume create save do");
</source>
Of course you can also automate any preceding steps. However, do not forget to ensure that the center of analysis will be available when the plugin is called:
<source lang="java">
// Impose spatial calibration
     run("Properties...", "unit=um pixel_width=1.5 pixel_height=1.5 voxel_depth=3.0");
// Subtract background
     run("Subtract Background...", "rolling=50 sliding stack");
// Apply a favorite threshold
    setAutoThreshold("Huang dark stack");
// >>>> Make sure the initial point selection remains available <<<<
    Overlay.activateSelection( Overlay.size - 1 );
// Run the plugin
run("Sholl Analysis...", "starting=10 ending=NaN radius_step=0 infer fit linear polynomial=[8th degree] semi-log normalizer=Volume create save do");
</source>
That's it. Use the Macro Recorder to generate the customizations you will need before parsing the entire folder of images with <span style="border-bottom:1px dotted #ccc;">Process▷ Batch▷ Macro...</span>
{{Tip
| tip = As you may have noticed, ImageJ plugins are controlled by a single lowercase sentence  in which arguments are separated by a space. Input fields and choice lists appear as ''keyword=value'' pairs, active checkboxes by a single keyword. Options that are not needed can be omitted. This makes it easier to generate customizable macros:
<div style="width:98%;">
<source lang="java">
start = 10;  // variable that controls starting radius
end   = 200; // variable that controls ending radius
step  = 2;   // variable that controls step size

// Run the plugin
run("Sholl Analysis...", "starting="+ start +" ending="+ end
        +" radius_step="+ step +" infer linear save do");
</source>
</div>
}}
=== Batch Analysis of Tabular Data ===
If you already have [[#Importing|obtained profiles]] (either from previous runs or from [[#Analysis of Traced Cells|traced cells]]) and would like to extract new [[#Metrics|metrics]] from such data, you can use the  {{bc|color=white|Analysis|Sholl|Sholl Analysis (Existing Profile)...}}. Here is an example macro that runs the plugin over a folder of .csv files containing Sholl profiles produced by [[Simple_Neurite_Tracer:_Sholl_analysis|Simple Neurite Tracer]]:

<source lang="java">
distanceCol = "Radius";		// Column in .csv file listing distances
intersecCol = "Inters.";	// Column in .csv file listing intersections

dir = getDirectory("Choose the directory containing the profiles");
list = getFileList(dir);

for (i=0; i<list.length; i++) {
	if (endsWith(list[i], ".csv")) {
		path = dir+list[i];
		run("Sholl Analysis (Existing Profile)...",
				"use=[External file...] open=["+ path +"]"
				+ " name=["+ list[i] +"]"
				+ " distance="+ distanceCol
				+ " intersections="+ intersecCol
				+ " 3d enclosing=1 infer linear polynomial=[Best fitting degree]"
				+ " semi-log log-log normalizer=Area/Volume show save do");
	}
}
</source>

== Advanced Usage ==
Advanced options can be set through [http://tferr.github.io/ASA/apidocs/ API calls]. Here are some examples:

{{GitHubEmbed|org=tferr|repo=ASA|path=scripting-examples/3D_Analysis_ImageStack.py|label=3D analysis of an ImageStack (Python)}}

Reduce the number of discretization steps involved in the calculation of [[#MeanValueOfFunction|Nav]], and [[#CriticalValue|Cv]], [[#CriticalRadius|Cr]] (IJ macro (IJM) language):
<source lang="java">
call("sholl.Sholl_Analysis.setPrecision", "100"); // Default is 1000, ie, 1/1000 of radius step size
</source>
Note that the IJM built-in [http://imagej.nih.gov/ij/developer/macro/functions.html#call call("class.method")] function can only pass strings to Java methods. For this reason, you have to quote the passed argument. <tt>Sholl_Analysis</tt> will then parse the string argument and interpreter its value. Note that calls made by the IJM language need to be set before running the plugin and remain in effect while ImageJ is running.

== Auxiliary Commands ==
;{{bc|color=white|Analyze|Sholl|Combine Sholl Profiles...}}[[File:CombineShollProfiles.png|400px|thumb|right|Screenshot of 15 files being processed by {{bc|color=white|Analyze|Sholl|Combine Sholl Profiles...}} (v3.6.12)]]
Analysis tool that 1) Merges individual Sholl profiles into a single table and 2) Obtains the average profile (with standard deviation) of a group of cells.

;{{bc|color=white|File|Open Samples|ddaC Neuron}}
:Opens a sample image of a Drosophila class IV ddaC sensory neuron in which dendrites have been previously segmented (2D arbor). Use it to get acquainted with the plugin. Run {{bc|color=white|Image|Show Info...}} (shortcut: {{key press|I}} ) to know more about this cell type.

;{{bc|color=white|Help|About Plugins|About Sholl Analysis...}}
:Retrieves information about the plugin version and provides links to several resources including its source code {{GitHub|org=tferr|repo=ASA|label=repository}}  and its  [http://tferr.github.io/ASA/apidocs/ API documentation]. It is also accessible through the ''More »'' dropdown menu.

== FAQ ==
'''General'''
<ol>
	<li>
		<span id="faq:citing"></span>'''How do I cite ''Sholl Analysis'''''?
	</li>
	<dl><dd>
		The authoritative reference for ''Sholl Analysis'' is:
* Ferreira T, Blackman A, Oyrer J, Jayabal A, Chung A, Watt A, Sjöström J, van Meyel D. ('''2014'''), [http://www.nature.com/nmeth/journal/v11/n10/full/nmeth.3125.html Neuronal morphometry directly from bitmap images], ''Nature Methods'' 11(10): 982–984.
The [[Citing|authoritative reference]] for Fiji:
* Schindelin J, Arganda-Carreras I, Frise E, Kaynig V, Longair M, Pietzsch T, Preibisch S, Rueden C, Saalfeld S, Schmid B, Tinevez JY, White DJ, Hartenstein V, Eliceiri K, Tomancak P, Cardona A. ('''2012''') [http://www.nature.com/nmeth/journal/v9/n7/full/nmeth.2019.html Fiji: an open-source platform for biological-image analysis], ''Nature Methods'' 9(7): 676-682.
	</dd></dl>
	<li>
		<span id="faq:plugin"></span>'''What is the difference between ''Sholl Analysis'' and an homonymous plugin released by the Ghosh laboratory in 2005'''?
	</li>
	<dl><dd>
		The [http://labs.biology.ucsd.edu/ghosh/software/ original Sholl Analysis plugin] by Tom Maddock (version 1.0) was released for ImageJ 1.35 and is now deprecated, unmaintained software that behaves erratically in newer versions of ImageJ. The current implementation of  ''Sholl Analysis'' inherits Tom's initial 2D algorithm, but has numerous [[#Release_Notes|added features]] to enhance its utility. Note that throughout 2012 the plugin was temporarily called ''Advanced Sholl Analysis''. You can follow the entire history of the plugin on {{GitHub|org=tferr|repo=ASA|label=GitHub}}.
	</dd></dl>
	<li>
		<span id="faq:threshold"></span>'''Why do I need to threshold the cell?'''
	</li>
	<dl><dd>
		Counting intersections is really a binary procedure: a shell either intercepts a branch or it doesn't. For this reason the image must be split in two phases: ''arbor'' and ''background''.
	</dd></dl>
	<li>
		<span id="faq:threshold2"></span>'''In version 1.0, it was not mandatory to adjust threshold values prior to analysis. Why is it now?'''
	</li>
	<dl><dd>
		Image segmentation has always been [[#faq:threshold|required]]. In its early implementations, the program dealt solely with binary images and used the intensity at the center of the analysis to decide how to segregate objects from background. This approach was very restringent: It assumed that the pixels representing the neuron would have the same (constant!) intensity that was not to be found in the remaining background.  As the program became aware of grayscale images, this "feature" had to be removed because a single intensity can no longer be used to infer which parts of the image should be analyzed.
	</dd></dl>
	<li>
		<span id="faq:pre-processing"></span>'''My images do not look that ''great''. How can I treat them prior to analysis?'''
	</li>
	<dl><dd>
		Have a look at [[#Pre-processing| Pre-processing]].
	</dd></dl>
	<li>
		<span id="faq:accuracy"></span>'''My bitmap profiles are different from the ones obtained from tracings of the same cells. Why?'''
	</li>
	<dl><dd>
		As mentioned several times, the quality of the analysis relies on how the arbor was segmented. If you are working with grayscale images you probably need to optimize your [[#Pre-processing|segmentation routines]]. On the other hand, if you already obtained binary images make sure you are [[#Cf._Segmentation|interpreting them properly]]. You should also confirm that [[#EndRadius |Ending radius]] does not intersect objects in the image canvas that extend beyond the analyzed arbor. As a rule of thumb, always refer to the [[#faq:sholl-mask|Sholl mask]] to visually inspect which regions of the image have been measured.
	</dd></dl>
	<li>
		<span id="faq:updates"></span>'''My version is not the latest after running <span style="border-bottom:1px dotted #ccc;">Help▷ Update Fiji...</span> Why?'''
	</li>
	<dl><dd>
		Please note that from version 3.4.6 onwards, updates are available through the [[##Fiji_Users|Java-8 update site]]. If you have manually installed/modified ''Sholl_Analysis.jar'' ([[#Release Notes and Pre-releases|Development build]]?). Run the [[Updater]], choose ''Advanced Mode'' then ''View locally modified files'' under ''View Options''. Type "Sholl" in the ''Search'' field, selecting ''Sholl_Analysis.jar'' from the list of files. If the ''Details pane'' indicates an available update, click on ''Locally modified'' under ''Status/Action'' and choose ''Install/Update''. The latest release version will be available once you press ''Apply changes''. See [[Frequently_Asked_Questions#Installing.2FUpdating|Installation FAQs]] for more details.
	</dd></dl>
	<li>
		<span id="faq:documentation"></span>'''This documentation is not that useful. How long do I have to wait until it gets improved?'''
	</li>
	<dl><dd>
		Around 20 seconds. This is the time it will take you to [[Help:Contents#New_accounts|create an account]] on this wiki. Once you have created one, you will be able to improve this page yourself.
	</dd></dl>
</ol>

'''Analysis'''
<ol START="9">
	<li>
		<span id="faq:image-types"></span>'''The plugin complains about a wrong image type. Why?'''
	</li>
	<dl><dd>
		The plugin does not parse RGB images, but will process any grayscale image (8/16-bit), including multi-channel (composite) images. This is intentional: RGB images are inflexible and images of fluorescence-labeled cells are typically non-RGB images. As explained in the [https://imagej.net/docs/guide/ ImageJ User Guide], RGB images can be converted using {{bc|color=white|Image|Color|Channels Tool...}} or {{bc|color=white|Image|Type|}} commands.
	</dd></dl>
	<li>
		<span id="faq:compartments"></span>'''I cannot see the hemicircle/hemisphere option. Why?'''
	</li>
	<dl><dd>
		This option is only available if an orthogonal line has been created by holding {{key press|Shift}} when using the <span style="border-bottom:1px dotted #ccc;">Straight Line Selection Tool</span>. See the [https://imagej.net/docs/guide/ ImageJ User Guide] for the full list of key modifiers that can be used while creating straight line ROIs.
	</dd></dl>
	<li>
		<span id="faq:z-position"></span>'''With 3D and 4D images, how do I set the Z-position and of the center?'''
	</li>
	<dl><dd>
		The Z-position (depth) of the center of analysis is the active Z-slice of the stack. With multichannel (composite) images, the active channel also defines the C-position. Both are reported in the ''Sholl Results'' table.
	</dd></dl>
	<li>
		<span id="faq:saving"></span>'''I cannot see the option to save the results. Why?'''
	</li>
	<dl><dd>
		The image you are trying to analyze is not saved locally. Saving it to a local directory (e.g., your Desktop or Home folder) should re-enable it.
	</dd></dl>
	<li>
		<span id="faq:parameters"></span>'''Why so many parameters?'''
	</li>
	<dl><dd>
		The plugin is designed for the analysis of a wide diversity of arbors and it is not biased to any particular cell type. The only way to ensure this broad applicability is to give users full control over the mathematical techniques used by the plugin to analyze sampled data.
	</dd></dl>
</ol>

'''Results'''
<ol START="14">
	<li>
		<span id="faq:Sholl-table"></span>'''How can I save/edit the ''Sholl Results'' table?'''
	</li>
	<dl><dd>
		Select the table, then choose <span style="border-bottom:1px dotted #ccc;">File▷ Save As…</span>The filename extension can be specified using the ''More » Options...'' command (see the [https://imagej.net/docs/guide/ ImageJ User Guide] for details). Single cells cannot be modified from within ImageJ, but custom extensions (e.g., .csv, .xls or .ods) will allow the table to be imported by other spreadsheet applications.
	</dd></dl>
	<li>
		<span id="faq:precision"></span>'''Can I modify the way data is displayed?'''
	</li>
	<dl><dd>
		Mostly, using {{bc|color=white|Analysis|Sholl|Metrics &amp; Options}} (also listed in the ''More » Options...'' shortcut), including the number of decimal places reported by the ''Sholl Results'' table or the usage of scientific notation. To resize plots, use the  ''More »'' dropdown menu in the ''Options'' dialog.
	</dd></dl>	
	<li>
		<span id="faq:sholl-mask"></span>'''What is the ''Sholl mask''?'''
	</li>
	<dl><dd>
		The Sholl mask ([[#CA1CellMask|see example of CA1 cell]]) is simply an illustration: a maximum intensity projection of the analyzed cell in which [[##Output_Options|intersection counts]] are used as pixel intensities. As explained in [[#Output_Options|Output Options]], its LUT can be modified, and intensities calibrated using ImageJ default commands. As mentioned, it can also be used to visually inspect for [[#Cf._Segmentation|segmentation artifacts]].
	</dd></dl>
	<li>
		<span id="faq:3Dvs2D"></span>'''The 3D profile looks ''worse'' than the 2D profile of the Maximum Intensity Projection of the same cell. Why?'''
	</li>
	<dl><dd>
		An anisotropic voxel size will have a strong impact on [[#StepSize|step size]]. On the other hand, 2D and 3D images can be sampled differently depending on the [[#Parameters|options chosen]]. If <span style="border-bottom:1px dotted #ccc;">Image▷ Properties…</span> ({{key press|Shift|P}}) reports the appropriate spatial calibration, make sure to read [[#Multiple_Samples_and_Noise_Reduction|Multiple Samples and Noise Reduction]] before deciding which type of images to use.
	</dd></dl>
	<li>
		<span id="faq:no-output"></span>'''The program terminates without warnings. What am I doing wrong?'''
	</li>
	<dl><dd>
		The program will not terminate without throwing an error message. However, do note that some exiting messages are displayed in the often overlooked status bar of the main ImageJ window. This is intentional, as it minimizes the frequency of modal windows popping up for each failed operation.
	</dd></dl>
</ol>

'''Metrics and Curve Fitting'''
<ol START="19">
	<li>
		<span id="faq:AUC"></span>'''Would it be possible to retrieve the Area Under the Curve  (linear Sholl plot)?'''
	</li>
	<dl><dd>
		Sure. But it would hardly be relevant for data sampled at fixed intervals. The area under the curve (AUC, the area between the sampled curve and the horizontal axis, i.e., its definite integral) could be estimated using, e.g., the [[wikipedia:Trapezoidal rule|trapezoidal rule]]. However, because data is always sampled at equally spaced intervals, doing so would be the same as multiplying [[#MeanInters|Mean inters.]] by the distance between [[#EndRadius|Ending radius]] and [[#StartRadius|Starting radius]]. Thus, effectively, AUC is  redundant with [[#MeanInters|Mean inters.]], that is already an integrated measurement of the sampled data. On the other hand,
one could retrieve the AUC of the polynomial fit, but such property is already covered by [[#MeanValueOfFunction|Mean value]].
	</dd></dl>
	<li>
		<span id="faq:inflection-points"></span>'''The shape of the polynomial changes at the edges of the profile. Why?'''
	</li>
	<dl><dd>
		Inflection points at [[#StartRadius|starting/ending radius]] are usually associated with a poor fit and/or the fact that all the radii in which no intersections were counted are ignored. The latter is required to calculate the [[#ShollDecay|Sholl regression coefficient]], as ''log(0)'' is undefined.
	</dd></dl>
	<li>
		<span id="faq:poor-fitting"></span>'''None of the fitting options is suitable for my datasets. What should I do?'''
	</li>
	<dl><dd>
		Have a look at [[#Complementary Tools|Complementary Tools]].
	</dd></dl>
</ol>

'''Batch Processing'''
<ol START="22">
	<li>
		<span id="faq:recorder"></span>'''The code that the Macro Recorder produced does not seem to work. What am I doing wrong?'''
	</li>
	<dl><dd>
		It is likely that frequent interactions with the dialog prompt(s) (from which the Recorder retrieves user-specified parameters) have "confused" ImageJ. While this process is usually flawless, it may happen that repeated triggering of GUI-specific commands that are not recordable (e.g., ''[[#Cf. Segmentation|Cf. Segmentation]]'' or ''[[#Importing|Import Other Data]]'' buttons) may lead to an incomplete recording. The solution is to repeat the recording, while minimizing such interactions.
	</dd></dl>
</ol>

'''Development'''
<ol START="23">
	<li>
		<span id="faq:bug-report"></span>'''I found a bug. How do I report it?'''
	</li>
	<dl><dd>
		Report it in the [http://forum.imagej.net ImageJ Forum] or file an [https://github.com/tferr/ASA/issues issue] on GitHub. Don't forget to include the [[Bug reporting best practices|steps needed to reproduce the problem]]. You may also want to check the {{GitHub|org=tferr|repo=ASA|path=Notes.md#development-builds|label=release notes}} for the latest [http://jenkins.imagej.net/job/Sholl-Analysis/lastBuild/ development version] to see if the issue has meanwhile been addressed.
	</dd></dl>
</ol>

== Release Notes and Pre-releases ==
Releases notes are available on {{GitHub|org=tferr|repo=ASA|path=Notes.md#release-notes-for-sholl-analysis|label=Github}}. If you do not mind unstable software, {{GitHub|org=tferr|repo=ASA|path=Notes.md#development-builds|label=development builds}} may be found [http://jenkins.imagej.net/job/Sholl-Analysis here]. Once new features mature and no major issues are found these development versions will be made available, as usual, through the [[Updater]].

== Related Resources ==
* '''[[Simple_Neurite_Tracer|Simple Neurite Tracer]] (SNT)''' The remarkable ImageJ framework for semi-automated of two- and three-dimensional tracing. SNT performs Sholl using the ''Sholl Analysis'' plugin. On the other hand, ''Sholl Analysis'' uses SNT to [[#Analyze Traced Cells|analyze tracings]]

* '''[[NeuronJ]]''' Another option for 2D reconstructions. NeuronJ features a friendly interface but is restricted to 2D images, is not capable of SWC export and is no longer under active development

* '''[[:Category:Neuroanatomy|Neuroanatomy Resources]]''' in this wiki

* '''[[Neuroanatomy|Neuroanatomy update site]]''', distributing e.g., the '''[[Strahler_Analysis|Strahler_Analysis]]''' plugin

* '''[[BAR]]''', ''B''roadly ''A''pplicable ''R''outines that complement ''Sholl Analysis''

== Publication ==
* Ferreira T, Blackman A, Oyrer J, Jayabal A, Chung A, Watt A, Sjöström J, van Meyel D. ('''2014'''), [http://www.nature.com/nmeth/journal/v11/n10/full/nmeth.3125.html Neuronal morphometry directly from bitmap images], ''Nature Methods'' 11(10): 982–984

== Citations ==
While in development (2005-2014), and prior to its [[#Publication|publication]], ''Sholl Analysis'' has been cited by [https://scholar.google.ca/scholar?as_q=&as_epq=Sholl+Analysis&as_oq=imagej+fiji&as_eq=%22Simple+Neurite+Tracer%22&as_occt=any&as_sauthors=&as_publication=&as_ylo=2005&as_yhi=2014&btnG=&hl=en&as_sdt=1%2C5&as_vis=1 several manuscripts]. The manuscripts citing its 2014 [[#Publication|publication]] can be retrieved using [https://scholar.google.ca/scholar?cites=15441574333602897335&as_sdt=2005&sciodt=1,5&hl=en Scholar] or [http://www.ncbi.nlm.nih.gov/pmc/articles/pmid/25264773/citedby/?tool=pubmed PubMed]. Below is a list of publications from authors that have made the developer aware how the program contributed to their research.

* Stanko JP, Easterling MR, Fenton SE. Application of Sholl analysis to quantify changes in growth and development in rat mammary gland whole mounts. Reprod Toxicol. 2015 Jul;54:129-35 [http://www.ncbi.nlm.nih.gov/pubmed/25463529 PMID 25463529]

* Kroner A, Greenhalgh AD, Zarruk JG, Passos Dos Santos R, Gaestel M, David S. TNF and increased intracellular iron alter macrophage polarization to a detrimental M1 phenotype in the injured spinal cord. Neuron. 2014 Sep 3;83(5):1098-116 [http://www.ncbi.nlm.nih.gov/pubmed/25132469 PMID 25132469]

{{ambox | text = Please append your work here, if the plugin has been useful to your work.}}

== References ==
<references >
<ref name="Gross">
Ferreira TA, Iacono LL, Gross CT. Serotonin receptor 1A modulates actin dynamics and restricts dendritic growth in hippocampal neurons. Eur J Neurosci. 2010 Jul;32(1):18-26. [http://www.ncbi.nlm.nih.gov/pubmed?term=20561047 PMID: 20561047]
</ref>
<ref name="Sholl">
Sholl DA. Dendritic organization in the neurons of the visual and motor cortices of the cat. J Anat. 1953 Oct;87(4):387-406. [http://www.ncbi.nlm.nih.gov/pubmed?term=13117757  PMID: 13117757]
</ref>
<ref name="Stulic">
Ristanović D, Milosević NT, Stulić V. Application of modified Sholl analysis to neuronal dendritic arborization of the cat spinal cord. J Neurosci Methods. 2006 Dec 15;158(2):212-8. [http://www.ncbi.nlm.nih.gov/pubmed?term=16814868 PMID: 16814868]
</ref>
<ref name="Schoenen">
Schoenen J. The dendritic organization of the human spinal cord: the dorsal horn. Neuroscience. 1982;7(9):2057-87.[http://www.ncbi.nlm.nih.gov/pubmed?term=7145088 PMID: 7145088]
</ref>
<ref name="Ristanovic">
Milosević NT, Ristanović D. The Sholl analysis of neuronal cell images: semi-log or log-log method? J Theor Biol. 2007 Mar 7;245(1):130-40 [http://www.ncbi.nlm.nih.gov/pubmed?term=17084415 PMID: 17084415]
</ref>
<ref name="DIADEM">
Brown KM, Barrionuevo G, Canty AJ, De Paola V, Hirsch JA, Jefferis GS, Lu J, Snippe M, Sugihara I, Ascoli GA. The DIADEM data sets: representative light microscopy images of neuronal morphology to advance automation of digital reconstructions. Neuroinformatics. 2011 Sep;9(2-3):143-57, [http://www.ncbi.nlm.nih.gov/pubmed?term=21249531 PMID: 21249531]
</ref>
</references>

== License ==
This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the [http://www.gnu.org/licenses/gpl.txt Free Software Foundation]. This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.

[[Category:Citable]]
[[Category:Plugins]]
[[Category:Analysis]]
[[Category:Neuroanatomy]]