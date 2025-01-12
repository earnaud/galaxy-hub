---
title: GCC2013 Training Day
---

<slot name="/events/gcc2013/header" />

import links from "../../links.json"
<link-box :links="links" />

<div class='right'><a href='/events/gcc2013/training-day/'><img src="/images/logos/GCC2013TrainingDayLogo300.png" alt="Training Day" width="200" /></a></div>

# Advanced Tool and Data Source Configuration

## Presenters: Ross Lazarus and Dan Blankenberg

## Scheduled duration: 15:00-17:00

## Helpful links

* http://wiki.galaxyproject.org/Admin/Tools/ToolConfigSyntax
* http://www.cheetahtemplate.org/

## 15:00-15:05 Introduction to advanced tool and data source configuration

As an administrator of your own local Galaxy, you can extend Galaxy by writing new tools. There are a few things you need to do to make them work. In the simplest possible case, you need to prepare some XML and make your Galaxy read it in at startup when it reads and parses tool_conf.xml. Each tool must include a unique tool id, a visible name and a command line template. In addition, they can also include multiple tool form parameters with labels, validation and help, outputs, tests, dependency/version requirements. Galaxy uses these to set up the tool list and each selected tool's user interface form.

In summary, the essence of the entire 2 hour session is that to create your own tool in Galaxy, you need to:

* ensure the executable is available to the execution host (your own VM/login in the workshop)

* write some valid XML in a text file to describe your new tool and put it somewhere under tools/

* edit tool_conf.xml to tell Galaxy where that XML file can be found - leave out the path .../tools/

* restart Galaxy to make the new tool available

### Intro

* Introduce presenters and circulating tutors
* Scope of the session - start with simplest possible tool.
* Add complexity in the form of one useful tool feature at a time.
* Offer a series of examples covering a wide range of common tool requirements.
* We'll work as far as we can get.
* NOT explaining how Galaxy actually works.
* Moving at a fair clip through the essential steps for a new tool to become available to users on your own local Galaxy.
* Command line skills really will be needed.

## 15:05-15:25 Hello world in Galaxian

The first exercise consists of creating (or copying, your choice) a text file containing valid XML describing a simple and admittedly, not very useful tool which calls a python script to do some work. However, it will demonstrate the bare bones of the power of the Galaxy tool interface. A few lines of XML and a small python script get you a familiar, simple user interface and a single new history item - a text file containing a string. Note that this is a trivial variation on the hello world tool used in the introductory session - instead of a fugly command line, we're also introducing a python script that does the actual work. 

Steps:

1. Make a new directory [galaxy root]/tools/hello_advanced and put hello_advanced.xml there containing

<div class='indent'>
```xml
<tool id="hello_advanced" name="Hello Advanced" version="0.01">
<description>World</description>
<command interpreter="python">
hello_advanced.py -o "${output1}" -s "hello advanced world"
</command>
<outputs>
    <data format="tabular" name="output1" label="hello_advanced_world"/>
</outputs>
<help>
**What it does**
Says hello advanced world by running a python script and passing appropriate parameters
</help>
</tool>
```


Make a new python script to match the name on the command line above (hello_advanced.py) containing

```python
#!/bin/env python
# python script to echo a command line parameter to an output file also passed on the command line
# your name here
# your favourite OSI approved licence here
import sys
import optparse

def advanced():
	"""
	Trivial example
	"""
        usage = "%s -o outfilename -s stringtowrite1 -s stringtowrite2 ..." % sys.argv[0]
	parser = optparse.OptionParser(usage = usage)
	parser.add_option("-s", "--stringtowrite",
	                 action="append", type="string",dest="mystring",help="Strings to write")
	parser.add_option("-o","--outputfile",
	                 action="store", type="string",dest="outputfile",help="output text file")
	(opts, args) = parser.parse_args()
	assert len(opts.mystring) > 0, "No strings to write found on command line"
	assert opts.outputfile,"No output file name found on command line"
	outf = open(opts.outputfile,'w')
	outf.write('\n'.join(opts.mystring))
	outf.write('\n')
	outf.close

if __name__ == "__main__":
	advanced()
```


Test this script on the command line - eg something like
```
python tools/hello_advanced/hello_advanced.py -s "hello" -s "advanced" -s "world" -o /tmp/test.txt
cat /tmp/test.txt
```


**Fix any syntax errors and make sure this runs and that the expected output is generated correctly** because if it doesn't run from the command line, it certainly won't run when you try calling it from Galaxy!


This text and the python script it calls are all you need for a new, real new tool, including some help to display to the user. In this example, the executable we use is a python script which echos it's input (the string) to a new history output file. A single command line parameter **"${output1}"** on the command line is replaced with the Galaxy job execution engine's choice of path and the command line is parsed in the script

The syntax ${...} is recommended and it is also recommended that all user supplied parameters be quoted in case the parameter contains slashes or spaces which might cause the tool to fail mysteriously.
</div>

2. #2 If not already done, adjust universe_wsgi.ini by adding an admin_user email you will register with when you first log in - use commas ONLY - no spaces - to separate admin email addresses. Adjust tool_conf.xml adding a new tool path that must exactly match the directory/filename you chose for your tool.

 ```xml
 <tool file="hello_advanced/hello_advanced.xml"/> 
 ```


3. #3 Restart

 **Stop Galaxy if it's running**
 ```
 sh run.sh –stop-daemon
 ```


 **Restart Galaxy**
 ```
 sh run.sh –daemon
 ```


4. #4 Check paster.log for errors (search for “hello” to find where your tool loaded – or barfed). If it fails to load, look for the syntax error, repair it, rinse, repeat... until it loads.

5. #5 When it loads correctly, test your new tool. In your VM webrowser, visit http://localhost:8080 . Register your admin email address if you haven't already done so and log in. Test your new tool. It will write “hello world” to a new file in your history. If/when it works, find the actual commands Galaxy executed to run your tool in paster.log. If it fails, look in paster.log for hints about what went wrong. Repair and reload via the admin interface (no need to restart the Galaxy server) until it works. 

6. #6 Raise arms in victory \o/

#### Bonus points if you finish early

1. Look at what's been written to paster.log during correct execution.

2. Make it do something more interesting.

## 15:25-15:45 Hello world test

Working automated functional tests are a great way to assure yourself that your tool works correctly for at least the test cases you provide and they are required for IUC approval of tool shed tools. Everytime Galaxy is updated, running the functional tests will assure you that changes to the core Galaxy code have not broken something in your tools. Without automated tests, you would need to test by hand every time you update.

Tests could fill a workshop on their own, but we can add a simple one for the hello advanced example with a few extra lines of code. We also need to provide the expected output from the test in the test-data subdirectory so the test framework can compare what is produced when the test is run against what is expected.

Steps:

1. #1 Make a new text file in your Galaxy test-data/ directory under the name hello_world_advanced_testout.txt - we will provide that name to the test tag and the test harness will find it there. It should contain exactly the same string as a successful run of the hello world advanced script which should be the single string
  ```

hello world advanced
```


2. #2 Save a copy of hello_world_advanced.xml as hello_world_advanced1.xml

3. #3 Adjust hello_world_advanced.xml so it includes the test section shown below

 ```xml
<tool id="hello_advanced" name="Hello Advanced" version="0.02">
<description>World</description>
<command interpreter="python">
hello_advanced.py -o "${output1}" -s "hello advanced world"
</command>
<outputs>
<data format="tabular" name="output1" label="hello_advanced"/>
</outputs>
<tests>
<test>
 <output name='output1' file='hello_world_advanced_testout.txt' />
</test>
</tests>
<help>

**What it does**
Says hello advanced world by running a python script and passing appropriate parameters with a functional test
</help>
</tool>
```


3. #4 Add the same line you added to tool_conf.xml to tool_conf.xml.sample - this is used by the functional test harness to find any tools to be tested. **The test will not work** unless it is also in that tool_conf.xml.sample file.

4. #5 Reload the hello_world_advanced tool and run it again to make sure there are no syntax errors in the test section - the test won't pass unless the tool itself runs in Galaxy.

5. #6 Run a functional test on the command line and use the -id parameter to pass the tool id hello_advanced
  ```

sh run_functional_tests.sh -id hello_advanced
```


If the test does not work you will see some tracebacks which will indicate what you need to fix. There will be a default output file run_functional_tests.html containing the test results with failure details if it did not work

## 15:40-15:55. Hello repeating input

Add a repeating group input parameter as shown. These are handy when you need an unknown number of parameters from the user since they allow the user to simply add more until they are done. Save. Reload the tool via the admin interface and test it out. Repeat until it's working right. Experiment and play with the new repeating parameter. Note how the repeats are passed to the python script, where the optparse "append" option adds them to a list of strings which are then written as newline delimited rows. 

```xml
<tool id="hello_advanced" name="Hello Advanced" version="0.03">
<description>World</description>
<command interpreter="python">
hello_advanced.py -o "${output1}"
#for x in $writeme
-s "$x.astring"
#end for
</command>
<inputs>
<repeat name="writeme" title="Strings to be written">
<param name="astring" type="text" label="An interesting string to write" help="keep adding these if you want"/>
</repeat>
</inputs>
<outputs>
<data format="tabular" name="output1" label="hello_advanced_repeats"/>
</outputs>
<help>
**What it does**
Says hello advanced world by running a python script and passing appropriate parameters
Any number of strings can be input by the user through the use of a repeat tag
</help>
</tool>
```


### Bonus points

1. 1 Experiment with tabs as separators ('\t') or commas or whatever instead of '\n'.

## 15:55 – 16:10 Hello_conditional

Conditional tags allow control flow in a tool form such as the "advanced options" control in the BWA/BWA2 tools forms. Add a very simple one as follows to allow the user to either input only one string without the repeat tag, or if they want to use the repeat tag and add as many as they feel like. Save hello_advanced.xml as hello_advanced2.xml as a backup and replace hello_advanced.xml with something like

```xml
<tool id="hello_advanced" name="Hello Advanced" version="0.04">
<description>World</description>
<command interpreter="python">
hello_advanced.py -o "${output1}"
#if $allowMulti.onlyOne == "yes"
 #for x in $allowMulti.writeme
  -s "$x.strings"
 #end for
#else
  -s "$allowMulti.astring"
#end if
</command>
<inputs>
   <conditional name="allowMulti">
      <param name="onlyOne" type="select" label="Allow multiple strings?">
        <option value="yes" selected="True">Use the repeat tag</option>
        <option value="no">No repeat tag</option>
      </param>
      <when value="yes" >
        <repeat name="writeme" title="Strings">
           <param name="strings" type="text" label="An interesting string to write" help="keep adding these if you want"/>
        </repeat>
      </when>
      <when value="no">
           <param name="astring" type="text" label="An interesting string to write" help="You only get one of these!"/>
      </when>
    </conditional>
</inputs>
<outputs>
<data format="tabular" name="output1" label="hello_advanced_repeats"/>
</outputs>
<help>
**What it does**
Says hello advanced world by running a python script and passing appropriate parameters
Optionally, any number of strings can be input by the user through the use of a repeat tag. Or not.
</help>
</tool>
```


Note the use of #if and other cheetah tags to control the command line depending on how the user has set the conditional tag and whether there are repeats to add to the command line. This additional logic is a necessary complication and studying working examples like the BWA wrapper is helpful if you get stuck.

Reload the hello_advanced tool from the admin interface and use the redo button to recreate the form - test it with the repeat tag turned off and a single string, then with the repeat.

Check the output of paster.log to see how Galaxy is setting up the command line for the call to the python script for you.

Notice how the repeat group starts out empty. See if you can change it so there is always at least one string parameter showing on the form when the repeat group is turned on.
http://wiki.galaxyproject.org/Admin/Tools/ToolConfigSyntax#A.3Crepeat.3E_tag_set has the change you need.

## 16:10 – 16:30 Hello Tool Data Tables

Many Galaxy tools are able to make use of built-in reference data, e.g. genome indexes for the bwa aligner, that a user can choose from e.g. a select list. Ordinarily, this select list would need to be hard-coded into the tool's xml config, but by relying on Tool Data Tables, we can have the options of the select list populated with content from an external file. 

Currently Tool Data Tables use tab-delimited files (the framework is generic and other formatted files can be defined); each field in the table is separated by a tab character.

A bare minimum for tool data tables is to include at least a value (required) and a display name (defaults to value when not specified) that will be used to populate the tool form and determine the value to pass on the command-line. The exact number and content of the columns to use with a tool data table will vary for the specific purpose, but a good practice would be to include an unique ID (value), name, dbkey (when needed), and command-line value. For example, the bwa_index.loc file has the form:
```
<unique_build_id>	<dbkey>	<display_name>	<file_path>
```


with a tool data table defined in tool_data_table_conf.xml:
```xml
<tables>
    <!-- Locations of indexes in the BWA mapper format -->
    <table name="bwa_indexes" comment_char="#">
        <columns>value, dbkey, name, path</columns>
        <file path="tool-data/bwa_index.loc" />
    </table>
</tables>
```

Here the value is the unique id and is the value stored in the database (for e.g. rerun). The path column contains the path to the indexes, which will be the value passed to the command-line; this allows the underlying paths to the indexes to change over time, as needed, but to remain usable in workflows or via rerun.

Inside of the bwa tool xml file, we then define the select list parameter as:
```xml
<param name="indices" type="select" label="Select a reference genome">
          <options from_data_table="bwa_indexes">
            <validator type="no_options" message="No indexes are available" />
          </options>
        </param>
```

