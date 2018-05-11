:author: Vyas Ramasubramani
:email: vramasub@umich.edu
:institution: Department of Chemical Engineering, University of Michigan, Ann Arbor

:author: Carl S. Adorf
:email: csadorf@umich.edu
:institution: Department of Chemical Engineering, University of Michigan, Ann Arbor

:author: Paul M. Dodd
:email: pdodd@umich.edu
:institution: Department of Chemical Engineering, University of Michigan, Ann Arbor

:author: Bradley D. Dice
:email: bdice@umich.edu
:institution: Department of Physics, University of Michigan, Ann Arbor

:author: Sharon Glotzer
:email: sglotzer@umich.edu
:institution: Department of Chemical Engineering, University of Michigan, Ann Arbor
:institution: Department of Materials Science and Engineering, University of Michigan, Ann Arbor
:institution: Department of Physics, University of Michigan, Ann Arbor
:institution: Biointerfaces Institute, University of Michigan, Ann Arbor

:bibliography: paper

-----------------------------------------------------------
signac: A Python framework for data and workflow management
-----------------------------------------------------------

.. class:: abstract

Computational research requires versatile data and workflow management tools that can easily adapt to the highly dynamic requirements of scientific investigations. 
Many existing tools require strict adherence to a particular usage pattern, so researchers often use less robust ad hoc solutions that they find easier to adopt. 
The resulting data fragmentation and methodological incompatibilities significantly impede research. 
Our talk showcases signac, an open-source Python framework that offers highly modular and scalable solutions for this problem. 
The framework's powerful workflow management tools enable users to construct and automate workflows that transitions seamlessly from laptops to HPC clusters. 
Crucially, the underlying data model is completely independent of the workflow. 
The flexible, serverless, and schema-free signac database can be introduced into other workflows with essentially no overhead and no recourse to the signac workflow model. 
Additionally, the data model's simplicity makes it easy to parse the underlying data without using signac at all. 
This modularity and simplicity eliminates significant barriers for consistent data management across projects, facilitating improved provenance management and data sharing with minimal overhead.

.. class:: keywords

	data management, database, data sharing, provenance, computational workflow 

Introduction
------------

Streamlining data generation and analysis is a critical challenge for science in the age of big data and high performance computing (HPC). Modern computational resources can generate and consume enormous quantities of data, but process automation and data management tools have lagged behind. The highly file-based workflows characteristic of computational science are not amenable to traditional relational databases, and HPC applications require that data is available on-demand, enforcing strict performance requirements for any data storage mechanism. Building processes acting on this data requires transparent interaction with HPC clusters without sacrificing testability on personal computers, and these processes must be sufficiently malleable to adapt to changes in scientific inquiries.

Consider studying the motion of an object through a fluid medium. If we initially consider the motion only as a function of one parameter, an ad hoc solution for data storage would be to store the trajectories in files named for the values of this parameter. If we then introduce some post-processing step, we could run it on each of these files. However, a problem arises if we realize that some additional parameter is also relevant. A simple solution might be to just rename the files to account for this parameter as well, but this approach would quickly become intractable if the parameter space increased further, and a more flexible traditional solution involving the use of, e.g., a relational database might introduce undesirable setup costs and performance bottlenecks for file-based workflows on HPC. Even if we do employ such a solution, we also have to account for our workflow process: we need a way to run analysis and post-processing on just the new data points without performing unnecessary work on the old ones.

This paper showcases the signac framework, a data and workflow management tool that aims to address these issues in a simple, powerful, and flexible manner. By JSON-encoded metadata directly and the associated data together on the file system, signac provides database functionality such as searching and grouping data without the overhead of maintaining a server or interfacing with external systems, and it takes advantage of the high performance file systems common to HPC. With signac, data space modifications like the one above are trivially achievable with just a few lines of Python code. Additionally, signac's workflow component makes it just as easy to modify the process of data generation, since we simply define the post-processing as a Python function. The workflow component of the framework, signac-flow, will immediately enable the use of this calculation on the existing data space through a single command, and it tracks which tasks are complete to avoid redoing completed tasks. Once the process has completed, the resulting data can be accessed without reference to the workflow, ensuring that it is immediately available to anyone irrespective of the tools they are using.


**Overview figure**

