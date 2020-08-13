---
title: "Guide to Running NAMD"
---
## What is NAMD
NAMD [^1] is a parallel molecular dynamics code designed for high-performance simulation of large biomolecular systems. Based on Charm++ parallel objects [^2], NAMD scales [^3] to hundreds of cores for typical simulations and beyond 500,000 cores [^4] for the largest simulations. NAMD uses the popular molecular graphics program VMD [^5] for simulation setup and trajectory analysis, but is also file-compatible with AMBER, CHARMM, and X-PLOR. NAMD is distributed free of charge [^6] with source code. You can build [^7] NAMD yourself or download binaries [^8] for a wide variety of platforms. NAMD tutorials [^9] show you how to use NAMD and VMD for biomolecular modelling.

## Current NAMD Functionality

### Software Setup
* Free to download and use. (Redistribution prohibited.)
* Precompiled binaries provided for Linux, Mac, and Windows.
* Installed at major NSF supercomputer sites.
* Portable to virtually any platform with ethernet or MPI.
* C++ source code and CVS access for modification.

### Molecule Building
* VMD used to prepare molecular structure for simulation. 
* Also reads X-PLOR, CHARMM, AMBER, and GROMACS input files. 
* Psfgen tool generates structure and coordinate files for CHARMM force field. 
* Efficient conjugate gradient minimization. 
* Fixed atoms and harmonic restraints. 
* Thermal equilibration via periodic rescaling, reinitialization, or Langevin dynamics.

### Basic Simulation
* Constant temperature via rescaling, coupling, or Langevin dynamics.
* Constant pressure via Berendsen or Langevin Nose-Hoover methods.
* Particle mesh Ewald full electrostatics for periodic systems.
* Symplectic multiple timestep integration.
* Rigid waters and bonds to hydrogen atoms.

### Advanced Simulation
* Chemical and conformational free energy calculations.
* Enhanced sampling via replica exchange.
* Tcl based scripting and steering forces.
* Analysis implemented as Tcl scripts in VMD [^5].
* Interactive visual steering interface to VMD [^5].

### Scalable Performance
* Based on the Charm++/Converse parallel runtime system.
* Spatial data decomposition for limited communication pattern.
* Message driven execution for latency tolerance on commodity networks.
* Measurement-based load balancing for scaling to thousands of processors.
* Largest simulation to date is over 100,000,000 atoms on 300,000 cores.

