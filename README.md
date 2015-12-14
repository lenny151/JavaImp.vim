JavaImp.vim
-----------
Short and sweet:  JavaImp generates and sorts your import statements so that
you don't have to.  It is also able to display JavaDoc HTML in a web browser of
your choice.

Features
--------
- Import Statement Management:
  - Semi-automatic insertion.
  - Sorting.
  - Styling/Organizing.
- Quick source lookup referenced classes.
- JavaDoc Display.

Requirements
------------
- Vim 7+ (with Python support) or Neovim 0.1.0+.
- The 'jar' binary must be your path.
- A web browser such as Chrome or Firefox or a pager such as w3m or lynx.

Installation
------------
You can use a modern Vim package manager such as Vundle or NeoBundle.  Simply add the following into your .vimrc.

    Plugin 'rustushki/JavaImp.vim'

Then run :PluginInstall.

You need to set two global variables in your .vimrc in order for this to work:

1. Paths to Java Project Source files.

        let g:JavaImpPaths =
	       \ $HOME . "/project/src/java," .
           \ $HOME . "/project2/javasrc," .
           \ $HOME . "/project3/javasrc"

   The g:JavaImpPaths is a comma separated list of paths that point to the
   roots of the 'com' or 'org' etc.  You can list your Java projects, external
   projects or even Java's source classes.  It is recommended that you do this
   so that you can take full advantage of automatic import statement
   generation.

   If ',' is not convenient for you, set g:JavaImpPathSep to the
   (single-character) separator you would like to use:

        let g:JavaImpPathSep = ':'

2. Path to JavaImp's temporary storage.  The default is:

	    let g:JavaImpDataDir = $HOME . "/vim/JavaImp"

Commands
========
Generate a cache of classes for import:

    :JavaImpGenerate or :JIG

If you have not created the directory for g:JavaImpDataDir yet, this will
create the appropriate paths.  JIG will go through your JavaImpPaths and search
for anything that ends with .java, .class, or .jar.  It'll then write the
mappings to the JavaImp.txt and/or the cache files.

After you've generated your JavaImp.txt file, move your cursor to a class name
in a Java file and do a:

    :JavaImp or :JavaImpSilent or :JI

And the magic happens!  You'll realize that you have an extra import statement
inserted after the last import statement in the file.  It'll also prompts you
with duplicate class names and insert what you have selected.  If the class
name is already imported, it'll do nothing.  

You can also sort the import statements in the file by doing:

    :JavaImpSort or :JIS

Source Viewing
--------------

JavaImp will try to find the source file of the class under your cursor by:

    :JavaImpFile or :JIF
    
Doing a :JavaImpFileSplit or :JIFS will open a split window on the file.

JavaDoc Viewing
---------------
If you want to use the JavaDoc viewing feature for JavaImp, you should set
g:JavaImpDocPaths.  Similar to how you set the g:JavaImpPaths,
g:JavaImpDocPaths contains a list of root level directories that contains your
java docs.  This, together with a HTML pager (like w3m or lynx on Unix), let
you view the JavaDocs very quickly by just hitting :JID on a class name.  For
example, you can set:

    let g:JavaImpDocPaths = "/usr/java/docs/api," .
       \ "/project/docs/api"

The default pager is set to:

    let g:JavaImpDocViewer = "w3m"

On windows, you can put iexplore.exe or mozilla.exe in your path and set the
g:JavaImpDocViewer to "iexplore.exe" or "mozilla.exe".  Note that windows'
shell and spaces in the path don't mix. Thus, given an absolute path of the
HTML viewer to g:JavaImpDocViewer may not work.

If you have your g:JavaImpDocPaths and g:JavaImpDocViewer set correctly You can
then do this with your cursor on a classname:

    :JavaImpDoc or :JID 

JavaImp will find your class accordingly and open the viewer to the class based
on the import list that you've generated by :JIG.  You can also set in your
java.vim  filetype plugin to get a similar man page behavior by press "K":

    nmap <buffer> K :JID<CR>

Import Statement Order
----------------------
JavaImp will order your import statements into groups:
* Statics Imports (if configured to come first)
* Top Imports
* Middle Imports
* Bottom Imports
* Statics Imports (if configured to come last)

Static import statements may come first or last.  The default is to place them
above the regular imports.  You can override this by setting:

	let g:JavaImpStaticImportsFirst = 0

Top import statements come next.  These are normal import statements which
match a prioritized list of regular expressions.  JavaImp uses similar setting
to Eclipse Mars by default:

	let g:JavaImpTopImports = [
		\ 'java\..*',
		\ 'javax\..*',
		\ 'org\..*',
		\ 'com\..*'
		\ ]

