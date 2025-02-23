.. _pipelines:

***************************
The Long Baseline Pipelines
***************************

.. _overview:

==================
Pipelines overview
==================

    .. image:: long_baseline_workflow_sketch.png
    
The **long baseline pipeline** is organized in two major parts to process **LOFAR** data with international stations:

``LB-Delay-Calibration``
    infos to be added
``LB-Split-Calibrators`` including ``loop3``
    infos to be added


.. _delay:

=================================
Direction independent calibration
=================================
This section will describe the steps performed by ``LB-Delay-Calibration.parset``. Within the parset, the following major steps are performed:

   1. Initial setup
   2. Target preparation
   3. A-team clipping
   4. AOFlagging [toggle]
   5. Subband concatenation
   6. Application of ddf-pipeline DI solutions [toggle]
   7. Self-calibration on the best in-field calibrator [toggle]
   8. Application of self-calibration solutions [toggle]

Options marked with ``[toggle]`` can be turned on or off by the user. Other steps are always executed.

Initial setup
=============
Initial setup consists of the following steps::

   mk_results_dir
   mk_inspect_dir
   mk_cal_values_dir
   createmap_target
   createmap_target_list
   cp_cal_solutions
   download_cats

``mk_results_dir``: creates the directory where results will be stored.

``mk_inspect_dir``: creates the directory where inspection plots will be stored.

``mk_cal_values_dir``: creates the directory where calibration tables will be stored.

``createmap_target``: creates a mapfile for the target subbands.

``createmap_target_list``: creates a mapfile containing all target subbands as one big list.

``cp_cal_solutions``: copies the prefactor calibration solutions to the inspection directory.

``download_cats``: downloads LoTSS and LBCS catalogues for the field.

Target preparation
==================

Target preparation consists of the following steps::

   h5parm_add_IS
   ndppp_prep_target
   ndppp_prep_target_list

``h5parm_add_IS``: adds dummy entries for the international stations to the solutions from prefactor. Phases are initialized as 0 and amplitudes as 1.

``ndppp_prep_target``: copies over the target data and applies the clock, polalign and bandpass corrections from prefactor, the beam, rotation measure corrections and the TGSS phase solutions.

``ndppp_prep_target_list``: creates a single mapfile pointing to all the target subbands.


A-team clipping
===============

A-team clipping consists of the following steps::

   create_ateam_model_map
   make_sourcedb_ateam
   expand_sourcedb_ateam
   predict_ateam
   ateamcliptar

``create_ateam_model_map``: creates a mapfile pointing to the A-team skymodel.

``make_sourcedb_ateam``: converts the skymodels into DP3 readable sourcedb.

``expand_sourcedb_ateam``: create a mapfile linking each subband to the skymodel.

``predict_ateam``: predict the A-team sources into MODEL_DATA.

``ateamcliptar``: flag data that is affected by the A-team. This is the same clipping as used in prefactor.

AOFlagging
==========
An extra round of AOFlagger can be run if the user enabled this. AOFlagging consists of the following steps::

   ms_concat_target
   ms_concat_target_map
   aoflag 

``ms_concat_target``: virtually concatenate the target subbands.

``ms_concat_target_map``: create a mapfile pointing to the virtually concatenated measurement sets.

``aoflag``: run AOFlagger on the data.

Concatenation
=============
Subbands will be concatenated into blocks of 10 for further processing. This consists of the following steps::

   sort_concatmap
   do_sortmap_maps
   dpppconcat
   dpppconcat_list

``sort_concatmap``: sorts the subbands by frequencies and adds in dummy entries for any missing subbands (to ensure regular frequency spacing).

``do_sortmap_maps``: create a usable mapfile from the previous step.

``dpppconcat``: runs DP3 to concatenate subbands and create blocks of 10.

``dpppconcat_list``: creates a mapfile pointing to the concatenated data.

Application of ddf-pipeline solutions
=====================================
In this optional step, the direction independent solutions obtained by the ddf-pipeline are applied to the data. This consists of the following steps::

   createmap_ddf
   ddf_solutions
   ddf_h5parms
   convert_to_h5
   addIS
   ndppp_applycal 

``createmap_ddf``: creates a mapfile pointing the pipeline to the ddf-pipeline solutions.

``ddf_solutions``: creates a mapfile of the specific DIS2 solutions.

``ddf_h5parms``: converts the solutions from killMS format to H5parms.

``addIS``: adds dummy entries for the international stations to the solutions.

``ndppp_applycal``: applies the solutions to the data. Calibrated data is stored in the ``delaycal_col`` column.


.. _split:

=================================
Direction-dependent calibration
=================================
This section will describe the steps performed by ``LB-Split-Calibrators.parset``. Within the parset, the following major steps are performed:

   1. Initial setup
   2. TEC-solve on in-field calibrators
   3. Amplitude and phase self-calibration loop

Initial setup
=============
Initial setup consists of the following steps::

   targetdata_map
   targetdata_map_list
   parallel_split
   caldata_map   
   cal_model
   h5parm_map
   sourcedb_map

``targetdata_map``: maps the location of the input target data

``targetdata_map_list``: writes all target data into a single list map

``parallel_split``: splits off data for every identified in-field calibrator, i.e., shifting the phase-centre to an in-field calibrator direction, averaging to smear out signals from sources outside of the region of interest, and coherently adding of all core stations to a superstation to increase the signal-to-noise-ratio for any upcoming solves on international station - superstation baselines

``caldata_map``: creates a mapfile for the target subbands.

``cal_model``: creates a mapfile containing all target subbands as one big list.

``h5parm_map``: copies the prefactor calibration solutions to the inspection directory.

``sourcedb_map``: downloads LoTSS and LBCS catalogues for the field.

TEC-solve on in-field calibrators
=================================

After the initial setup there is a TEC-solve running on each individual in-field calibrator data set::

   tecsolve
   plot_tec
   apply_tec

``tecsolve``: solves for TEC for each individual in-field long-baseline calibrator

``plot_tec``: provides diagnostic plots of the TEC solutions for each individual in-field calibrator

``apply_tec``: applies the derived solutions from each individual in-field calibrator to the corresponding data set


Amplitude and phase self-calibration loop
=========================================

The amplitude and phase self-calibration loop consists of the following steps::

   cal_loop3

``cal_loop3``: loop3 is a self-calibration routine for amplitude and phase-corrections with a selected set of strategies. It takes the TEC-corrected in-field calibrator data and provides a set of ``h5parm`` solution sets for each individual in-field calibrator.
