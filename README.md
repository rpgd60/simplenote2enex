
# Export Notes from Simple Note to ENEX

## Usage and Examples

### Introduction

* simplenote2enex.py  is a Python (3.7) script to convert Notes from [Simple Note](www.simplenote.com), a popular multiplatform Note taking app, to [ENEX](https://evernote.com/blog/how-evernotes-xml-export-format-works/)  XML-based format.
  * Simple Note exports notes to a proprietary JSON format. 
  * ENEX ([Evernote Export](https://evernote.com/blog/how-evernotes-xml-export-format-works/)) is an XML-based format that appears to be some kind of a *lingua franca* in the Note application world. Many apps such as Evernote, Joplin, StandardNotes, [Apple Notes](https://support.apple.com/en-eg/HT205793) --even Simple Note itself-- can import notes in ENEX format.
* See the "Background Information" section  below for additional details.
* simplenote2enex requires Python 3.6+,  mainly because of extensive use of f-Strings.

### Usage

```
$ python simplenote2enex.py --help
usage: simplenote2enex.py [-h] --json-file JSON_FILE [--author AUTHOR]
                          [--create-title] [--title_size TITLE_SIZE]
                          [--tag-filter TAG_FILTER] [--match-tagged]
                          [--match-untagged] [--invert-match]
                          [--verbose-level VERBOSE_LEVEL] [--number NUM_NOTES]

optional arguments:
  -h, --help            show this help message and exit
  --json-file JSON_FILE
                        Simple Note export file (json) to be converted to ENEX
  --author AUTHOR       Specify an author for all converted notes
  --create-title        Attempt to create a title for each ENEX note from
                        first line of "Simple Note" notes
  --title_size TITLE_SIZE
                        Maximum size in characters of title - default 250
  --tag-filter TAG_FILTER
                        Comma-separated list of tags. Will convert notes
                        matching any tag in list
  --match-tagged        Convert tagged notes
  --match-untagged      Convert untagged notes
  --invert-match        Invert match after combining (OR) all other
                        filters/matching conditions
  --verbose-level VERBOSE_LEVEL
                        Verbose output level. Output to stderr. Default 0 - no
                        output
  --number NUM_NOTES    Number of notes to convert (Optional, default is
                        convert all notes)



```

### Usage Examples

* All examples assume the existence of a JSON file generated by exporting  from Simple Note  (Menu: File / Export Notes) .  A sample  file is available at the [repository]( https://github.com/rpgd60/simplenote2enex/blob/master/test1.json)
* Convert all notes in file *test1.json*;  attempt to generate a  <title>first line of note</title> from the first line in each note (default separator '\r\n');  fill ENEX 'author' field with 'John D.'.  Output to stdout.  Redirect to file 'all.test1.enex'.  [Sample available](https://github.com/rpgd60/simplenote2enex/blob/master/all.test1.enex)
* Note :  when importing the ENEX files into Joplin,  the default behavior appears to create a Joplin "notebook" with the name of the ENEX file.  

```bash
$ python simplenote2enex.py --json-file test1.json --author 'John D.'  --create-title  --verbose-level 1  >  all.test1.enex
Processing file: test1.json 
Notes author:  John D.
Active notes:   8
Converted 8 notes
```

* The generated sample ENEX file can be seen at  
* Note:  the next examples pipe the output to a grep command to highlight the filtering behavior.  
  * For the  output to be useful as ENEX file to be imported,  it should be redirected to a file (in these examples,  *somefile.enex*)
  * Thus we use the GNU *tee* command to pipe the output both to a file and to the grep command. 
* Convert all notes in file *test1.json* matching tags "tag1" and "tag3".  Redirect output to  somefile.enex

```bash
$ python simplenote2enex.py --json-file test1.json --author 'John D.'  --create-title --verbose-level 1 --tag-filter 'tag1,tag3'  | tee somefile.enex |  grep -E "<tag>|<title>"
Processing file: test1.json 
Notes author:  John D.
Active notes:   8
Converted 4 notes
<title>Test  - Markdown with single tag</title>
<tag>tag1</tag>
<title>Test  - Markdown with three tags</title>
<tag>tag1</tag>
<tag>tag2</tag>
<tag>tag3</tag>
<title>Test  - Markdown with 2 tags</title>
<tag>tag1</tag>
<tag>tag2</tag>
<title>Test  - Markdown with 4 tags</title>
<tag>tag1</tag>
<tag>tag2</tag>
<tag>tag3</tag>
<tag>tag4</tag>

```

* Invert match of previous command - convert only notes without 'tag1' or 'tag3'.  Note this also includes untagged notes in the conversion. 

```bash
$ python simplenote2enex.py --json-file test1.json --author 'John D.'  --create-title --verbose-level 1 --tag-filter 'tag1,tag3' --invert-match | tee somefile.enex  |  grep -E "<tag>|<title>"

Processing file: test1.json 
Notes author:  John D.
Active notes:   8
Converted 4 notes
<title>Test No Markdown plain test - One tag</title>
<tag>test1</tag>
<title>Test No Markdown plain test - NO tag</title>
<title>Test  - Markdown No tag</title>
<title>Test  - Markdown with single tag (tag4)</title>
<tag>tag4</tag>

```

* Convert all untagged Notes and notes with tag 'test1'

```bash
$ python simplenote2enex.py --json-file test1.json --author 'John D.'  --create-title --verbose-level 1 --tag-filter 'test1' --match-untagged | tee somefile.enex  |  grep -E "<tag>|<title>"

Processing file: test1.json 
Notes author:  John D.
Active notes:   8
Converted 3 notes
<title>Test No Markdown plain test - One tag</title>
<tag>test1</tag>
<title>Test No Markdown plain test - NO tag</title>
<title>Test  - Markdown No tag</title>

```

### Revisions - bug and functionality fixes

- Issue #2 - title not generated if note content started with the "line separator" ("\r\n").  Modified to remove whitespace and one or more sequences of "\r\n" at the beginning or the end of the note content.
- Unsolicited feature:  add a cli parameter to control the maximum size of generated note title (default value 250 chars)

### Importing generated ENEX files to other applications  - Joplin

* I have successfully imported into Joplin an ENEX file with  500+ notes,  generated from my Simple Note deployment.  

  * I tested the import  with joplin CLI (*joplin import --log-level debug  thefile.enex*) , and found two hiccups, both related to having "&" in the Note title;  solved by changing the character to "-".  I will inquire about this bug/feature in the Joplin forum)
  * Once sorted I imported from the Joplin desktop application without a glitch :  File / Import / ENEX - Evernote Export File (as Markdown)  -  

* Importing with the Joplin desktop application appears to hang if for some reason it cannot process some element due to some encoding error or problem, such as the one reported above.

  * "Troubleshooting importing ENEX files is made  simpler if using joplin's  CLI application, available at https://joplinapp.org/terminal/
    * IMPORTANT note:  It appears that --even if installed in the same system -- the desktop Joplin application and the CLI command Joplin use different databases and in effect manage a different set of notes.    
  * One option is to use the CLI application for testing / troubleshooting if the desktop/GUI Joplin application hangs when importing a specific ENEX file.  Once all the offending notes have been identified and modified,   perform the import procedure from the Joplin GUI app.
  * Alternatively, it is possible to use Joplin's debugging functionality documented at:  https://github.com/laurent22/joplin/blob/master/readme/debugging.md

  

  

## Background Information

## Simple Note

### General Info

* Simple Note (www.simplenote.com) is a popular Note taking and synchronization application, developed by Automattic (sic), the makers of WordPress.

* Simple Note is indeed simple and works seamlessly across the main platforms and operating systems:  Mac, Windows, Linux, Web App, Android and IOS. It does pretty well what it says in the can.

* I have been using Simple Note since early 2018 (Linux, Windows and Android) and I have about 600 notes. I am very happy with this application and intend to continue using it.
  * The main advantages I see are the robust synchronization between mobile and desktop/web (based on Automattic's Simperium Library (www.simperium.com)) and  the simplicity of the interface.  
  * The main drawback, in my opinion, is the lack of client side encryption. Lack of encryption limits its usability for me to "don't-care-if-it-becomes-public" items,  like cooking recipes and general note taking on books, music, films, technical tips, shopping lists, etc.    
  * Simple Note is only text based with support of markdown.  I am personally happy with that "limitation" since it fits well with a subset of my needs (I write my notes in very rudimentary markdown).  At the same time I want to explore  other Note applications with other options like embedding images.
  
* Since I intend to continue using Simple Note, one of my goals is to ensure there is a convenient and reliable way to share data between Simple Note and whatever additional Note application I end up using.

### Simple Note -  Exporting Notes

* Simple Note does not export Notes to ENEX format (thus the need for simplenote2enex).  It exports notes to a proprietary JSON format.
* Exporting Notes from Simple Note ('File/Export Notes' or 'Ctrl-Shift-E') generates a zip file with default name *notes.zip* 
* Extracting *notes.zip* (to, say, directory *notes*) yields the following directory structure :

```
  $ tree ./notes
  ./notes
  ├── source
  │   └── notes.json  <<--- the file processed by simplenote2enex
  ├── <Active Note 1 name>.txt
  ├── <Active Note 2 name>.txt
  │   (...)
  ├── <Active Note N name>.txt
  ├── trash
  │   ├── <Deleted note 1 name>.txt
  │   ├── <Deleted note 2 name>.txt
  │   │   (...)
  │   ├── <Deleted note M name>.txt
```

  * The file *notes.json* contains two main sections (a 2-element JSON array.)
    * **activeNotes**  (focus of this utility) -  contains one JSON object per Active Note.  Each note will be read by simplenote2enex (json module) and processed as a "dict" 
    * **trashedNotes**  contains one JSON object per Deleted (trashed) Note. This section is not read at the moment by simpllenote2enex, although it would be a simple addition.

### Simple Note - Importing Notes

* Simple Note can also import notes in ENEX format.  I intend to explore this functionality to transfer data in the opposite direction.
  

## ENEX

* ENEX is the [Evernote](https://evernote.com/) xml-based import/export format for notes, used as some kind of "lingua franca" by many note-taking applications.
* See: https://evernote.com/blog/how-evernotes-xml-export-format-works/
* Example of ENEX file : https://gist.github.com/evernotegists/6116886

## Joplin 

* I am exploring other Note taking applications, starting with Joplin, "an open source note taking and to-do application with synchronization capabilities for Windows, macOS, Linux, Android and iOS" 
* Information about Joplin can be found at: 
  * Homepage: https://joplinapp.org/
  * GitHub: https://github.com/laurent22/joplin
  * Forum: https://discourse.joplinapp.org/
  