## Step 1 - Log in
The examples used in this guide are now configured to run on both the Cardiff Hawk and Swansea Sunbird clusters. In the former case, connect to hawklogin.cf.ac.uk with your Supercomputing user credentials using your preferred method (e.g. PuTTY from a Windows machine or ssh from any Linux terminal, thus

<pre style="color: silver; background: black;">$ ssh username@hawklogin.cf.ac.uk</pre>

The steps below involve typing commands (in black background) in the terminal window.

## Step 2 - Load a NAMD module 
In contrast to the initial release of most of the application guides in this series, we no longer assume that the module of choice would be selected from those legacy modules originally available on HPC Wales or Raven. The focus now is on the native, optimised implementations on the Skylake-based cluster itself, although the original legacy modules are still available through issuing the appropriate “module load hpcw” or “module load raven”. 
The first step would now be to determine and gain access to the specific native module required by issuing the command.

<pre style="color: silver; background: black;">$ module avail</pre>

this will provide visibility of and access to the entire set of native modules. 

A number of NAMD binary packages are available. Note that in common with most other software packages on the system, these are built with the Intel compiler.
<ul>
 <li>List pre-installed NAMD versions:
     <pre style="color: silver; background: black;">$ module avail namd</pre> </li>

 <li>Load the CPU version (namd/2.13-cpu): 
     <pre style="color: silver; background: black;">$ module avail namd/2.13-cpu</pre> </li>

 <li>Confirm the loaded modules. All dependencies are handled automatically via the module file:
     <pre style="color: silver; background: black;">$ module list</pre> </li>
</ul>

## Step 3 - Create a directory 
From your home directory, create a directory to hold the NAMD data: 

<pre style="color: silver; background: black;">
$ cd ~
mkdir namd
</pre>

## Step 4 - Obtain a test case
A number of NAMD benchmark test cases are provided with the installation. These include: 
* **ApoA1**: ApoA1 benchmark (92,224 atoms, periodic; 2fs timestep with rigid bonds, 12A cutoff with PME every 2 steps). Apolipoprotein A1 (ApoA1) is the major protein component of high-density lipoprotein (HDL) in the bloodstream and plays a specific role in lipid metabolism. The ApoA1 benchmark consists of 92,224 atoms and has been a standard NAMD cross-platform benchmark for years.
  ![apoa1 molecule]({{ page.root }}/fig/apoa1.png) 
* **F1atpase**: ATP synthase is an enzyme that synthesizes adenosine triphosphate (ATP), the common molecular energy unit in cells. The F1-ATPase benchmark (327,506 atoms) is a model of the F1 subunit of ATP s synthase (12A cutoff with PME every 2 steps).
  ![f1atpase molecule]({{ page.root }}/fig/f1atpase.png) 
* **STMV**: STMV benchmark (1,066,628 atoms, periodic; 2fs timestep with rigid bonds, 12A cutoff with PME every 2 steps). Satellite Tobacco Mosaic Virus (STMV ) is a small, icosahedral plant virus that worsens the symptoms of infection by Tobacco Mosaic Virus (TMV). SMTV is an excellent candidate for research in molecular dynamics because it is relatively small for a virus and is on the medium to high end of what is feasible to simulate using traditional molecular dynamics in a workstation or a small server.
  ![stmv molecule]({{ page.root }}/fig/stmv.png) 

An analysis of the performance of these benchmark cases can be found in [^3]. Choose the ApoA1 example at: `/apps/chemistry/namd/2.13-cpu/examples/apoa1`

Copy the CPU benchmark job script, namd.apoa1.slurm.q to your user space: 

<pre style="color: silver; background: black;">
cd ~/namd
cp -rip /apps/chemistry/namd/2.13-cpu/examples/namd.apoa1.slurm.q .
</pre>

A number of input files are required to run a namd job. These will be transferred to `/scratch` from `/apps/chemistry/namd/2.13-cpu/examples/apoa1` under control of the `namd.apoa1.slurm.q` script. 

Note also that the `/apps/chemistry/namd/2.13-cpu/examples/apoa1` directory also contains sample output files for both the CPU and GPU runs (output_scw_apoa1). 

## Step 5 - Submit a job
Now you are ready to run the examples with the supplied job script. First, however, you will need to edit the project specification line in the scripts 
~~~
#SBATCH -A scwxxxx                # project specification (change this)
~~~
{: .language-bash}

replacing the text string <b><i>scwxxxx</i></b> with your own project code.

* From your working directory, submit the job using:
  <pre style="color: silver; background: black;">$ sbatch namd.apoa1.slurm.q</pre> 
* Check the job queue by:
  <pre style="color: silver; background: black;">$ squeue</pre>

  This MPI run executes on 80 cores and is run twice to provide a timing sensitivity analysis. When the job commences, a file called `apoa1.out.p78.ppn39.<Job_ID>.0` is created in the NAMD directory that holds the job output - many temporary files are generated and routed to the user’s scratch directory, `/scratch/$USER/NAMD.apoa1.<Job_ID>`, created by the job (where <Job_ID> is the ID assigned by the queuing system).
* The job should take well under a minute minute using 80 cores if the case runs successfully. The two output files, `apoa1.out.p78.ppn39.<Job_ID>.0` and `apoa1.out.p78.ppn39.<Job_ID>.1` should contain all the output and point to successful completion of the job.
* Compare your job output with the output file, `apoa1.out.p78.ppn39.6925701.0`, in directory `/apps/chemistry/namd/2.13-cpu/examples/output_scw_apoa1`.

## Step 6 - More Test Cases
Three additional cases are provided, corresponding to the M06, M27 and M45 Major Urinary Protein (MUP) + IBM ligands, located at `/apps/chemistry/namd/2.13-cpu/examples/f1atpase` and `/apps/chemistry/namd/2.13-cpu/examples/stmv`.

Try running these using Step 4 and Step 5 described above for **apoa1**. Each MPI run of the f1atpase and stmv cases take ca. 1 minutes and 2 minutes respectively on 160 CPU cores.

## Scaling Performance
To provide an idea of the scaling of NAMD with increasing core count, we show in Figure 1 the relative performance and scaling of the **apoa1**, **f1atpase** and **stmv** test cases with increasing core (node) count. In all three cases the performance is normalised to that seen on a single node i.e. 40 cores. Note that the NAMD outputs used in generating the plots are available at 

* `/apps/chemistry/namd/2.13-cpu/examples/output_scw_apoa1`
* `/apps/chemistry/namd/2.13-cpu/examples/output_scw_f1atpase`
* `/apps/chemistry/namd/2.13-cpu/examples/output_scw_stmv`

The scaling tests were based on performance measurements captured as `days/ns` rather than `wallclock`, elapsed times, thereby removing the I/O overhead associated with start-up and final analysis. All three calculation show excellent scalability, with the relative performance at 320 cores increasing with the size of test case. Thus the **apoa1** simulation shows a speedup of 6.69, the **f1atpase** a speedup of 7.37 while the **stmv** case shows a speedup of 7.67, close to the linear speedup of a factor of 8.0. We note in passing that these factors are significantly greater than those found for DL_POLY (NaCl, 4.68; gramicidin, 5.27) and AMBER (M06, 3.21, M27, 3.49; M45, 3.87), albeit using very different test cases.

An in-depth analysis of the performance attributes of NAMD can be found in [^3].

<table align="center" style="width:50%">
  <tr>
    <td><img src="{{ page.root }}/fig/NAMD-benchmark-results.png" alt="NAMD-bench-results" width="100%" height="100%" /></td>
  </tr>
  <tr>
    <td><i> Figure 1 - Relative Performance of three NAMD test cases – apoa1, f1atpase and stmv – with increasing core count using Version 2.13 of the code</i></td>
  </tr>
</table>

## Further reading
* B. Acun, D. J. Hardy, L. V. Kale, K. Li, J. C. Phillips, and J. E. Stone, Scalable Molecular Dynamics with NAMD on the Summit System, IBM Journal of Research and Development, 2018. https://ieeexplore.ieee.org/document/8585036.
* James C. Phillips, What You Should Know About NAMD and Charm++ But Were Hoping to Ignore. In Proceedings of the Practice and Experience on Advanced Research Computing (PEARC '18). ACM, New York, NY, USA, Article 55, 6 pages. 2018, doi 10.1145/3219104.3219134
* Marcelo C R Melo, Rafael C Bernardi, Till Rudack, Maximilian Scheurer, Christoph Riplinger, James C Phillips, Julio D C Maia, Gerd B Rocha, João V Ribeiro, John E Stone, Frank Neese, Klaus Schulten, & Zaida Luthey-Schulten, NAMD goes quantum: an integrative suite for hybrid simulations, Nature Methods, volume 15, pages 351-354, 2018, https://www.nature.com/articles/nmeth.4638
* Bilge Acun, Ronak Buch, Laxmikant Kale, and James C. Phillips, NAMD: Scalable Molecular Dynamics Based on the Charm++ Parallel Runtime System. https://charm.cs.illinois.edu/papers/17-12
* Exascale Scientific Applications: Scalability and Performance Portability, Tjerk P. Straatsma, Katerina B. Antypas, and Timothy J. Williams, editors, CRC Press, 2017.

__________________________________________________

[^1]: http://www.ks.uiuc.edu/Research/namd/

[^2]: http://charm.cs.illinois.edu/

[^3]: http://www.ks.uiuc.edu/Research/namd/benchmarks/

[^4]: http://charm.cs.illinois.edu/papers/14-17

[^5]: http://www.ks.uiuc.edu/Research/vmd/

[^6]: http://www.ks.uiuc.edu/Research/namd/license.html

[^7]: http://www.ks.uiuc.edu/Research/namd/development.html

[^8]: https://www.ks.uiuc.edu/Development/Download/download.cgi?PackageName=NAMD

[^9]: A variety of useful tutorials are available at http://www.ks.uiuc.edu/Training/Tutorials/


{% include links.md %}