and can pass the "path" value of the selected data table entry as:
```
"${indices.fields.path}"
```


Create a new location file tool-data/hello_world.loc, and add several entries, e.g. of the form:
```
#<greeting_id>	<greeting_text>	<path_to_image_file_of_greeting>	<world_where_greating_is_valid>
greeting_hello	Hello	/path/to/file.png	Earth
```

**Be sure to check that white space between fields are `<TABS>` and not spaces (double check, some editors automatically replace tab with space).**

Edit your tool_data_tables_conf.xml file and define the structure of the data table:
```xml
<table name="hello_world" comment_char="#">
        <columns>value, name, image_path, valid_world</columns>
        <file path="tool-data/hello_world.loc" />
    </table>
```


Define the new parameter as 
```xml
<param name="builtin_greeting" type="select" label="Select a greeting">
          <options from_data_table="hello_world">
            <validator type="no_options" message="No indexes are available" />
          </options>
        </param>
```


You can then access the various fields in the command-line by using e.g
```
-s "${builtin_greeting.fields.name}"
```

or
```
-s "${builtin_greeting.fields.valid_world}"
```


Feel free to play around with different numbers of entries passing different values via the command-line.

## 16:30 – 16:45 Hello Macros

Macros allow the reuse of commonly used chunks of code (e.g. parameter definitions and commandline cheetah code). This is particularly useful for tool suites that may have multiple individual tools, but which share a collection of commonly defined parameters.

Extensively documented: http://wiki.galaxyproject.org/Admin/Tools[/ToolConfigSyntax](/events/gcc2013/training-day/advance-tool-data/ToolConfigSyntax/)#Reusing_Repeated_Configuration_Elements

An example is the GATK. See tools/gatk/unified_genotyper.xml which makes use of the Macro file tools/gatk/gatk_macros.xml.

