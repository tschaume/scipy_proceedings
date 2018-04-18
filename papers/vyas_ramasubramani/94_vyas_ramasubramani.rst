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
:email: vramasub@umich.edu
:institution: Department of Physics, University of Michigan, Ann Arbor

:author: Sharon Glotzer
:email: sglotzer@umich.edu
:institution: Department of Chemical Engineering, University of Michigan, Ann Arbor
:institution: Department of Materials Science and Engineering, University of Michigan, Ann Arbor
:institution: Department of Physics, University of Michigan, Ann Arbor
:institution: Biointerfaces Institute, University of Michigan, Ann Arbor

:bibliography: paper

.. :video: http://www.youtube.com/watch?v=dhRUe-gz690

-----------------------------------------------------------
signac: A Python framework for data and workflow management
-----------------------------------------------------------

.. class:: abstract

	**Based on the comments on our proposal I've written an abstract to really highlight why we are important/unique. It's probably a little too long, but I first want to make sure key ideas come across**

    Organizing and conducting computational research requires versatile, minimally restrictive tools that can easily adapt to the highly dynamic data schemas and workflows characteristic of scientific investigations.
    The workflow requirements of many existing tools are sufficiently onerous to discourage their incorporation into existing work, with researchers preferring their own *ad hoc* data and provenance management methods that are less robust but require no additional effort to adopt.
    The resulting data fragmentation and methodological incompatibilities significantly impede computational research.
	Our talk showcases the signac framework, an open-source Python package that offers highly modular and scalable solutions for this problem.
    Its highly flexible, server-free data model allows easy adaptation into pre-existing file-based workflows without sacrificing critical database functionality such as filtering, searching, and grouping data.
    Furthermore, by decoupling data and workflow management, signac nearly eliminates the barrier for initial adoption while also providing powerful workflow management tools on top of its data model as needed.
    The framework is designed with high performance computing in mind, meaning that both its data and workflow management components scale from laptop or desktop usage to large compute clusters.


    **I think that the critical benefit of signac that we need to focus on is how lightweight and easy it is to adopt. Users can start using it with basically 0 startup cost, and it doesn't penalize users for having poorly defined schemas. This property is what makes it so useful to scientists who don't wnat to have to come up with a complex schema to begin with. That being the case, I actually think focusing on flow too much takes away our ability to claim this. While it is powerful and flexible, there's enough boilerplate that it's not yet an easy sell that it is *easy* to adopt flow on top of signac.
    I think that if we do a sufficiently good job refactoring flow, then this could change and we have an opportunity to make gains there. In any case, though, I think that for this talk we really should start by focusing on signac and its simplicity. We can get to flow after as being optional, but because flow still requires insubstantial boilerplate I think in that sense while it's still good it weakens that part of our argument. Same goes for dashboard. That said, we should definitely still present them, we just have to frame them differently. Also this makes reworking our flow front-end pretty critical IMO (as discussed with Simon)**

.. class:: keywords

	data management, database, data sharing, provenance, computational workflow 

Introduction
------------

In the age of big data and high performance computing, streamlining the processes of data generation and analysis has become a primary challenge.
Increases in computational resources have made it easier to generate and consume large quantities of data, but computational science has not yet maximized the possible gains from this due to limitations in the downstream processes for organizing and analyzing the data.
Many applications in computational science depend on highly file-based workflows not amenable to traditional relational database for storing data, and HPC applications in particular require that data be available on-demand, enforcing strict performance requirements for any data storage mechanism.
The ad hoc solutions typically employed in standard scientific workflows, such as using file-naming conventions (**Need to show examples**) for data management, are highly inflexible and lead to various downstream inefficiencies.
This paper showcases the signac framework, a data and workflow management tool that aims to address these issues in a simple, powerful, and flexible manner.

To concretely demonstrate signac's capabilities, we turn to an example.

Consider the simple experiment of measuring the distance traveled by a projectile.
Initially, we consider the simple experiment of a person throwing a pumpkin.
Hypothesizing that the distance traveled is a function of the launch angle, you set up a simple computational experiment that applies basic kinematic to predict the distance traveled and stores the data in files named by the angle it contains data for (a relatively common practice in fields where the standard software packages generate all outputs as files).
After the initial experiments are completed, however, you realize that a much larger initial velocity (and therefore a greater distance) could be attained by using a cannon instead of your arm!
The simplest method for addressing this change, simply renaming all files to account for the new parameter, would quickly become infeasible if the parameter space increased further, and a more flexible traditional solution involving the use of e.g. a relational database might introduce unsustainable performance bottlenecks for file-based workflows.

The signac framework is designed to handle this problem in a flexible and natural way for such workflows.
By storing data with its associated metadata in an organized manner *directly on the filesystem*, signac provides database functionality such as searching and grouping data without the overhead of maintaining a server or interfacing with external systems.
With signac, data space modifications like the one above are trivially achievable with just a few lines of Python code.
Additionally, signac's workflow component makes it just as easy to modify the process of data generation.
For example, accounting for air resistance in our calculation and comparing that result to our more naive approach is as simple as defining the new distance calculation as a Python function; signac's workflow component will immediately enable the use of this calculation on the existing data space through a single command-line command. 


Usage and Overview
------------------
**Maybe have a global schematic of both signac and flow together here. It can show like workspace, job, project, etc as one summary, then include Flowprojects as part of the other summary to show the components of flow and the link**

*Given the audience, it may be fine to just include the code as part of the introduction*

To make this more concrete, we demonstrate how the above example might work in practice.

