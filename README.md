<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline1">1. Code</a></li>
<li><a href="#orgheadline5">2. Tests</a>
<ul>
<li><a href="#orgheadline2">2.1. Hello World</a></li>
<li><a href="#orgheadline3">2.2. Tables</a></li>
<li><a href="#orgheadline4">2.3. Compiler flags and command line args</a></li>
</ul>
</li>
</ul>
</div>
</div>


# Code<a id="orgheadline1"></a>

org-babel funtions for C# evaluation based on Eric Schulte&rsquo;s ob-java.el &#x2013; don&rsquo;t blame Eric for any harm this might cause, he has *nothing* to do with ob-csharp.el!  

    ;;; ob-csharp.el --- org-babel functions for csharp evaluation
    
    ;; Copyright (C) 2011-2015 Free Software Foundation, Inc.
    
    ;; Original Author: Eric Schulte (ob-java.el) 
    ;; Author: thomas "at" friendlyvillagers.com 
    ;; Keywords: literate programming, reproducible research
    ;; Homepage: http://orgmode.org
    
    ;; This file is NOT YET part of GNU Emacs.
    
    ;; GNU Emacs is free software: you can redistribute it and/or modify
    ;; it under the terms of the GNU General Public License as published by
    ;; the Free Software Foundation, either version 3 of the License, or
    ;; (at your option) any later version.
    
    ;; GNU Emacs is distributed in the hope that it will be useful,
    ;; but WITHOUT ANY WARRANTY; without even the implied warranty of
    ;; MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    ;; GNU General Public License for more details.
    
    ;; You should have received a copy of the GNU General Public License
    ;; along with GNU Emacs.  If not, see <http://www.gnu.org/licenses/>.
    
    ;;; Commentary:
    
    ;; Currently this only supports the external compilation and execution
    ;; of csharp code blocks (i.e., no session support).
    
    ;;; Code:
    (require 'ob)
    
    (defvar org-babel-tangle-lang-exts)
    (add-to-list 'org-babel-tangle-lang-exts '("csharp" . "cs"))
    
    (defcustom org-babel-csharp-command "mono"
      "Name of the csharp command.
    May be either a command in the path, like mono
    or an absolute path name, like /usr/local/bin/mono
    parameters may be used, like mono -verbose"
      :group 'org-babel
      :version "24.3"
      :type 'string)
    
    (defcustom org-babel-csharp-compiler "gmcs"
      "Name of the csharp compiler.
    May be either a command in the path, like mcs
    or an absolute path name, like /usr/local/bin/mcs
    parameters may be used, like mcs -warnaserror+"
      :group 'org-babel
      :version "24.3"
      :type 'string) 
    
    
    (defun org-babel-execute:csharp (body params)
      (let* ((full-body (org-babel-expand-body:generic body params))
             (cmpflag (or (cdr (assoc :cmpflag params)) ""))
             (cmdline (or (cdr (assoc :cmdline params)) ""))
             (src-file (org-babel-temp-file "csharp-src-" ".cs"))
             (exe-file (concat (file-name-sans-extension src-file)  ".exe"))
             (compile 
              (progn (with-temp-file  src-file (insert full-body))
                     (org-babel-eval 
                      (concat org-babel-csharp-compiler " " cmpflag " "  src-file) ""))))
        (let ((results (org-babel-eval (concat org-babel-csharp-command " " cmdline " " exe-file) "")))
          (org-babel-reassemble-table
           (org-babel-result-cond (cdr (assoc :result-params params))
             (org-babel-read results)
             (let ((tmp-file (org-babel-temp-file "c-")))
               (with-temp-file tmp-file (insert results))
               (org-babel-import-elisp-from-file tmp-file)))
           (org-babel-pick-name
            (cdr (assoc :colname-names params)) (cdr (assoc :colnames params)))
           (org-babel-pick-name
            (cdr (assoc :rowname-names params)) (cdr (assoc :rownames params)))))))
    
    (defun org-babel-prep-session:csharp (session params)
      "Return an error because csharp does not support sessions."
      (error "Sessions are not (yet) supported for CSharp"))
    
    
    (provide 'ob-csharp)
    ;;; ob-csharp.el ends here

Put this file into your orgmode-folder (i. e. `.emacs.d/elpa/org-20XXXXXX`). 

In your `.emacs`-file add csharp to the list of babel languages, i. e.: 

    (org-babel-do-load-languages 'org-babel-load-languages '((csharp . t)))

# Tests<a id="orgheadline5"></a>

## Hello World<a id="orgheadline2"></a>

    class HelloWorld {
      public static void Main()
      {
        System.Console.WriteLine("Hello World!");
      }
    }

    Hello World!

## Tables<a id="orgheadline3"></a>

    class Table {
      public static void Main()
      {
        for (char  c = 'a'; c < 'd'; c++)
          System.Console.Write("{0} ",c);
        System.Console.WriteLine();
        for (int i = 0; i < 3; i++)
          System.Console.Write("{0} ",i);
      }
    }

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-right" />

<col  class="org-right" />

<col  class="org-right" />
</colgroup>
<tbody>
<tr>
<td class="org-right">a</td>
<td class="org-right">b</td>
<td class="org-right">c</td>
</tr>


<tr>
<td class="org-right">0</td>
<td class="org-right">1</td>
<td class="org-right">2</td>
</tr>
</tbody>
</table>

## Compiler flags and command line args<a id="orgheadline4"></a>

    public class TestFlags {
      public static void Main()
      {
       int i;  // unused; throw compile time error
       System.Console.WriteLine("You won't see this!");
      }
    }

    public class TestCmd {
      public static void Main()
      {
       System.Console.WriteLine("You won't see this!");
      }
    }

    Mono JIT compiler version 3.2.8 (Debian 3.2.8+dfsg-10)
    Copyright (C) 2002-2014 Novell, Inc, Xamarin Inc and Contributors. www.mono-project.com
            TLS:           __thread
            SIGSEGV:       altstack
            Notifications: epoll
            Architecture:  amd64
            Disabled:      none
            Misc:          softdebug 
            LLVM:          supported, not enabled.
            GC:            sgen

    public class TestForms {
      public static void Main()
      {
        System.Windows.Forms.MessageBox.Show("Hello Messagebox", "Hello"); 
      }
    }