unified_genotyper.xml:
```xml
<tool id="gatk_unified_genotyper" name="Unified Genotyper" version="0.0.6">
  <description>SNP and indel caller</description>
  <requirements>
      <requirement type="package" version="1.4">gatk</requirement>
      <requirement type="package">samtools</requirement>
  </requirements>
  <macros>
    <import>gatk_macros.xml</import>
  </macros>
  <command interpreter="python">gatk_wrapper.py
   --max_jvm_heap_fraction "1"
   --stdout "${output_log}"
   #for $i, $input_bam in enumerate( $reference_source.input_bams ):
       -d "-I" "${input_bam.input_bam}" "${input_bam.input_bam.ext}" "gatk_input_${i}"
       #if str( $input_bam.input_bam.metadata.bam_index ) != "None":
           -d "" "${input_bam.input_bam.metadata.bam_index}" "bam_index" "gatk_input_${i}" ##hardcode galaxy ext type as bam_index
       #end if
   #end for
   -p 'java 
    -jar "${GALAXY_DATA_INDEX_DIR}/shared/jars/gatk/GenomeAnalysisTK.jar"
    -T "UnifiedGenotyper"
    --num_threads 4 ##hard coded, for now
    --out "${output_vcf}"
    --metrics_file "${output_metrics}"
    -et "NO_ET" ##ET no phone home
    ##-log "${output_log}" ##don't use this to log to file, instead directly capture stdout
    #if $reference_source.reference_source_selector != "history":
        -R "${reference_source.ref_file.fields.path}"
    #end if
    --genotype_likelihoods_model "${genotype_likelihoods_model}"
    --standard_min_confidence_threshold_for_calling "${standard_min_confidence_threshold_for_calling}"
    --standard_min_confidence_threshold_for_emitting "${standard_min_confidence_threshold_for_emitting}"
   '
    #set $rod_binding_names = dict()
    #for $rod_binding in $rod_bind:
        #if str( $rod_binding.rod_bind_type.rod_bind_type_selector ) == 'custom':
            #set $rod_bind_name = $rod_binding.rod_bind_type.custom_rod_name
        #else
            #set $rod_bind_name = $rod_binding.rod_bind_type.rod_bind_type_selector
        #end if
        #set $rod_binding_names[$rod_bind_name] = $rod_binding_names.get( $rod_bind_name, -1 ) + 1
        -d "--dbsnp:${rod_bind_name},%(file_type)s" "${rod_binding.rod_bind_type.input_rod}" "${rod_binding.rod_bind_type.input_rod.ext}" "input_${rod_bind_name}_${rod_binding_names[$rod_bind_name]}"
    #end for
   
    #include source=$standard_gatk_options#
    ##start analysis specific options
    #if $analysis_param_type.analysis_param_type_selector == "advanced":
        -p '
        --p_nonref_model "${analysis_param_type.p_nonref_model}"
        --heterozygosity "${analysis_param_type.heterozygosity}"
        --pcr_error_rate "${analysis_param_type.pcr_error_rate}"
        --genotyping_mode "${analysis_param_type.genotyping_mode_type.genotyping_mode}"
        #if str( $analysis_param_type.genotyping_mode_type.genotyping_mode ) == 'GENOTYPE_GIVEN_ALLELES':
            --alleles "${analysis_param_type.genotyping_mode_type.input_alleles_rod}"
        #end if
        --output_mode "${analysis_param_type.output_mode}"
        ${analysis_param_type.compute_SLOD}
        --min_base_quality_score "${analysis_param_type.min_base_quality_score}"
        --max_deletion_fraction "${analysis_param_type.max_deletion_fraction}"
        --max_alternate_alleles "${analysis_param_type.max_alternate_alleles}"
        --min_indel_count_for_genotyping "${analysis_param_type.min_indel_count_for_genotyping}"
        --indel_heterozygosity "${analysis_param_type.indel_heterozygosity}"
        --indelGapContinuationPenalty "${analysis_param_type.indelGapContinuationPenalty}"
        --indelGapOpenPenalty "${analysis_param_type.indelGapOpenPenalty}"
        --indelHaplotypeSize "${analysis_param_type.indelHaplotypeSize}"
        ${analysis_param_type.doContextDependentGapPenalties}
        #if str( $analysis_param_type.annotation ) != "None":
            #for $annotation in str( $analysis_param_type.annotation.fields.gatk_value ).split( ','):
                --annotation "${annotation}"
            #end for
        #end if
        #for $additional_annotation in $analysis_param_type.additional_annotations:
            --annotation "${additional_annotation.additional_annotation_name}"
        #end for
        #if str( $analysis_param_type.group ) != "None":
            #for $group in str( $analysis_param_type.group ).split( ','):
                --group "${group}"
            #end for
        #end if
        #if str( $analysis_param_type.exclude_annotations ) != "None":
            #for $annotation in str( $analysis_param_type.exclude_annotations.fields.gatk_value ).split( ','):
                --excludeAnnotation "${annotation}"
            #end for
        #end if
        ${analysis_param_type.multiallelic}
        '
    #end if
  </command>
  <inputs>
    <conditional name="reference_source">
      <expand macro="reference_source_selector_param" />
      <when value="cached">
        <repeat name="input_bams" title="BAM file" min="1" help="-I,--input_file &amp;lt;input_file&amp;gt;">
            <param name="input_bam" type="data" format="bam" label="BAM file">
              <validator type="unspecified_build" />
              <validator type="dataset_metadata_in_data_table" table_name="gatk_picard_indexes" metadata_name="dbkey" metadata_column="dbkey" message="Sequences are not currently available for the specified build." /> <!-- fixme!!! this needs to be a select -->
            </param>
        </repeat>
        <param name="ref_file" type="select" label="Using reference genome" help="-R,--reference_sequence &amp;lt;reference_sequence&amp;gt;">
          <options from_data_table="gatk_picard_indexes">
            <!-- <filter type="data_meta" key="dbkey" ref="input_bam" column="dbkey"/> does not yet work in a repeat...--> 
          </options>
          <validator type="no_options" message="A built-in reference genome is not available for the build associated with the selected input file"/>
        </param>
      </when>
      <when value="history"> <!-- FIX ME!!!! -->
        <repeat name="input_bams" title="BAM file" min="1" help="-I,--input_file &amp;lt;input_file&amp;gt;">
            <param name="input_bam" type="data" format="bam" label="BAM file" >
            </param>
        </repeat>
        <param name="ref_file" type="data" format="fasta" label="Using reference file" help="-R,--reference_sequence &amp;lt;reference_sequence&amp;gt;" />
      </when>
    </conditional>
    
    <repeat name="rod_bind" title="Binding for reference-ordered data" help="-D,--dbsnp &amp;lt;dbsnp&amp;gt;">
        <conditional name="rod_bind_type">
          <param name="rod_bind_type_selector" type="select" label="Binding Type">
            <option value="dbsnp" selected="True">dbSNP</option>
            <option value="snps">SNPs</option>
            <option value="indels">INDELs</option>
            <option value="custom">Custom</option>
          </param>
          <when value="dbsnp">
              <param name="input_rod" type="data" format="vcf" label="ROD file" />
          </when>
          <when value="snps">
              <param name="input_rod" type="data" format="vcf" label="ROD file" />
          </when>
          <when value="indels">
              <param name="input_rod" type="data" format="vcf" label="ROD file" />
          </when>
          <when value="custom">
              <param name="custom_rod_name" type="text" value="Unknown" label="ROD Name"/>
              <param name="input_rod" type="data" format="vcf" label="ROD file" />
          </when>
        </conditional>
    </repeat>
    
    <param name="genotype_likelihoods_model" type="select" label="Genotype likelihoods calculation model to employ" help="-glm,--genotype_likelihoods_model &amp;lt;genotype_likelihoods_model&amp;gt;">
      <option value="BOTH" selected="True">BOTH</option>
      <option value="SNP">SNP</option>
      <option value="INDEL">INDEL</option>
    </param>
    
    <param name="standard_min_confidence_threshold_for_calling" type="float" value="30.0" label="The minimum phred-scaled confidence threshold at which variants not at 'trigger' track sites should be called" help="-stand_call_conf,--standard_min_confidence_threshold_for_calling &amp;lt;standard_min_confidence_threshold_for_calling&amp;gt;" />
    <param name="standard_min_confidence_threshold_for_emitting" type="float" value="30.0" label="The minimum phred-scaled confidence threshold at which variants not at 'trigger' track sites should be emitted (and filtered if less than the calling threshold)" help="-stand_emit_conf,--standard_min_confidence_threshold_for_emitting &amp;lt;standard_min_confidence_threshold_for_emitting&amp;gt;" />

    
    <expand macro="gatk_param_type_conditional" />
    
    <expand macro="analysis_type_conditional">
        <param name="p_nonref_model" type="select" label="Non-reference probability calculation model to employ" help="-pnrm,--p_nonref_model &amp;lt;p_nonref_model&amp;gt;">
          <option value="EXACT" selected="True">EXACT</option>
          <option value="GRID_SEARCH">GRID_SEARCH</option>
        </param>
        <param name="heterozygosity" type="float" value="1e-3" label="Heterozygosity value used to compute prior likelihoods for any locus" help="-hets,--heterozygosity &amp;lt;heterozygosity&amp;gt;" />
        <param name="pcr_error_rate" type="float" value="1e-4" label="The PCR error rate to be used for computing fragment-based likelihoods" help="-pcr_error,--pcr_error_rate &amp;lt;pcr_error_rate&amp;gt;" />
        <conditional name="genotyping_mode_type">
          <param name="genotyping_mode" type="select" label="How to determine the alternate allele to use for genotyping" help="-gt_mode,--genotyping_mode &amp;lt;genotyping_mode&amp;gt;">
            <option value="DISCOVERY" selected="True">DISCOVERY</option>
            <option value="GENOTYPE_GIVEN_ALLELES">GENOTYPE_GIVEN_ALLELES</option>
          </param>
          <when value="DISCOVERY">
            <!-- Do nothing here -->
          </when>
          <when value="GENOTYPE_GIVEN_ALLELES">
            <param name="input_alleles_rod" type="data" format="vcf" label="Alleles ROD file" help="-alleles,--alleles &amp;lt;alleles&amp;gt;" />
          </when>
        </conditional>
        <param name="output_mode" type="select" label="Should we output confident genotypes (i.e. including ref calls) or just the variants?" help="-out_mode,--output_mode &amp;lt;output_mode&amp;gt;">
          <option value="EMIT_VARIANTS_ONLY" selected="True">EMIT_VARIANTS_ONLY</option>
          <option value="EMIT_ALL_CONFIDENT_SITES">EMIT_ALL_CONFIDENT_SITES</option>
          <option value="EMIT_ALL_SITES">EMIT_ALL_SITES</option>
        </param>
        <param name="compute_SLOD" type="boolean" truevalue="--computeSLOD" falsevalue="" label="Compute the SLOD" help="--computeSLOD" />
        <param name="min_base_quality_score" type="integer" value="17" label="Minimum base quality required to consider a base for calling" help="-mbq,--min_base_quality_score &amp;lt;min_base_quality_score&amp;gt;" />
        <param name="max_deletion_fraction" type="float" value="0.05" label="Maximum fraction of reads with deletions spanning this locus for it to be callable" help="to disable, set to &lt; 0 or &gt; 1 (-deletions,--max_deletion_fraction &amp;lt;max_deletion_fraction&amp;gt;)" />
        <param name="max_alternate_alleles" type="integer" value="5" label="Maximum number of alternate alleles to genotype" help="-maxAlleles,--max_alternate_alleles &amp;lt;max_alternate_alleles&amp;gt;" />
        <param name="min_indel_count_for_genotyping" type="integer" value="5" label="Minimum number of consensus indels required to trigger genotyping run" help="-minIndelCnt,--min_indel_count_for_genotyping &amp;lt;min_indel_count_for_genotyping&amp;gt;" />
        <param name="indel_heterozygosity" type="float" value="0.000125" label="Heterozygosity for indel calling" help="1.0/8000==0.000125 (-indelHeterozygosity,--indel_heterozygosity &amp;lt;indel_heterozygosity&amp;gt;)"/>
        <param name="indelGapContinuationPenalty" type="float" value="10.0" label="Indel gap continuation penalty" help="--indelGapContinuationPenalty" />
        <param name="indelGapOpenPenalty" type="float" value="45.0" label="Indel gap open penalty" help="--indelGapOpenPenalty" />
        <param name="indelHaplotypeSize" type="integer" value="80" label="Indel haplotype size" help="--indelHaplotypeSize" />
        <param name="doContextDependentGapPenalties" type="boolean" truevalue="--doContextDependentGapPenalties" falsevalue="" label="Vary gap penalties by context" help="--doContextDependentGapPenalties" />
        <param name="annotation" type="select" multiple="True" display="checkboxes" label="Annotation Types" help="-A,--annotation &amp;lt;annotation&amp;gt;">
          <!-- load the available annotations from an external configuration file, since additional ones can be added to local installs -->
          <options from_data_table="gatk_annotations">
            <filter type="multiple_splitter" column="tools_valid_for" separator=","/>
            <filter type="static_value" value="UnifiedGenotyper" column="tools_valid_for"/>
          </options>
        </param>
        <repeat name="additional_annotations" title="Additional annotation" help="-A,--annotation &amp;lt;annotation&amp;gt;">
          <param name="additional_annotation_name" type="text" value="" label="Annotation name" />
        </repeat>
<!--
        <conditional name="snpEff_rod_bind_type">
          <param name="snpEff_rod_bind_type_selector" type="select" label="Provide a snpEff reference-ordered data file">
            <option value="set_snpEff">Set snpEff</option>
            <option value="exclude_snpEff" selected="True">Don't set snpEff</option>
          </param>
          <when value="exclude_snpEff">
          </when>
          <when value="set_snpEff">
            <param name="snpEff_input_rod" type="data" format="vcf" label="ROD file" />
            <param name="snpEff_rod_name" type="hidden" value="snpEff" label="ROD Name"/>
          </when>
        </conditional>
-->
        <param name="group" type="select" multiple="True" display="checkboxes" label="Annotation Interfaces/Groups" help="-G,--group &amp;lt;group&amp;gt;">
            <option value="RodRequiringAnnotation">RodRequiringAnnotation</option>
            <option value="Standard">Standard</option>
            <option value="Experimental">Experimental</option>
            <option value="WorkInProgress">WorkInProgress</option>
            <option value="RankSumTest">RankSumTest</option>
            <!-- <option value="none">none</option> -->
        </param>
    <!--     <param name="family_string" type="text" value="" label="Family String"/> -->
        <param name="exclude_annotations" type="select" multiple="True" display="checkboxes" label="Annotations to exclude" help="-XA,--excludeAnnotation &amp;lt;excludeAnnotation&amp;gt;" >
          <!-- load the available annotations from an external configuration file, since additional ones can be added to local installs -->
          <options from_data_table="gatk_annotations">
            <filter type="multiple_splitter" column="tools_valid_for" separator=","/>
            <filter type="static_value" value="UnifiedGenotyper" column="tools_valid_for"/>
          </options>
        </param>
        <param name="multiallelic" type="boolean" truevalue="--multiallelic" falsevalue="" label="Allow the discovery of multiple alleles (SNPs only)" help="--multiallelic" />
    </expand>
  </inputs>
  <outputs>
    <data format="vcf" name="output_vcf" label="${tool.name} on ${on_string} (VCF)" />
    <data format="txt" name="output_metrics" label="${tool.name} on ${on_string} (metrics)" />
    <data format="txt" name="output_log" label="${tool.name} on ${on_string} (log)" />
  </outputs>
  <trackster_conf/>
  <tests>
      <test>
          <param name="reference_source_selector" value="history" />
          <param name="ref_file" value="phiX.fasta" ftype="fasta" />
          <param name="input_bam" value="gatk/gatk_table_recalibration/gatk_table_recalibration_out_1.bam" ftype="bam" />
          <param name="rod_bind_type_selector" value="dbsnp" />
          <param name="input_rod" value="gatk/fake_phiX_variant_locations.vcf" ftype="vcf" />
          <param name="standard_min_confidence_threshold_for_calling" value="0" />
          <param name="standard_min_confidence_threshold_for_emitting" value="4" />
          <param name="gatk_param_type_selector" value="basic" />
          <param name="analysis_param_type_selector" value="advanced" />
          <param name="genotype_likelihoods_model" value="BOTH" />
          <param name="p_nonref_model" value="EXACT" />
          <param name="heterozygosity" value="0.001" />
          <param name="pcr_error_rate" value="0.0001" />
          <param name="genotyping_mode" value="DISCOVERY" />
          <param name="output_mode" value="EMIT_ALL_CONFIDENT_SITES" />
          <param name="compute_SLOD" />
          <param name="min_base_quality_score" value="17" />
          <param name="max_deletion_fraction" value="-1" />
          <param name="min_indel_count_for_genotyping" value="2" />
          <param name="indel_heterozygosity" value="0.000125" />
          <param name="indelGapContinuationPenalty" value="10" />
          <param name="indelGapOpenPenalty" value="3" />
          <param name="indelHaplotypeSize" value="80" />
          <param name="doContextDependentGapPenalties" />
          <!-- <param name="annotation" value="" />
          <param name="group" value="" /> -->
          <output name="output_vcf" file="gatk/gatk_unified_genotyper/gatk_unified_genotyper_out_1.vcf" lines_diff="4" /> 
          <output name="output_metrics" file="gatk/gatk_unified_genotyper/gatk_unified_genotyper_out_1.metrics" /> 
          <output name="output_log" file="gatk/gatk_unified_genotyper/gatk_unified_genotyper_out_1.log.contains" compare="contains" />
      </test>
  </tests>
  <help>
**What it does**

A variant caller which unifies the approaches of several disparate callers.  Works for single-sample and multi-sample data.  The user can choose from several different incorporated calculation models.

For more information on the GATK Unified Genotyper, see this `tool specific page &lt;http://www.broadinstitute.org/gsa/wiki/index.php/Unified_genotyper&gt;`_.