Overview and Examples
---------------------
**Maybe have a global schematic of both signac and flow together here. It can show like workspace, job, project, etc as one summary, then include Flowprojects as part of the other summary to show the components of flow and the link**
To demonstrate how signac works, we take a simple, concrete example of the scenario described above. Consider an experiment in which we want to find the optimal launch angle to maximize the distance traveled by a projectile through air. The first step is initializing the data space, as shown in Figure 2. Figure 2 demonstrates the creation of the core entities in signac's data model: (1) the project, the overarching directory in which everything happens; (2) the workspace, where data is stored for individual data points; (3) the job, which represents a single data point; and (4) the state point, the unique key identifying the job. Although in practice we can see that everything is stored as a file, these objects provide layers of abstraction that make them far more useful than simple file system storage, as we will see.

**Insert figure**

One could easily imagine interfacing existing scripts with this data model. The only requirement is some concept of a unique key for all data so that it can be inserted into the database. The unique key is what enables the creation of the 32 character hash, or job id, used to identify the job and its workspace folder (shown in Fig. 2). The uniqueness of this hash is what enables all of signac's indexing and related functionality. 

Ultimately, however, it is important to define the processes that generate and operate on this data cleanly and concisely. The signac-flow component of the framework provides the tools to accomplish this. In the below code block, we demonstrate how we could automate the generation of this data using signac-flow.

**Insert figure**

Note that we are now using the job document, a lightweight JSON storage mechanism, but we can also directly store files into the job's workspace and operate on them later. This unrestrictive model enables easy adaptation and modification as needed. For example, if we instead want to consider how changing initial velocity affects the distance traveled for a particular angle, we can add the velocity to the existing job state points by taking advantage of the fact that the project object is an iterable:

.. code-block:: python
    import numpy as np
    for job in project: # equivalent to `for job in project.find_jobs():`
        job.sp.v = 1

Note that an alternative to operating on all jobs is by selecting some subset. One way to accomplish this would be to apply a filter within the loop using conditionals based on the job state point. A more elegant solution, however, is to take advantage of signac's query API, which allows the user to find only the jobs of interest using a dictionary as a filter. The job finding functionality of signac is the entry point for its database functionality, enabling advanced indexing, selection, and grouping operations.

Having made the above change to our data space, we could now  easily add new data points to test:

.. code-block:: python
    import numpy as np
    for v in [1, 2, 3]:
        for theta in np.round(np.linspace(0, 3.14/2, 5), 2):
            project.open_job({"v": v, "theta": theta}).init()

Note that jobs that already exist in the data space will not be overwritten, so there is no harm in performing a loop like this multiple times.

Note that so far, we have shown examples of working with signac both in scripts and inside Jupyter notebooks. In fact, all of signac's core functionality is also available on the command line, making it easy to interface signac with almost any pre-existing workflow.  While these features are critical for interfacing with non-Python code bases, they are also very useful for more ad hoc analyses of signac data spaces. For example, the search functionality is available directly on the command line, which can be very useful for quick inspection of data:

.. code-block:: bash
    $ # signac's command line interface will interpret simple queries
    $ signac find theta 0.39
    Interpreted filter arguments as '{"theta": 0.39}'.
    d3012d490304c3c1171a273a50b653ad
    1524633c646adce7579abdd9c0154d0f
    22fa30ddf3cc90b1b79d19fa7385bc95

    $ # More complex queries can be constructed with JSON
    $ signac find '{"v": {"$lt": 2}}'
    d61ac71a00bf73a38434c884c0aa82c9
    00e5f0c36294f0eee4a30cabb7c6046c
    585599fe9149eed3e2dced76ef246903
    22fa30ddf3cc90b1b79d19fa7385bc95
    9fa1900a378aa05b9fd3d89f11ef0e5b


Additionally, at any point we can get an overview of what the data space schema looks like:

.. code-block:: bash
    $ signac schema
    {
     'theta': 'int([3], 1), float([0.0, 0.39, 0.78, 1.18, 1.57], 5)',
     'v': 'int([1, 2, 3], 3)',
    }

Now that we have this data space, one could imagine defining more complex workflows than the simple single-operation one shown in Fig. 3. In fact, signac-flow enables arbitrarily complex workflows that use simple pre- and post-conditions on individual operations to construct a directed acyclic graph:

.. code-block:: python
    # The project.py shown in the notebook
    ...
    @FlowProject.operation
    @FlowProject.post(lambda job: 'tmax' in job.document())
    def calculate(job):
        ...

Once the operations are defined along with the associated conditions, the interface demonstrated above will automatically run through the workflow in the appropriate sequence, ensuring that only incomplete tasks are run, i.e., once `tmax` has been calculated for a particular job, the `calculate` operation will not run again for that job. While this automatic parallelization over the workflow is the default behavior, signac-flow also enables much more fine-grained control:

.. code-block:: bash
    $ # Automatically run all outstanding operations for all jobs
    $ python project.py run
    $ # ignore workflow status and run a specific job operation
    $ python project.py exec -o ${OP} -j ${JOB_ID}
    $ python project.py run --

    A critical feature of the signac framework is scalability to HPC. The file-based data model is designed to leverage the high performance file systems common on such systems, and workflows designed locally are immediately executable on HPC clusters. In particular, any operation that can be successfully executed in the manner shown in Fig. 3 can also be immediately submitted to cluster schedulers. The signac-flow package achieves this by creating cluster job scripts that perform the above operations:

    .. code-block:: bash
    $ python project.py submit -n 1 -w 12 --pretend # Print the script for one 12-hour job
    Submitting operation 'calculate' for job 'd61ac71a00bf73a38434c884c0aa82c9'...
    #PBS -N d61ac71a00bf73a38434c884c0aa82c9-calculate
    #PBS -l walltime=12:00:00
    #PBS -l nodes=1
    #PBS -V
    set -e
    set -u

    cd /nfs/glotzer/projects/signac/scipy/sample_project

    # Operation 'calculate' for job 'd61ac71a00bf73a38434c884c0aa82c9':
    python project.py exec calculate d61ac71a00bf73a38434c884c0aa82c9

The workflow tracking functionality of signac-flow extends to compute clusters. In general, users can always check the status of particular jobs to see how far they have progressed in the workflow

.. code-block:: bash
    $ python project.py submit -n 3 -w 12 --hold # Submit 3 random jobs for 12 hours
    $ python project.py status -d
    Status project 'Projectiles':
    Total # of jobs: 16

    label    progress
    -------  ----------
    [no labels]

    Detailed view:
    job_id                               			S    	next_op    	labels
    --------------------------------  			---  	---------  	--------
    00e5f0c36294f0eee4a30cabb7c6046c  	U !  	calculate
    585599fe9149eed3e2dced76ef246903  	A  	calculate
    2faf0f76bde3af984a91b5e42e0d6a0b  	U !  	calculate
    75e65263ecf783a50858e3c73365de16  	U !  	calculate
    13d54ee5821a739d50fc824214ae9a60  	U !  	calculate
    09310923e2ddaf5d55201ccfa25b594a  	U !  	calculate
    029bd71f9412e12a881df1aaf9a3a093 	U !  	calculate
    03d50a048c0423bda80c9a56e939f05b  	U !  	calculate
    2fc4156e493deb1ab16607a3c2b99630  	U !  	calculate
    d61ac71a00bf73a38434c884c0aa82c9  	A  	calculate
    22fa30ddf3cc90b1b79d19fa7385bc95  	U !  	calculate
    41dea88eaee4159c3a5e7dce6d8e51f7	A 	calculate
    3201fd381819dde4329d1754233f7b76  	U !  	calculate
    1524633c646adce7579abdd9c0154d0f	U !  	calculate
    d3012d490304c3c1171a273a50b653ad  	U !  	calculate
    9fa1900a378aa05b9fd3d89f11ef0e5b  	U !  	calculate

    Abbreviations used:
    !: requires_attention
    A: active
    S: status
    U: unknown

All jobs in the projects are currently eligible for the calculate operation (next_op column), but 3 of them are currently active on the cluster as shown in the second column. Once the operation has completed, the next_op column will become empty since the post-condition that we set would indicate that the operation has completed. Note that the labels column is empty because we have not created any labels. The label feature of signac-flow provides a way to enrich the status output by defining custom functions that indicate indicate additional information about jobs. Additionally, these label functions can be used as conditions for operation execution.

The quick overview of this section highlights the core features of the signac framework. Although demonstrated here for a very simple example, the data model scales easily to thousands of data points and far more complex and nonlinear workflows. **Demonstrations can be seen on the documentation on ReadTheDocs, the signac website, or the paper in JCMS.**


Design and Implementation
-------------------------

Having provided an overview of signac's functionality, we now provide a few more specifics on its implementation. The framework prioritizes modularity and interoperability over monolithic functionality, so it is implemented in pure Python with no hard dependencies to ensure that it can be easily used with other programs. The software runs equally well on Python 2.7 and 3.4+, and the full-featured command line interface enables its use even with non-Python tools. To ensure that the data representation is completely independent of the workflow, the data component is developed as a standalone database solution. This database is the primary dependency for signac-flow. 

