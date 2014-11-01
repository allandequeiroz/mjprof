MJProf
=======
MJProf is a command line monadic java profiler.

Introduction
=============
MJProf is a Monadic jstack analysis tool set. It is a fancy way to say it analyzes jstack output using a series of simple composable building blocks (monads).

Motivation
==========
So, You are out there in the wild vs a production machine. All you have have in hand is the 'poor's man profiler': jstack.
You take one, two, three stack dumps and then you need to look at them manually inside an editor, vi or less etc....
If you done it enough you will probably know that it is a lot of manual work. Especially when you have thousands of threads in your process.



Running MJProf
===========
MJProf reads thread dumps from one or more data sources and writes to standard output.  An example of a data source can be standard input. 
Example: The following example will filter out all the threads which are not in RUNNABLE state.  
`jstack -l pid | ./mjprof.sh contains/state,RUNNABLE/`  
The commands passed to mjprof consists of several building blocks, monads from now on, concatenated with . (comma)
While the monads are relatively simple mixing and matching them can result a very powerful 
analysis, that will enable you focus on the data you need.
Parameters to monads are wrapped with / (instead of () {} or [] which are special chars in the shell) and seperated by ,


Monads 
======

Data Sources
------------
Data sources generate thread dumps and feed them into MJProf. The default data source is stdin. When no data source is specified stdin will be used.
When more than one data source is specified the thread dumps generated by all of them will be fed into MJProf.
* _**stdin**_  - Reads a sequence of thread dumps in a textual format which is similar to jstack format from standard input  
* _**path**/file/_  - Reads a sequence of thread dumps  a file. 
* _**jmx**/host,port,recording-time,sampling-frequency,user,password/_  - Reads a sequence of thread dumps directly from JMX  
* _**jstack**/pid,recording-time,sampling-frequency/_  - Reads a sequence of thread dumps  by running jstack command line  
* _**visualvm/file/_  - read profiling data from visualvm xml export - *****not implemented yet*****

Filters
-------
* _**contains**/attr,string/_  - returns only threads which contains the string (regexp not supported)
* _**ncontains**/attr,string/_  - returns only threads which do no contains the string(regexp not supported)

Mappers
-------
* _**eliminate**/attr/_         - Removes a certain attribute e.g. eliminate/stack/
* _**sort**/attr/_              - Sorts based on attribute
* _**sortd**/attr/_             - Sorts based on attribute (descending order)
* _**keeptop**/int/_            - Returns at most n top stack frames of the stack
* _**keepbot**/int/_            - Returns at most n bottom stack frames of the stack
* _**trimbelow**/int/_          - Trim all stack frames below the first occurance of string 
* _**stackelim**/string/_       - Elminate stackframes from all stacks which contain string
* _**stackkeep**/string/_       - Keep only stackframes from all stacks which contain string
* _**pkgelim**_                 - Eliminates package name from stack frames
* _**ifnelim**_                 - Eliminates file name and line number  from stack frames
* _**noop**_                     - Do nothing.

Terminals
---------
* _**count**_                   - Counts number of threads
* _**tree**_                    - Creates a profiling tree based on all stack traces
* _**list**_                    - List the stack trace attributes
* _**group**/attr/_             - Group by a certain attribute

Properties
----------
Properties may change from one dump to another and the can also be eliminated by **mjstack**.
Following is the list of usual properties  
* _**status**_          - The status of the thread
* _**nid**_             - Native thread id ( a number)
* _**name**_            - Name of thread
* _**state**_           - State of thread
* _**los**_             - The locked ownable synchronizers part of the stack trace
* _**daemon**_          - Whether the thread is a daemon or not
* _**tid**_             - The thread id (a number)
* _**prio**_            - Thread priority, a number
* _**stack**_           - The actual stack trace

You can also get the actual list of properties bu using the list monad.  
`jstack -l pid | ./mjprof.sh list`

Examples
=============
jstack Original output:  
`jstack -l 38515 > mystack.txt`  
Keep only thread which their names contain ISCREAM:  
`jstack -l 38515  | ./mjprof.sh contains/name,ISCREAM/`  
Sort them by state  
`cat mystack.txt  | ./mjprof.sh contains/name,ISCREAM/.sort/state/`  
Eliminate the Locked Ownable Synchronizers Section  
`jstack -l 38515  | ./mjprof.sh contains/name,ISCREAM/.sort/state/.eliminate/los/`  
Shorten stack traces to include only 10 last stack frames  
`jstack -l 38515  | ./mjprof.sh contains/name,ISCREAM/.sort/state/.eliminate/los/.keeptop/10/`  
Count threads  
`jstack -l 38515  | ./mjprof.sh contains/name,ISCREAM/.sort/state/.eliminate/los/.keeptop/10/.count`



Building MJProf
=============
MJProf uses maven for compilation. In order to build MJProf use the following command line:  
`mvn clean install assembly:assembly`
This will create a zip file in `target/dist/mjstack1.0-bin.zip` which contains everything you need.


Writing a plugin
===============

MJProf monadic capabilities can be extended with plugins. If you feel something is missing you can write your own plugin. 
We will appreciate if you will contribute it back to the community. 
A plugin is a Java class which is annotated with the com.performizeit.mjstack.plugin.Plugin. The annotation accepts three parameters. "name" the
plugin name (must be unique) paramTypes the expected parameter types in the plugin. description one liner that will be used for 
synopsis. For example:  
`import com.performizeit.mjstack.api.Plugin;

@Plugin(name="group", paramTypes={String.class},description="Group by an attribute")`
There are several plugin types mappers filters terminals etc... for each type you will  need to implement a different interface.

Mapper
------
Implement JStackMapper interface which includes a single method 
`ThreadInfo map(ThreadInfo stck)`

Filter
------
Implement JStackFilter interface which includes a single method 
`boolean filter(ThreadIndo stck)`

Terminal
------
Implement JStackFilter interface which includes a single method 
`void addStackDump(JStackDump jsd)` 

Comparator
------
Implement JStackComparator interface which includes a single method 
`int compare(ThreadInfo o1, ThreadInfo o2)`



Installing a plugin
============
In order to install the plugin just drop your jar which contains plugin implementation  into the 'plugins' directory 
inside mjprof installation directory.