To learn about best practices for variant detection using GATK, see this `overview &lt;http://www.broadinstitute.org/gsa/wiki/index.php/Best_Practice_Variant_Detection_with_the_GATK_v3&gt;`_.

If you encounter errors, please view the `GATK FAQ &lt;http://www.broadinstitute.org/gsa/wiki/index.php/Frequently_Asked_Questions&gt;`_.

------

**Inputs**

GenomeAnalysisTK: UnifiedGenotyper accepts an aligned BAM input file.


**Outputs**

The output is in VCF format.


Go `here &lt;http://www.broadinstitute.org/gsa/wiki/index.php/Input_files_for_the_GATK&gt;`_ for details on GATK file formats.

-------

**Settings**::

 genotype_likelihoods_model                        Genotype likelihoods calculation model to employ -- BOTH is the default option, while INDEL is also available for calling indels and SNP is available for calling SNPs only (SNP|INDEL|BOTH)
 p_nonref_model                                    Non-reference probability calculation model to employ -- EXACT is the default option, while GRID_SEARCH is also available. (EXACT|GRID_SEARCH)
 heterozygosity                                    Heterozygosity value used to compute prior likelihoods for any locus
 pcr_error_rate                                    The PCR error rate to be used for computing fragment-based likelihoods
 genotyping_mode                                   Should we output confident genotypes (i.e. including ref calls) or just the variants? (DISCOVERY|GENOTYPE_GIVEN_ALLELES)
 output_mode                                       Should we output confident genotypes (i.e. including ref calls) or just the variants? (EMIT_VARIANTS_ONLY|EMIT_ALL_CONFIDENT_SITES|EMIT_ALL_SITES)
 standard_min_confidence_threshold_for_calling     The minimum phred-scaled confidence threshold at which variants not at 'trigger' track sites should be called
 standard_min_confidence_threshold_for_emitting    The minimum phred-scaled confidence threshold at which variants not at 'trigger' track sites should be emitted (and filtered if less than the calling threshold)
 noSLOD                                            If provided, we will not calculate the SLOD
 min_base_quality_score                            Minimum base quality required to consider a base for calling
 max_deletion_fraction                             Maximum fraction of reads with deletions spanning this locus for it to be callable [to disable, set to &lt; 0 or &gt; 1; default:0.05]
 min_indel_count_for_genotyping                    Minimum number of consensus indels required to trigger genotyping run
 indel_heterozygosity                              Heterozygosity for indel calling
 indelGapContinuationPenalty                       Indel gap continuation penalty
 indelGapOpenPenalty                               Indel gap open penalty
 indelHaplotypeSize                                Indel haplotype size
 doContextDependentGapPenalties                    Vary gap penalties by context
 indel_recal_file                                  Filename for the input covariates table recalibration .csv file - EXPERIMENTAL, DO NO USE
 indelDebug                                        Output indel debug info
 out                                               File to which variants should be written
 annotation                                        One or more specific annotations to apply to variant calls
 group                                             One or more classes/groups of annotations to apply to variant calls