*Basic example of ad hoc data space creation using just signac to address the previous problem*

If we run this script, we now see the following folder structure has been created.

*Show folder structure*

In the context of signac, this parent folder is the *project*, denoted by the presence of the signac.rc configuration file (which at the moment just contains the name of the project).
The *workspace* is the other core component of signac; it is the folder within which all data is stored.
We can easily parse this data structure with signac.
For example, to find the name of the project, we can enter `signac project`; to find the specific directory, or *job*, which has a particular :math:`v_0` value, we can execute `signac find v0 1`.
A major benefit of signac is that the data can be easily parsed even **without using signac**.
The metadata is encoded in JSON (show this in the figure detailed below), making it easy to crawl the filesystem and immediately get this data out.

*Make figure showing evolution of folder structure and the commands that can be executed. Basically, first show that there is nothing, then signac.rc, then one job, then many jobs. Below that, show contents of one job.stateopin*

The setup cost here was essentially zero.
Once the software was installed, we could immediately store the data and manage it without spinning up a server or introducing more complex dependencies.
Additionally, the service is completely divorced from any particular workflow.
Simply call the relevant function and data will be stored in the right place; how you generate that data is up to you.

**Should talk some about indexing**


Defining workflows
++++++++++++++++++

Defining the workflow we have detailed above is equally trivial.
Assuming that the function to perform the calculation already exists, we simply have to modify it to take the job as an argument and extract the required code from it.
Alternatively, if the function is in the form of a python script that takes the desired arguments on the command line, we can simply write another function to call it as follows: **example to follow pending changes to flow**.
If code already exists to perform specific tasks, the goal of signac-flow is to make it trivial to immediately string these into a logical sequence of operations that can be easily automated.
Once defined (as shown above), running these operations is also extremely simple from the command line: `python project.py run ${OPERATION}` will execute any operation.
The project interface is more powerful than that, though; it also enables the user to get the status of all jobs in a project, to determine the next jobs in the sequence, or submit to clusters.
Cluster submission is one of the most important roles of the FlowProject (**needs earlier intro**).
Designed to work with any cluster environment (and prepackaged to work with Slurm and Torque PBS schedulers out of the box), given a FlowProject defined as above submitting a job to a cluster is as simple as `python project.py submit`.
In addition to the time-saving but not groundbreaking result that you never have to write a job script again, this submission mode is enormously powerful, enabling essentially complete control over the submission process.
The basic command described above will immediately execute the next operation defined for each job; however, the user can instead specify particular jobs or operations to run.
In addition, all standard submission controls are available, such as specifying the job walltime, the number of processors, the type of processor (cpu vs gpu), etc.


Design and Comparisons
----------------------

The software is designed to be as lightweight and flexible as possible, simultaneously offering the benefits of filesystem usage and more traditional DBMS.
**Contrast with the use of DBMS, talk about indexing and how it is implementing via JSON metadata and crawlers. Also talk about the efficiency in our use-cases i.e. lots of parallelism and serialization.**
Using file-system crawlers and leveraging the JSON metadata stored in each job, we can efficiently traverse the data space and construct indices.
While users can interact with these directly, they are automatically generated on-the-fly for all search operations, ensuring that their usage is transparent unless a more explicit representation is required.
In this fashion, the underlying data is highly flexible in the signac representation, since the only requirement is that there is some sort of primary key that can be expressed as a JSON encodeable object.

The closest comparison to the signac data model we have found is datreant, which is even less restrictive in not requiring such a key structure.
**Talk a little about datreant's drawbacks**
The primary distinction between datreant and signac, however, is the existence of a workflow manager on top of these data structures.

The signac framework's explicit focus on flexible data representation distinguishes signac from the majority of similar software in the field, which *couple* the workflow and data aspects.
Important examples of this are the AiiDA project and the Fireworks tools, both of which enable powerful workflow management at the cost of data representation flexibility.
Using either of these tools strongly couples you to a particular data representation, which can be not only a significant barrier to implementation, but in fact a hindrance when this data model is not the most natural way to structure your data.
In contrast, while signac-flow requires using signac as the back-end, the data schema and details of the database usage are left entirely to the user, allowing much greater flexibility.
In this respect, signac-flow is more comparable to something like Sacred, which enables the tracking of experiments through a workflow management like engine.
However, signac offers the additional benefit of enabling multistep workflows in a transparent manner, and additionally, the ability to scale these workflows to HPC environments (**Make sure to verify this and include comparison to sacred**).

The closest thing to replicating signac's capabilities would be an integrated setup utilizing datreant and Sacred.
Since those pieces of software individually address the data and workflow components, it is possible to integrate them in a manner that affords sufficient flexibility to the user.
**Maybe show an example?**
However, this requires much more work, and is not nearly as transparent as the usage of signac.

**We can also talk more about signac's modularity and extensibility here, e.g. with respect to dashboard's development**


Implementation Details
------------------------

The signac framework is implemented in entirely in pure Python with no additional hard dependencies.
The software runs equally well on 2.x and 3.x, and a command line interface is available and easy to use for core functionality to simplify integration with other tools.
The central component to the signac framework is the Project class, which provides the interface to signac's data model and features.
The signac project encapsulates much of the functionality for indexing, searching, selecting, and grouping individual data points from the data space.
Each of these processes generates an index, and accessing individual data points from this index leads to the instantation of Jobs, which are Python object handles that represent individual data points.
Since these data points effectively correspond to filesystem locations, they can be mapped directly and operated on in this fashion.

**What else is important to include here? We can incorporate details as desired here**


Conclusion
----------