The signac package is designed to be as lightweight and flexible as possible, simultaneously offering the benefits of filesystem usage and more traditional DBMS. From the Python implementation standpoint, the central component to the signac framework is the Project class, which provides the interface to signac's data model and features. The signac project encapsulates much of the functionality for searching, selecting, and grouping individual data points from the data space. Central to this process is signac's efficient on-the-fly indexing. This process, which leverages the fact that the state point keys map uniquely to a workspace directory, uses filesystem crawlers to efficiently traverse the data space. The indexing process is completely transparent to the user, but can be exposed on demand. Accessing individual data points from this index leads to the instantiation of Jobs, which are Python object handles that represent individual data points. Since these data points effectively correspond to filesystem locations, they can be mapped directly and operated on in this fashion.

The central object in the signac-flow package is the Python FlowProject class, which encapsulates a set of operations acting on a signac data space. There is a tight relationship between the FlowProject and the underlying data space, because operations are assumed to act on a per-job basis. Using the sequence of conditions associated with each operation, FlowProjects also track workflow progress on per-job basis, and therefore they always know which operations to run next for a given job. Different HPC environments and cluster schedulers are represented by separate Python classes that provide the means for querying schedulers for cluster job statuses, writing out the job scripts, and constructing the submission commands. This means that workflows designed on one cluster can be easily ported to another.

The extensibility of the signac framework makes it easy to build other tools on top of signac. One such tool is signac-dashboard, a web interface to signac data spaces that provides an easy way to visualize existing data. This tool has been released open source and is currently under active development.


Comparisons
-----------

Numerous tools have been developed for Python workflow management in recent years. Of these, some of the best known are Fireworks, AiiDA, Sacred, and Sumatra. Fireworks and AiiDA are full-featured workflow managers that, like signac-flow, interface with high performance compute clusters to execute complex, potentially nonlinear workflows. These tools in fact currently offer more powerful features than signac-flow for monitoring the progress of jobs, features that are supported by the use of databases on the back end. However, with these software the data representation is far more rigid than with signac. In particular, the data cannot be accessed outside the context of the workflow definition, a significant limitation for data analysis and a barrier to integration with pre-existing workflows. Additionally, the need to maintain a server for workflow management can be cumbersome and introduce additional unnecessary complexities.

Superficially, the signac framework appears quite similar to Sacred. Both use decorators to convert functions into command line executable operations, and configurations can be injected into these functions (in signac's case, using the job object). However, the similarities largely stop there; both Sacred and Sumatra are much more focused on provenance management than defining complex workflows.The tracking performed by Sacred is almost entirely focused on when an operation is executed, the configuration used, and whether any data was saved. The same is true of Sumatra, which primarily differs from Sacred in that its provenance tracking functionality is entirely available on the command line. Since Sacred is dependent on using Python decorators to define the command-line interface to Python scripts, it is incompatible with signac. However, Sumatra can be used with signac and integrates naturally.

We have found fewer similar alternatives to the signac data model. The closest comparison that we have found is datreant.core, which provides the means for interacting with files on the file system along with some features for finding, filtering, and grouping. The primary distinctions between datreant and signac arise from two things: signac's requirement of a unique key, and the fact that signac provides tightly integrated workflow management using signac-flow. The datreant data model is even simpler than signac's, which provides additional flexibility at the cost of signac's database functionality. This difference is indicative of datreant's focus on more general file management problems than the issues signac is designed to solve. Similar to signac, datreant's data model easily supports the creation of workflows that use it, and indeed one such example exists in MDSynthesis. However, this tool is highly domain-specific, unlike signac-flow, and it cannot be used for other types of computational investigations. In the field of molecular simulation, the combination of MDSynthesis and datreant is the closest analog to the signac framework, but that software does not generalize to other use-cases.



Conclusions
-----------

The signac framework provides all the tools required for thorough data, workflow, and provenance management in scientific computing investigations. Motivated by the need for managing the dynamic, heterogeneous data spaces characteristic of computational science investigations, the tools are tailored for the use-cases most commonly faced in this field. By minimizing external requirements, employing open data formats like JSON, and striving for simplicity, we have made the framework as easy to use as possible. The framework frees computational scientists from repeatedly solving common data and workflow problems throughout their research, and at a higher level, it reduced the burden of sharing data and provenance tracking, both of which are critical to accelerating the production of reproducible scientific results.