@CITATION_SECTION@
  </help>
</tool>
```


gatk_macros.xml:
```xml
<macros>
  <template name="standard_gatk_options">      
    ##start standard gatk options
    #if $gatk_param_type.gatk_param_type_selector == "advanced":
        #for $pedigree in $gatk_param_type.pedigree:
            -p '--pedigree "${pedigree.pedigree_file}"'
        #end for
        #for $pedigree_string in $gatk_param_type.pedigree_string_repeat:
            -p '--pedigreeString "${pedigree_string.pedigree_string}"'
        #end for
        -p '--pedigreeValidationType "${gatk_param_type.pedigree_validation_type}"'
        #for $read_filter in $gatk_param_type.read_filter:
            -p '--read_filter "${read_filter.read_filter_type.read_filter_type_selector}"
            ###raise Exception( str( dir( $read_filter ) ) )
            #for $name, $param in $read_filter.read_filter_type.iteritems():
                #if $name not in [ "__current_case__", "read_filter_type_selector" ]:
                    #if hasattr( $param.input, 'truevalue' ):
                        ${param}
                    #else:
                        --${name} "${param}"
                    #end if
                #end if
            #end for
            '
        #end for
        #for $interval_count, $input_intervals in enumerate( $gatk_param_type.input_interval_repeat ):
            -d "--intervals" "${input_intervals.input_intervals}" "${input_intervals.input_intervals.ext}" "input_intervals_${interval_count}"
        #end for
        
        #for $interval_count, $input_intervals in enumerate( $gatk_param_type.input_exclude_interval_repeat ):
            -d "--excludeIntervals" "${input_intervals.input_exclude_intervals}" "${input_intervals.input_exclude_intervals.ext}" "input_exlude_intervals_${interval_count}"
        #end for

        -p '--interval_set_rule "${gatk_param_type.interval_set_rule}"'
        
        -p '--downsampling_type "${gatk_param_type.downsampling_type.downsampling_type_selector}"'
        #if str( $gatk_param_type.downsampling_type.downsampling_type_selector ) != "NONE":
            -p '--${gatk_param_type.downsampling_type.downsample_to_type.downsample_to_type_selector} "${gatk_param_type.downsampling_type.downsample_to_type.downsample_to_value}"'
        #end if
        -p '
        --baq "${gatk_param_type.baq}"
        --baqGapOpenPenalty "${gatk_param_type.baq_gap_open_penalty}"
        ${gatk_param_type.use_original_qualities}
        --defaultBaseQualities "${gatk_param_type.default_base_qualities}"
        --validation_strictness "${gatk_param_type.validation_strictness}"
        --interval_merging "${gatk_param_type.interval_merging}"
        ${gatk_param_type.disable_experimental_low_memory_sharding}
        ${gatk_param_type.non_deterministic_random_seed}
        '
        #for $rg_black_list_count, $rg_black_list in enumerate( $gatk_param_type.read_group_black_list_repeat ):
            #if $rg_black_list.read_group_black_list_type.read_group_black_list_type_selector == "file":
                -d "--read_group_black_list" "${rg_black_list.read_group_black_list_type.read_group_black_list}" "txt" "input_read_group_black_list_${rg_black_list_count}"
            #else
                -p '--read_group_black_list "${rg_black_list.read_group_black_list_type.read_group_black_list}"'
            #end if
        #end for
    #end if
    
    #if str( $reference_source.reference_source_selector ) == "history":
        -d "-R" "${reference_source.ref_file}" "${reference_source.ref_file.ext}" "gatk_input"
    #end if
    ##end standard gatk options
  </template>
  <xml name="gatk_param_type_conditional">
    <conditional name="gatk_param_type">
      <param name="gatk_param_type_selector" type="select" label="Basic or Advanced GATK options">
        <option value="basic" selected="True">Basic</option>
        <option value="advanced">Advanced</option>
      </param>
      <when value="basic">
        <!-- Do nothing here -->
      </when>
      <when value="advanced">
        <repeat name="pedigree" title="Pedigree file" help="-ped,--pedigree &amp;lt;pedigree&amp;gt;">
            <param name="pedigree_file" type="data" format="txt" label="Pedigree files for samples"/>
        </repeat>
        <repeat name="pedigree_string_repeat" title="Pedigree string" help="-pedString,--pedigreeString &amp;lt;pedigreeString&amp;gt;">
            <param name="pedigree_string" type="text" value="" label="Pedigree string for samples"/>
        </repeat>
        <param name="pedigree_validation_type" type="select" label="How strict should we be in validating the pedigree information" help="-pedValidationType,--pedigreeValidationType &amp;lt;pedigreeValidationType&amp;gt;">
          <option value="STRICT" selected="True">STRICT</option>
          <option value="SILENT">SILENT</option>
        </param>
        <repeat name="read_filter" title="Read Filter" help="-rf,--read_filter &amp;lt;read_filter&amp;gt;">
            <conditional name="read_filter_type">
              <param name="read_filter_type_selector" type="select" label="Read Filter Type">
                <option value="BadCigar">BadCigar</option>
                <option value="BadMate">BadMate</option>
                <option value="DuplicateRead">DuplicateRead</option>
                <option value="FailsVendorQualityCheck">FailsVendorQualityCheck</option>
                <option value="MalformedRead">MalformedRead</option>
                <option value="MappingQuality">MappingQuality</option>
                <option value="MappingQualityUnavailable">MappingQualityUnavailable</option>
                <option value="MappingQualityZero">MappingQualityZero</option>
                <option value="MateSameStrand">MateSameStrand</option>
                <option value="MaxInsertSize">MaxInsertSize</option>
                <option value="MaxReadLength" selected="True">MaxReadLength</option>
                <option value="MissingReadGroup">MissingReadGroup</option>
                <option value="NoOriginalQualityScores">NoOriginalQualityScores</option>
                <option value="NotPrimaryAlignment">NotPrimaryAlignment</option>
                <option value="Platform454">Platform454</option>
                <option value="Platform">Platform</option>
                <option value="PlatformUnit">PlatformUnit</option>
                <option value="ReadGroupBlackList">ReadGroupBlackList</option>
                <option value="ReadName">ReadName</option>
                <option value="ReadStrand">ReadStrand</option>
                <option value="ReassignMappingQuality">ReassignMappingQuality</option>
                <option value="Sample">Sample</option>
                <option value="SingleReadGroup">SingleReadGroup</option>
                <option value="UnmappedRead">UnmappedRead</option>
              </param>
              <when value="BadCigar">
                  <!-- no extra options -->
              </when>
              <when value="BadMate">
                  <!-- no extra options -->
              </when>
              <when value="DuplicateRead">
                  <!-- no extra options -->
              </when>
              <when value="FailsVendorQualityCheck">
                  <!-- no extra options -->
              </when>
              <when value="MalformedRead">
                  <!-- no extra options -->
              </when>
              <when value="MappingQuality">
                  <param name="min_mapping_quality_score" type="integer" value="10" label="Minimum read mapping quality required to consider a read for calling"/>
              </when>
              <when value="MappingQualityUnavailable">
                  <!-- no extra options -->
              </when>
              <when value="MappingQualityZero">
                  <!-- no extra options -->
              </when>
              <when value="MateSameStrand">
                  <!-- no extra options -->
              </when>
              <when value="MaxInsertSize">
                  <param name="maxInsertSize" type="integer" value="1000000" label="Discard reads with insert size greater than the specified value"/>
              </when>
              <when value="MaxReadLength">
                  <param name="maxReadLength" type="integer" value="76" label="Max Read Length"/>
              </when>
              <when value="MissingReadGroup">
                  <!-- no extra options -->
              </when>
              <when value="NoOriginalQualityScores">
                  <!-- no extra options -->
              </when>
              <when value="NotPrimaryAlignment">
                  <!-- no extra options -->
              </when>
              <when value="Platform454">
                  <!-- no extra options -->
              </when>
              <when value="Platform">
                  <param name="PLFilterName" type="text" value="" label="Discard reads with RG:PL attribute containing this string"/>
              </when>
              <when value="PlatformUnit">
                  <!-- no extra options -->
              </when>
              <when value="ReadGroupBlackList">
                  <!-- no extra options -->
              </when>
              <when value="ReadName">
                  <param name="readName" type="text" value="" label="Filter out all reads except those with this read name"/>
              </when>
              <when value="ReadStrand">
                  <param name="filterPositive" type="boolean" truevalue="--filterPositive" falsevalue="" label="Discard reads on the forward strand"/>
              </when>
              <when value="ReassignMappingQuality">
                  <param name="default_mapping_quality" type="integer" value="60" label="Default read mapping quality to assign to all reads"/>
              </when>
              <when value="Sample">
                  <param name="sample_to_keep" type="text" value="" label="The name of the sample(s) to keep, filtering out all others"/>
              </when>
              <when value="SingleReadGroup">
                  <param name="read_group_to_keep" type="integer" value="76" label="The name of the read group to keep, filtering out all others"/>
              </when>
              <when value="UnmappedRead">
                  <!-- no extra options -->
              </when>
            </conditional>
        </repeat>
        <repeat name="input_interval_repeat" title="Operate on Genomic intervals" help="-L,--intervals &amp;lt;intervals&amp;gt;">
          <param name="input_intervals" type="data" format="bed,gatk_interval,picard_interval_list,vcf" label="Genomic intervals" />
        </repeat>
        <repeat name="input_exclude_interval_repeat" title="Exclude Genomic intervals" help="-XL,--excludeIntervals &amp;lt;excludeIntervals&amp;gt;">
          <param name="input_exclude_intervals" type="data" format="bed,gatk_interval,picard_interval_list,vcf" label="Genomic intervals" />
        </repeat>
        
        <param name="interval_set_rule" type="select" label="Interval set rule" help="-isr,--interval_set_rule &amp;lt;interval_set_rule&amp;gt;">
          <option value="UNION" selected="True">UNION</option>
          <option value="INTERSECTION">INTERSECTION</option>
        </param>
        
        <conditional name="downsampling_type">
          <param name="downsampling_type_selector" type="select" label="Type of reads downsampling to employ at a given locus" help="-dt,--downsampling_type &amp;lt;downsampling_type&amp;gt;">
            <option value="NONE" selected="True">NONE</option>
            <option value="ALL_READS">ALL_READS</option>
            <option value="BY_SAMPLE">BY_SAMPLE</option>
          </param>
          <when value="NONE">
              <!-- no more options here -->
          </when>
          <when value="ALL_READS">
              <conditional name="downsample_to_type">
                  <param name="downsample_to_type_selector" type="select" label="Downsample method">
                      <option value="downsample_to_fraction" selected="True">Downsample by Fraction</option>
                      <option value="downsample_to_coverage">Downsample by Coverage</option>
                  </param>
                  <when value="downsample_to_fraction">
                      <param name="downsample_to_value" type="float" label="Fraction [0.0-1.0] of reads to downsample to" value="1" min="0" max="1" help="-dfrac,--downsample_to_fraction &amp;lt;downsample_to_fraction&amp;gt;"/>
                  </when>
                  <when value="downsample_to_coverage">
                      <param name="downsample_to_value" type="integer" label="Coverage to downsample to at any given locus" value="0" help="-dcov,--downsample_to_coverage &amp;lt;downsample_to_coverage&amp;gt;"/>
                  </when>
              </conditional>
          </when>
          <when value="BY_SAMPLE">
              <conditional name="downsample_to_type">
                  <param name="downsample_to_type_selector" type="select" label="Downsample method">
                      <option value="downsample_to_fraction" selected="True">Downsample by Fraction</option>
                      <option value="downsample_to_coverage">Downsample by Coverage</option>
                  </param>
                  <when value="downsample_to_fraction">
                      <param name="downsample_to_value" type="float" label="Fraction [0.0-1.0] of reads to downsample to" value="1" min="0" max="1" help="-dfrac,--downsample_to_fraction &amp;lt;downsample_to_fraction&amp;gt;"/>
                  </when>
                  <when value="downsample_to_coverage">
                      <param name="downsample_to_value" type="integer" label="Coverage to downsample to at any given locus" value="0" help="-dcov,--downsample_to_coverage &amp;lt;downsample_to_coverage&amp;gt;"/>
                  </when>
              </conditional>
          </when>
        </conditional>
        <param name="baq" type="select" label="Type of BAQ calculation to apply in the engine" help="-baq,--baq &amp;lt;baq&amp;gt;">
          <option value="OFF" selected="True">OFF</option>
          <option value="CALCULATE_AS_NECESSARY">CALCULATE_AS_NECESSARY</option>
          <option value="RECALCULATE">RECALCULATE</option>
        </param>
        <param name="baq_gap_open_penalty" type="float" label="BAQ gap open penalty (Phred Scaled)" value="40" help="Default value is 40. 30 is perhaps better for whole genome call sets. -baqGOP,--baqGapOpenPenalty &amp;lt;baqGapOpenPenalty&amp;gt;" />
        <param name="use_original_qualities" type="boolean" truevalue="--useOriginalQualities" falsevalue="" label="Use the original base quality scores from the OQ tag" help="-OQ,--useOriginalQualities" />
        <param name="default_base_qualities" type="integer" label="Value to be used for all base quality scores, when some are missing" value="-1" help="-DBQ,--defaultBaseQualities &amp;lt;defaultBaseQualities&amp;gt;"/>
        <param name="validation_strictness" type="select" label="How strict should we be with validation" help="-S,--validation_strictness &amp;lt;validation_strictness&amp;gt;">
          <option value="STRICT" selected="True">STRICT</option>
          <option value="LENIENT">LENIENT</option>
          <option value="SILENT">SILENT</option>
          <!-- <option value="DEFAULT_STRINGENCY">DEFAULT_STRINGENCY</option> listed in docs, but not valid value...-->
        </param>
        <param name="interval_merging" type="select" label="Interval merging rule" help="-im,--interval_merging &amp;lt;interval_merging&amp;gt;">
          <option value="ALL" selected="True">ALL</option>
          <option value="OVERLAPPING_ONLY">OVERLAPPING_ONLY</option>
        </param>
        
        <repeat name="read_group_black_list_repeat" title="Read group black list" help="-rgbl,--read_group_black_list &amp;lt;read_group_black_list&amp;gt;">
          <conditional name="read_group_black_list_type">
            <param name="read_group_black_list_type_selector" type="select" label="Type of reads read group black list">
              <option value="file" selected="True">Filters in file</option>
              <option value="text">Specify filters as a string</option>
            </param>
            <when value="file">
              <param name="read_group_black_list" type="data" format="txt" label="Read group black list file" />
            </when>
            <when value="text">
              <param name="read_group_black_list" type="text" value="tag:string" label="Read group black list tag:string" />
            </when>
          </conditional>
        </repeat>
        
        <param name="disable_experimental_low_memory_sharding" type="boolean" truevalue="--disable_experimental_low_memory_sharding" falsevalue="" label="Disable experimental low-memory sharding functionality." checked="False" help="--disable_experimental_low_memory_sharding"/>
        <param name="non_deterministic_random_seed" type="boolean" truevalue="--nonDeterministicRandomSeed" falsevalue="" label="Makes the GATK behave non deterministically, that is, the random numbers generated will be different in every run" checked="False"  help="-ndrs,--nonDeterministicRandomSeed"/>
        
      </when>
    </conditional>    
  </xml>
  <xml name="analysis_type_conditional">
    <conditional name="analysis_param_type">
      <param name="analysis_param_type_selector" type="select" label="Basic or Advanced Analysis options">
        <option value="basic" selected="True">Basic</option>
        <option value="advanced">Advanced</option>
      </param>
      <when value="basic">
        <!-- Do nothing here -->
      </when>
      <when value="advanced">
        <yield />
      </when>
    </conditional>
  </xml>
  <xml name="reference_source_selector_param">
    <param name="reference_source_selector" type="select" label="Choose the source for the reference list">
      <option value="cached">Locally cached</option>
      <option value="history">History</option>
    </param>
  </xml>
  <token name="@CITATION_SECTION@">------

