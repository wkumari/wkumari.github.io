# TLDs that are also file extensions

There has recently been some concerns expressed around new TLDs colliding with file extensions. While I'm the first to raise concerns around collisions, I think that this particular ship has sailed. As an example, the DEC VAX and CP/M used .COM as a file extension. This was then carried over into MS-DOS (and Windows) - a good example of this being `command.com`. While this confusion was briefly abused, but mitigations were quickly put in place. This is a great example of the contextual important of naming - `command.com` on your computer is a different context to https://www.command.com - the 3M Command Adhesive website.



However, before completely dismissing the threat, I figured I'd quickly look and see how many collisions there are between well known file extensions and delegated TLDs. I used the [Webopedia List Of Data Formats & File Extensions](https://www.webopedia.com/reference/data-formats-and-file-extensions/) and wrote a small Python program to compare these with the [IANA list of delegated TLDs](https://data.iana.org/TLD/tlds-alpha-by-domain.txt)



Here are the results:


| Extension | Description |
|---------------|-------------------|
| abc | ActionScript Byte Code File |
| ad | Screen saver data (AfterDark) |
| ads | Ada Package Specification |
| afl | Font file (for Allways) (Lotus 1-2-3) |
| ag | Applixware graphics file |
| ai | Vector graphics (Adobe Illustrator) |
| app | Add-in application file (Symphony) |
| art | Graphics (scrapbook) (Art Import) |
| au | Sound (audio) file (SUN Microsystems) |
| aw | Text file (HP AdvanceWrite) |
| aws | Data (STATGRAPHICS) |
| ax | DirectShow Filter |
| bar | Horizontal bar menu object file (dBASE Application Generator) |
| bb | Database backup (Papyrus) |
| bcn | Business Card Pro Design |
| bid | BidMaker 2002 file |
| bio | OS/2 BIOS |
| bm | Bitmap graphics |
| bom | MicroSim PCBoard Bill of Materials |
| boo | Compressed file ASCII archive created by BOO (msbooasm.arc) |
| book | Adobe FrameMaker Book |
| bot | Linkbot file |
| box | Myriad Jukebox file |
| br | Script (Bridge) |
| buy | Datafile format (movie) |
| ca | Initial cache data for root domain servers (Telnet) |
| cab | Cabinet File (Microsoft installation archive) |
| cal | Calendar file (Windows 3.x) |
| cam | Casio Camera Graphic |
| car | AtHome Assistant file |
| cat | Catalog (dBASE IV) |
| cc | C++ source code file |
| cf | Sendmail Configuration file |
| ch | Header file (Clipper 5) |
| cl | Common LISP source code file |
| cm | Data file (CraftMan) |
| com | Command (memory image of executable program) (DOS) |
| crs | File Conversion Resource (WordPerfect 5.1) |
| data | Sid Tune audio file |
| day | Journal file |
| de | MetaProducts Download Express incompletely downloaded file |
| dev | Device driver |
| do | ModelSim Filter Design HDL Coder |
| dog | Screen file (Laughing Dog Screen Maker) |
| dot | Line-type definition file (CorelDRAW) |
| dvr | Windows Media Center Recorded file |
| dz | Dzip Compressed file |
| eco | NetManage ECCO file |
| email | Outlook Express Mail Message |
| fi | Interface file (MS Fortran) |
| fit | Fits graphics |
| fm | Spreatsheet (FileMaker Pro) |
| frl | FormFlow file |
| gal | Corel Multimedia Manager Album |
| gb | Pagefox Bitmap Image file |
| gg | Google Desktop Gadget file |
| gl | Animation (GRASP GRAphical System for Presentation) |
| gp | Geode parameter file (Geoworks Glue) |
| ibm | Compressed file archive created by ARCHDOS (Internal IBM only) |
| id | Disk identification file |
| inc | Include file (several programming languages) |
| ink | Pantone reference fills file (CorelDRAW) |
| int | Borland Interface Units |
| io | Compressed file archive created by CPIO |
| ist | Digitrakker Instrument File (n-FaCToR) |
| it | Settings (intalk) |
| java | Java source code file |
| lat | Crossword Express Lattice file |
| man | Command manual |
| map | Color palette |
| md | Compressed file archive created by MDCD (mdcd10.arc) |
| me | Usually ASCII text file READ.ME |
| med | Macro Editor delete save (WordPerfect Library) |
| mk | Makefile |
| mlb | Macro library file (Symphony) |
| mm | Text file (MultiMate Advantage II) |
| mov | QuickTime Video Clip |
| msd | MS Diagnostic Utility Report |
| mu | Menu (Quattro Pro) |
| nc | Graphics (netcdf) |
| net | Network configuration/info file |
| new | New info |
| ng | Online documentation database (Norton Guide) |
| now | Text file |
| np | Project schedule (Nokia Planner) (Visual Planner 3.x) |
| nra | Nero Audio-CD Compilation |
| nrw | Nero WMA Compilation file |
| one | OneNote Document File |
| org | Calendar file (Lotus Organizer) |
| pa | Print Artist |
| pet | Program Editor top overflow file (WordPerfect Library) |
| pf | Windows Prefetch file |
| pg | Pagefox File |
| ph | Optimized .goh file (Geoworks) |
| pk | Packed bitmap font bitmap file (TeX DVI drivers) |
| pl | Perl source code file |
| pro | Prolog source code file |
| ps | PostScript file (text/graphics) (ASCII) |
| pub | Page template (MS Publisher) |
| pw | Text file (Professional Write) |
| py | Python script file |
| red | Path info (Clarion Modula-2) |
| rip | Graphics (Remote Access) |
| rs | Data file (Amiga Resource – Reassembler) |
| ru | JavaSoft Library file |
| run | RunScanner Saved file |
| sas | SAS System program |
| sb | Audio file (signed byte) |
| sbi | Sound Blaster Instrument file (Creative Labs) |
| sbs | SWAT HRU Output file |
| sc | Pal script (Paradox) |
| sca | Datafile (SCA) |
| sh | Unix shell script |
| skin | Skin file |
| sl | S-Lang source code file |
| sm | Smalltalk source code file |
| so | Apache Module file |
| spa | Macromedia FutureSplash file |
| ss | Bitmap graphics (Splash) |
| st | Smalltalk source code file (Little Smalltalk) |
| tab | Guitar Tablature file |
| tax | TurboTax file |
| tc | Configuration (Turbo C – Borland C++) |
| td | Configuration file (Turbo Debugger for DOS) |
| tdk | Keystroke recording file (Turbo Debugger) |
| tel | Host file (Telnet) |
| tf | Configuration (Turbo Profiler) |
| thd | Thread |
| tr | Session-state settings (Turbo Debugger for DOS) |
| tv | Table view settings (Paradox) |
| tz | Compressed file archive created by TAR and COMPRESS (.tar.Z) |
| vc | Include file with color definitions (Vivid 2.0) |
| vi | Graphics (Jovian Logic VI) |
| win | Opera Saved Window file |
| wow | Music (8 channels) (Grave Mod Player) |
| ws | Text file (WordStar 5.0-6.0) |
| xxx | Singer Sewing Machine Professional SewWare file |
| xyz | ASCII RPG Maker Graphic Format |
| zip | ZIP Compressed file archive |

**Total: 139**



Quick-n-dirty Python to generate this:

```python
#!/usr/bin/env python3

"""
PoC to find file extensions that are also valid TLDs
"""
import sys

def read_filelist(filename):
  file_types = {}
  with open(filename) as f:
    lines = f.readlines()
    for line in lines:
      if line.startswith('.'):
        (ext, desc) = line.split(' ', 1)
        ext = ext.strip('.')
        file_types[ext] = desc.strip()
  return file_types

def read_tlds(filename):
  tlds = []
  with open(filename) as f:
    lines = f.readlines()
    for line in lines:
      tlds.append(line.lower().strip())
  return tlds

def main():
  """pylint FTW"""
  tlds = read_tlds('tlds-alpha-by-domain.txt')
  extensions = read_filelist('file_extension_tlds.txt')
  print ("| Extension | Description |")
  print ("| --------- | ----------- |")
  for ext in extensions:
    if ext in tlds:
      print ("| %s | %s |" % (ext, extensions[ext]))


if __name__ == "__main__":
    main()
    sys.exit(0)

```

