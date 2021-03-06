SHARPLINTER 1.0

https://github.com/jamietre/SharpLinter

(c) 2011 James Treworgy

This is based on code written by Luke Page that was the predecessor
to his extension "JSLint for Visual Studio 2010:"

http://www.scottlogic.co.uk/2010/09/js-lint-in-visual-studio-part-1/

### Code use license.

LICENSE (MIT License)
 
Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:
 
The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Summary

SharpLinter is a command line tool to automate error-checking Javascript files. It produces output that is formatted for 
Visual Studio's output window, so clicking on a line will locate the file and line in the IDE.

Each of the linting options can be specified in a configuration file as well (see below).

## Roadmap for Distribution

All these files should be copied into a folder from where it will be invoked.

    bin/SharpLinter.exe                     The executable.
    bin/JTC.SharpLinter.dll                 The class library (if you want to use this from within another app)
    bin/Noesis.Javascript.dll               Noesis Javascript.NET V8 runtime
    bin/Yahoo.Yui.Compressor.dll            Yahoo YUI Compressor for .NET
    bin/EcmaScript.NET.modified.dll         Distributed with Yahoo YUI

These are needed by the Noesis Javascript V8 wrapper:
    
    bin/msvcp100.dll                        MS VC++ runtimes. You probably do not need these, but if you have problems 
    bin/msvcr100.dll                           try copying them to the Windows system folder.                       
    
Noesis project: <http://javascriptdotnet.codeplex.com/>
Yahoo YUI project: <http://yuicompressor.codeplex.com/>

Basic usage: Process all files in the current directory:

    sharplinter -f *.js    

Detailed usage:

    
    sharplinter [-o options] [-v]
            [-c sharplinter.conf] [-j jslint.js] [-y]
            [-p[h] yui|packer|best mask] [-k]
            [-i ignore-start ignore-end] [-if text] [-of "format"]
            [-[r]f /path/*.js] [filemame1 filename2 ...] 

    Options:


    -[r]f c:\scripts\*.js     parse all files matching "*.js" in "c:\scripts"
                              if called with "r", will recurse subfolders
                              The "f" is optional, any parameters passed 
                              without an option will be treated as files.
                              
    -o "option option ..."    set jslint/jshint options specified, separated by
                              spaces, in format "option" or "option: true|false"
                              
    -v                        be verbose (report information other than errors)

    -k                        Wait for a keytroke when done
    
    -c c:\sharplinter.conf    load config options from file specified. if omitted, will 
                              attempt to read a file 'sharplinter.conf' in the application
                              directory
    
    -j jslint.js              use file specified to parse files instead of embedded
                              (probably old) script. if omitted, will attempt first to
                              read the file 'jslint.js' from the application director,
                              and will fall back on the embedded file.
                              
    -y                        Also run the script through YUI compressor to look for
                              errors
                              
    -i text-start text-end    Ignore blocks bounded by /*text-start*/ and
                              /*text-end*/
                              
    -if text-skip             Ignore files that contain /*text-skip*/ anywhere
    
    -of "output format"       Use the string as a format for the error output. The
                              default is:
                              "{0}({1}): ({2}) {3} at character {4}". The parms are
                              {0}: full file path, {1}: line number, {2}: source
                              (lint or yui), {4}: character

    -p[h] yui|packer|best *.min.js      Pack/minimize valid input using YUI
                                    Compressor, Dean Edwards' JS Packer, or
                              whichever produces the smallest file. Output to a
                              file "filename.min.js". If validation fails,
                              the output file will be deleted (if exists)
                              to ensure no version mismatch. If  -h is specified,
                              the first comment block in the file /* ... */
                              will be passed uncompressed at the beginning of the
                              output.

## JSLINT/JSHINT options

The options passed with -o should be of the format:

    "[option : value | option][,option: value | option]..."

Most options (except maxlen) are boolean, so the "value" is optional, in which case it will be set "true."
All options are false by default, and true disables or enables the specified behavior. In JSHINT, some options
have the opposite behavior as their JSLINT counterpart, and setting them "true" will actually enable rather
than disable a particular validation.


## Config File Format

A basic file is included. The file has up four sections. Each section is structured like:

    /*sectiontitle  
    ...
    sectiontitle*/


Anything else in the file that is not found between these constructs will be ignored. Valid section names are jslint, global, exclude, and sharplinter. An example:

    // set global options

    /*jslint 
        browser: true, 
        wsh: true, 
        laxbreak:true,
        evil: true,
        eqeqeq: true,
        curly: true,
        forin: true,
        immed: true,
        newcap: true,
        nomen: false,
        onevar: true,
        undef: true,
        jquery: true	
    jslint*/

    // define some globals 

    /*global 
	mynamespace,alert,confirm	
    global*/

    // define paths for file exclusions. Wild cards only work for the file name part of the path, not the 
    // folders. Pattern matching within the path is therefore very simple, but wildcard matching for file names works

    /*exclude
        *.min.js
        /temp/
        /test/
        /google-tracker.js
        /thirdp/
    exclude*/

    // this section has only three options, as per below. The first two define a construct for ignoring blocks
    // within files. The text used, when found tightly wrapped in a comment block like:
       /*ignore-start*/
           //...
       /*ignore-end*/
    // will cause everything between the markers to be ignored.
    // The final option, ignorefile, if found anywhere in a script will cause the entire file to be ignored.

    /*sharplinter
        ignorestart: ignore-start,
        ignoreend: ignore-end,
        ignorefile: lint-ignore-file
    sharplinter*/


# Visual Studio Integration

SharpLinter produces output that Visual Studio can use to take you to the file and line associated with each error. It's not an extension, so you need
to configure it as an external tool. Do the following:

1. Go to Tools -> External Tools
2. Create a new tool with title "SharpLinter" (or whatever you like).
3. Set "Command" to the path of the executable.
4. Set arguments to something like this:

    -ph best *.min.js -f $(ItemPath) 
    
    This will process the active file and compress it to filename.min.js if validation passes.

5. Check "Output Window"

You are now done. You can lint a file with Tools-> SharpLinter. If you want to make life even easier, add a keyboard shortcut:

6. Go to Tools -> Options -> Environment -> Keyboard
7. Type "externalcommand" where it says "show commands containing:"
8. All the external commands appear as "Tools.ExternalCommand1" through "Tools.ExternalCommand20". These numbers are the ordinal order in which they appear in your list. Choose the right one.
9. Click on "Press shortcut keys" and type whatever you want. I like Alt+z. You may have to remove it from something else first.

You can also set it up to process all the files in your project. Make another entry in External Tools, but instead of specifying the active file, use a pattern in your file spec, and add "r" to recurse subfolders:

    -rf $(ProjectDir)/scripts/*.js -x *.min.js -x /thirdparty/

for example, to process all files in the "scripts" subfolder of your project, but skip the folder "thirdparty" within "scripts" and don't try to process minimized files.

# TextPad

SharpLinter can also be configured as an external tool in the Textpad editor.

1. Configure -> Preferences -> Tools and click "Add -> DOS Command"
2. Enter the full path to sharplinter.exe
3. Select the new tool from Preferences tree end edit options.
4. Edit "Parameters" so it is a complete command line with any options you want.
   Use "$File" for the current file, e.g.
   
    C:\Program Files\SharpLinter\SharpLinter.exe --ph best *.min.js -r $File

5. Enter the following regular expression:

    ^\(.*\)(\([0-9]+\))
   
6. Choose "1" for File, and "2" for Line under the regex input.

Now you're done. Set up a keyboard shortcut, and it will process the active file. You can click a line in the output window to jump to it in the editor.