**Citation**

For the underlying tool, please cite `DePristo MA, Banks E, Poplin R, Garimella KV, Maguire JR, Hartl C, Philippakis AA, del Angel G, Rivas MA, Hanna M, McKenna A, Fennell TJ, Kernytsky AM, Sivachenko AY, Cibulskis K, Gabriel SB, Altshuler D, Daly MJ. A framework for variation discovery and genotyping using next-generation DNA sequencing data. Nat Genet. 2011 May;43(5):491-8. &lt;http://www.ncbi.nlm.nih.gov/pubmed/21478889&gt;`_

If you use this tool in Galaxy, please cite Blankenberg D, et al. *In preparation.*

  </token>
</macros>
```


Exercises:
Can you use Macros to simplify the inclusion of common tool content between the various phases of the Hello World examples?


## 16:45 - 17:00

Open questions and Free play.

Some suggestions for exploration (http://wiki.galaxyproject.org/Admin/Tools/ToolConfigSyntax):
* Config files
* Validators
* Defining Datatypes and Metadata
  * Composite Datatypes
* Parameter sanitizers
* Advanced Data source tool configuration
* Dynamic Select parameters
* Customizing output attributes
  * Labels
  * output `<actions>` (e.g. see tools/filters/cutWrapper.xml)

## 17:00 session ends

<slot name="/events/gcc2013/footer" />
