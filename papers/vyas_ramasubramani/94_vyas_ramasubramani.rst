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

:bibliography: mybib

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

    **Main differences with sacred I see is that at the core we are more focused on the data itself. Sacred is more focused on the workflow. We should emphasize this distinction somehow**
.. class:: keywords

	data management, database, data sharing, provenance, computational workflow 

Introduction
------------

**As I write this outline, I'm finding that I'm naturally creating a similar set of sections to the original paper. Someone else feel free to tell me if you think of a better layout.**

I think our introduction needs to be a little bit more balanced with respect to flow than the JCMS paper.
We can use a similar example potentially, but I think showing the data vs. workflow components will be critical to convincing our readers that we have a useful and unique product.
In that paper the bar was higher in terms of showing that there is actually a problem that had to be solved, in the sense that we want materials scientists to recognize that there are better ways to store data.
Here, the bar is higher in terms of showing that what we're implementing is actually a substantial contribution to the existing software ecosystem.

Design
------

Here we can go into detail about how signac works, and the various parts.
I think flow should get a more first-class treatment here than it did in the JCMS paper so that we can clearly show the delineation between data and workflow.
Also dashboard can be shown as an example of a plug-in that takes advantage of signac's modularity (if it's at that point).
I also think this is a good place to draw comparisons to existing software; we can repurpose the comparable solutions from the paper.
Rather than keeping that separate, however, I think we should inline it with our design decisions to help justify why we did what we did.

Implementation
--------------

Relative to the paper, I think we want to show more examples here.
How do you install signac?
What can you do?
I think this section can be more code-heavy now.


Bibliographies, citations and block quotes
------------------------------------------

If you want to include a ``.bib`` file, do so above by placing  :code:`:bibliography: yourFilenameWithoutExtension` as above (replacing ``mybib``) for a file named :code:`yourFilenameWithoutExtension.bib` after removing the ``.bib`` extension.

**Do not include any special characters that need to be escaped or any spaces in the bib-file's name**. Doing so makes bibTeX cranky, & the rst to LaTeX+bibTeX transform won't work.

To reference citations contained in that bibliography use the :code:`:cite:`citation-key`` role, as in :cite:`hume48` (which literally is :code:`:cite:`hume48`` in accordance with the ``hume48`` cite-key in the associated ``mybib.bib`` file).

However, if you use a bibtex file, this will overwrite any manually written references.

So what would previously have registered as a in text reference ``[Atr03]_`` for

::

     [Atr03] P. Atreides. *How to catch a sandworm*,
           Transactions on Terraforming, 21(3):261-300, August 2003.

what you actually see will be an empty reference rendered as **[?]**.

E.g., [Atr03]_.


If you wish to have a block quote, you can just indent the text, as in

    When it is asked, What is the nature of all our reasonings concerning matter of fact? the proper answer seems to be, that they are founded on the relation of cause and effect. When again it is asked, What is the foundation of all our reasonings and conclusions concerning that relation? it may be replied in one word, experience. But if we still carry on our sifting humor, and ask, What is the foundation of all conclusions from experience? this implies a new question, which may be of more difficult solution and explication. :cite:`hume48`

Dois in bibliographies
++++++++++++++++++++++

In order to include a doi in your bibliography, add the doi to your bibliography
entry as a string. For example:

.. code-block:: bibtex

   @Book{hume48,
     author =  "David Hume",
     year =    "1748",
     title =   "An enquiry concerning human understanding",
     address =     "Indianapolis, IN",
     publisher =   "Hackett",
     doi = "10.1017/CBO9780511808432",
   }


If there are errors when adding it due to non-alphanumeric characters, see if
wrapping the doi in ``\detokenize`` works to solve the issue.

.. code-block:: bibtex

   @Book{hume48,
     author =  "David Hume",
     year =    "1748",
     title =   "An enquiry concerning human understanding",
     address =     "Indianapolis, IN",
     publisher =   "Hackett",
     doi = \detokenize{10.1017/CBO9780511808432},
   }

Source code examples
--------------------

Of course, no paper would be complete without some source code.  Without
highlighting, it would look like this::

   def sum(a, b):
       """Sum two numbers."""

       return a + b

With code-highlighting:

.. code-block:: python

   def sum(a, b):
       """Sum two numbers."""

       return a + b

Maybe also in another language, and with line numbers:

.. code-block:: c
   :linenos:

   int main() {
       for (int i = 0; i < 10; i++) {
           /* do something */
       }
       return 0;
   }

Or a snippet from the above code, starting at the correct line number:

.. code-block:: c
   :linenos:
   :linenostart: 2

   for (int i = 0; i < 10; i++) {
       /* do something */
   }

Important Part
--------------

It is well known [Atr03]_ that Spice grows on the planet Dune.  Test
some maths, for example :math:`e^{\pi i} + 3 \delta`.  Or maybe an
equation on a separate line:

.. math::

   g(x) = \int_0^\infty f(x) dx

or on multiple, aligned lines:

.. math::
   :type: eqnarray

   g(x) &=& \int_0^\infty f(x) dx \\
        &=& \ldots

The area of a circle and volume of a sphere are given as

.. math::
   :label: circarea

   A(r) = \pi r^2.

.. math::
   :label: spherevol

   V(r) = \frac{4}{3} \pi r^3

We can then refer back to Equation (:ref:`circarea`) or
(:ref:`spherevol`) later.

Mauris purus enim, volutpat non dapibus et, gravida sit amet sapien. In at
consectetur lacus. Praesent orci nulla, blandit eu egestas nec, facilisis vel
lacus. Fusce non ante vitae justo faucibus facilisis. Nam venenatis lacinia
turpis. Donec eu ultrices mauris. Ut pulvinar viverra rhoncus. Vivamus
adipiscing faucibus ligula, in porta orci vehicula in. Suspendisse quis augue
arcu, sit amet accumsan diam. Vestibulum lacinia luctus dui. Aliquam odio arcu,
faucibus non laoreet ac, condimentum eu quam. Quisque et nunc non diam
consequat iaculis ut quis leo. Integer suscipit accumsan ligula. Sed nec eros a
orci aliquam dictum sed ac felis. Suspendisse sit amet dui ut ligula iaculis
sollicitudin vel id velit. Pellentesque hendrerit sapien ac ante facilisis
lacinia. Nunc sit amet sem sem. In tellus metus, elementum vitae tincidunt ac,
volutpat sit amet mauris. Maecenas [#]_ diam turpis, placerat [#]_ at adipiscing ac,
pulvinar id metus.

.. [#] On the one hand, a footnote.
.. [#] On the other hand, another footnote.

.. figure:: figure1.png

   This is the caption. :label:`egfig`

.. figure:: figure1.png
   :align: center
   :figclass: w

   This is a wide figure, specified by adding "w" to the figclass.  It is also
   center aligned, by setting the align keyword (can be left, right or center).

.. figure:: figure1.png
   :scale: 20%
   :figclass: bht

   This is the caption on a smaller figure that will be placed by default at the
   bottom of the page, and failing that it will be placed inline or at the top.
   Note that for now, scale is relative to a completely arbitrary original
   reference size which might be the original size of your image - you probably
   have to play with it. :label:`egfig2`

As you can see in Figures :ref:`egfig` and :ref:`egfig2`, this is how you reference auto-numbered
figures.

.. table:: This is the caption for the materials table. :label:`mtable`

   +------------+----------------+
   | Material   | Units          |
   +============+================+
   | Stone      | 3              |
   +------------+----------------+
   | Water      | 12             |
   +------------+----------------+
   | Cement     | :math:`\alpha` |
   +------------+----------------+


We show the different quantities of materials required in Table
:ref:`mtable`.


.. The statement below shows how to adjust the width of a table.

.. raw:: latex

   \setlength{\tablewidth}{0.8\linewidth}


.. table:: This is the caption for the wide table.
   :class: w

   +--------+----+------+------+------+------+--------+
   | This   | is |  a   | very | very | wide | table  |
   +--------+----+------+------+------+------+--------+

Unfortunately, restructuredtext can be picky about tables, so if it simply
won't work try raw LaTeX:


.. raw:: latex

   \begin{table*}

     \begin{longtable*}{|l|r|r|r|}
     \hline
     \multirow{2}{*}{Projection} & \multicolumn{3}{c|}{Area in square miles}\tabularnewline
     \cline{2-4}
      & Large Horizontal Area & Large Vertical Area & Smaller Square Area\tabularnewline
     \hline
     Albers Equal Area  & 7,498.7 & 10,847.3 & 35.8\tabularnewline
     \hline
     Web Mercator & 13,410.0 & 18,271.4 & 63.0\tabularnewline
     \hline
     Difference & 5,911.3 & 7,424.1 & 27.2\tabularnewline
     \hline
     Percent Difference & 44\% & 41\% & 43\%\tabularnewline
     \hline
     \end{longtable*}

     \caption{Area Comparisons \DUrole{label}{quanitities-table}}

   \end{table*}

Perhaps we want to end off with a quote by Lao Tse [#]_:

  *Muddy water, let stand, becomes clear.*

.. [#] :math:`\mathrm{e^{-i\pi}}`

.. Customised LaTeX packages
.. -------------------------

.. Please avoid using this feature, unless agreed upon with the
.. proceedings editors.

.. ::

..   .. latex::
..      :usepackage: somepackage

..      Some custom LaTeX source here.

References
----------
.. [Atr03] P. Atreides. *How to catch a sandworm*,
           Transactions on Terraforming, 21(3):261-300, August 2003.