Next come the Middle Imports Statements.  These are any import statements which
are not static and do not match the top nor bottom import statement regular
expressions.

Bottom Import Statements appear below the Middle Import Statements.  These
statements will match a configured list of regular expressions.  By default,
this list is empty:

	let g:JavaImpBottomImports = []

By default, JavaImp will insert a blank line among package group with package
root for 2 similar levels.  For example, the import of the following:

    import java.util.List;
    import org.apache.tools.zip.ZipEntry;
    import javax.mail.search.MessageNumberTerm;
    import java.util.Vector;
    import javax.mail.Message;
    import org.apache.tools.ant.types.ZipFileSet;

will become:

    import java.util.List;
    import java.util.Vector;

    import javax.mail.Message;
    import javax.mail.search.MessageNumberTerm;

    import org.apache.tools.ant.types.ZipFileSet;
    import org.apache.tools.zip.ZipEntry;

Note the classes that begins similar package root with two beginning levels of
package hierarchy are stuck together.  You can set the g:JavaImpSortPkgSep to
change this behavior.  The default g:JavaImpSortPkgSep is set to 2.  Do not set
it too high though for you'll insert a blank line after each import.  If you do
not want to insert blank lines among the imports, set:

    let g:JavaImpSortPkgSep = 0

Please note that this setting only applies to non-static imports.  Static
Imports will not have a package separator of any kind.

Extras
------
After you have generated the JavaImp.txt file by using :JIG, you can use it as
your dictionary for autocompletion.  For example, you can put the following in
your java.vim ftplugin (note here g:JavaImpDataDir is set before running this):

    exe "setlocal dict=" . g:JavaImpDataDir . "/JavaImp.txt"
    setlocal complete-=k
    setlocal complete+=k

or put this in your .vimrc

    exe "set dict=" . g:JavaImpDataDir . "/JavaImp.txt"
    set complete-=k
    set complete+=k

After you have done so, you can open a .java file and use ^P and ^N to
autocomplete your Java class names.

Importing your JDK Classes
--------------------------
JavaImp also support a file type called "jmplst".  A jmplst file essentially
contains the output of a "jar tf" command.  It is mainly used in the case where
you want to import some external classes in JavaImp but you do not have the
source directory nor the jar file.  For example, you can import the JDK
classes, which can be located in $JAVA_HOME/src.jar (different in different
distributions) with your JDK distribution, by using the jmplst file:

To expose the standard JDK classes to JavaImp:

1. Generate your jmplst file.

        # If you have "sed":
        $ jar tf $JAVA_HOME/src.jar | sed -e 's#^src/##' > jdk.jmplst
        
        # If you do not have "sed":
        $ jar tf $JAVA_HOME/src.jar > jdk.jmplst
        $ vim jdk.jmplst

        # Execute the following vim commands:
        1G0<C-v>G3ld:w

    This will select vertically the all the "src/" prefixes and delete them, then
    save the file. Essentially, we want to get rid of the src directory otherwise
    it'll screw up the import statements.

2. Put the jdk.jmplst in a directory that you've added in your g:JavaImpPaths.
For example, I put my jdk.jmplst in $HOME/vim/JavaImp/jmplst directory, and add
$HOME/vim/JavaImp/jmplst to g:JavaImpPaths.

3. Open vim with the JavaImp.vim loaded and Do a :JIG.  You should see that
JavaImp will pick up many more classes.

4. Try to do a :JI on a "Vector" class, for example, to see whether you can add
the import statement from the JDK.

Enjoy!

History
-------
This script is descended from [Vim Script #325]
(http://www.vim.org/scripts/script.php?script_id=325) originally written by
William Lee.  I work with Java on a regular basis and found it handy.  Ten
years of dust left some shortcommings, and so I decided to fork it earlier this
year.  At first I used these changes privately, but now I've decided to publish
them so that others can benefit.

Credits
--------
William Lee &lt;wl1012@yahoo.com&gt;
(c) 2002-2004. All Rights Reserved

Russ Adams (rustushki)

Thanks
------
William Lee for an excellent and most useful Vim plugin.

Eric Y. Kow for his bug fixes, addition on jar support, elimination of
duplicates, and Unix sorting implementation.

Robert Webb for his sorting function.

Matt Paduano, Toby Allsopp, Dirk Duehr, Adam Hawthorne, and others for their
bug fixes/patches.
