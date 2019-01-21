# AutoStatsQ
Toolbox for automated station quality control for MT inversion 
Please contact me for further description, help or when something does not behave as expected ;-) : gesap@gfz-potsdam.de

- Catalog search for teleseismic events with uniform azimuthal coverage around array
- Download of data & metadata for these events + computation of synthetic data
- Relative gain factors in time domain (relative to reference station or to synthetic data)
- Rayleigh wave polarization analysis for detection of sensor misorientations
- Comparison of obs. and synth. PSDs; determining frequency ranges suitable for MT inversion
- Test for large timing errors (resolution depends on sampling rate of synthetic data)


Latest changes
-------------
- new simple test for lare timing errors (not yet sufficiently tested...)
- gain test: additional method comparing max A to synthetics
- added SNR event filter for gain test
- added debug mode for a nice to gain and orientation test window selection experience;) can also be used to test filter settings
- all tests can now deal with multiple locations at one station


Requirements
------------

- python3
- Seismology toolbox pyrocko: https://pyrocko.org/ (Heimann et al. 2017) (and all requirements needed for pyrocko)
- To compute synthetic data a pre-calculated GF database can be downloaded using `fomosto download` from the pyrocko environment. http://kinherd.org:8080/gfws/static/stores/


Tested on Ubuntu 16.04 and openSUSE, using matplotlib version 1.5.1 and gmt version 5.4.2 (Wessel et al., 2013).

Download and Installation
-------------------------

- cd into the folder where you want to do the installation
- git clone https://github.com/gesape/AutoStatsQ
- cd AutoStatsQ
- (sudo) python3 setup.py install

Update
------
... AutoStatsQ is updated whenever new ideas are implemented or bugs found...

- cd into your AutoStatsQ installation directory
- git pull origin master
- (sudo) python3 setup.py install

Basic commands
--------------

show basic commands/ help:

	autostatsq -h


generate an example config file:

	autostatsq --generate_config GENERATE_CONFIG

run 

	autostatsq --config name_of_config_file --run RUN


Config file settings
--------------------

The default config file should look like this (without the comments):

```yaml
--- !autostatsq.config.AutoStatsQConfig
Settings:
- !autostatsq.config.GeneralSettings
  data_dir: /some/data/directory/
  list_station_lists: [/path/to/station-file/file.csv, /path/to/station-file/file.xml]

- !autostatsq.config.CatalogConfig
  search_events: true 
  # search gCMT catalog for events?

  use_local_catalog: false
  # Or use a local (already downloaded) catalog? 
  # Needed re-runs using same catalog.

  subset_of_local_catalog: true
  # Find a subset of the full catalog?

  use_local_subsets: false
  # Use local (already saved) subset instead?
  
  subset_fns: {}
  # if so, give here paths to subset-catalog-files: e.g. 
  # {'deep': 'catalog_deep_subset.txt',
  # 'shallow': 'catalog_shallow_subset.txt'}

  catalog_fn: catalog.txt
  # filename of catalog that is downloaded or already exists

  min_mag: 6.5
  max_mag: 8.5
  tmin_str: '2000-01-01 00:00:00'
  tmax_str: '2018-10-01 00:00:00'
  min_dist_km: 1000.0
  max_dist_km: 9999999.9  
  depth_options:
    deep: [25000, 1000000] # [m]
    shallow: [100, 40000] # [m]

  wedges_width: 15
  # backazimuthal step for subset generation
  # adjust to get more events, especially if time range is small

  mid_point: [46.98, 10.74]
  # give a rough estimate of midpoint of array/ network

  ### catalog plotting options ###
  dist: 165.0
  # max. distance (degrees) for catalog plot  
  plot_catalog_all: false
  # plots entire catalog on a map

  plot_hist_wedges: false
  plot_wedges_vs_dist: false
  plot_wedges_vs_magn: false
  plot_dist_vs_magn: false
  # catalog statistics plot
  
  plot_catalog_subset: false
  # plots the subset(s) on a map

- !autostatsq.config.ArrTConfig
  calc_first_arr_t: true
  # Should first arrivals be computed?

  phase_select: P|p|P(cmb)P(icb)P(icb)p(cmb)p|P(cmb)Pv(icb)p(cmb)p|P(cmb)P<(icb)(cmb)p
  # which phases?

  calc_est_R: true
  # compute arrival time of Rayleigh waves for each station-event pair? 
  # (needed for orientation test)


- !autostatsq.config.CakeTTTGenerator
  # uses travel time tables instead of cake (faster, but settings more difficult)
  # if you are not familiar with fomosto' travel time tables use ArrTConfig section 
  # instead! (for settings see src/config.py)
  calc_ttt: false


- !autostatsq.config.MetaDataDownloadConfig
  # download of metadata and data

  download_data: false
  download_metadata: false 
  use_downmeta: false
  # Set to true if downloaded metadata should be used.

  # local_metadata: [stations.xml]
  # list of local metadata files (uncomment if needed)
  # local_data: [./data]
  # list with paths to local data (uncomment if needed)
  # this option works but might be very slow in case of large amounts of data. 
  # working on it...

  components_download: HH*
  # '*' would download all and analyse the most broadband channel for each
  # station
  token:
    geofon: /path/to/token/token.asc
  # delete token-dictionary, if no token needed for fdsn query
  sites: [geofon, orfeus, iris]

  dt_start: 0.1
  # start time before origin time [h]
  dt_end: 1.5
  # end time after origin time [h]

- !autostatsq.config.RestDownRotConfig
  # restitution, downsampling and rotation of data
  # required for all tests
  rest_data: false
  freqlim: [0.005, 0.01, 0.2, 0.25] # [Hz]
  rotate_data: false
  deltat_down: 2 [s]
  # set deltat_down to 0.0 if no downsampling is wanted. (This will slow down everything,
  # and the PSD-test does only work if the sampling freuqency of synthtic and real data is 
  # the same.)

- !autostatsq.config.SynthDataConfig
  # computation of synthetic data
  # needed for PSD-test only, can otherwise be left out
  make_syn_data: false
  engine_path: /path/to/GF_stores
  store_id: global_2s

- !autostatsq.config.GainfactorsConfig
  # settings for first test
  calc_gainfactors: false
  gain_factor_method:
  - reference_nsl
  - [GE, MATE]
  ### describe different methods
  fband:
    corner_hp: 0.01 # [Hz]
    corner_lp: 0.2 # [Hz]
    order: 4
  taper_xfrac: 0.25 # [s]

  wdw_st_arr: 5
  wdw_sp_arr: 60
  # time window around P phase onset, start [s] before and end [s] after theo. arrival time

  snr_thresh: 5. # threshold for snr of used event
  debug_mode: false # if true, time windows are opened in snuffler to check window settings.

  phase_select: first(P|p|P(cmb)P(icb)P(icb)p(cmb)p|P(cmb)Pv(icb)p(cmb)p|P(cmb)P<(icb)(cmb)p)
  components: [Z, R, T]

  # plotting options
  plot_median_gain_on_map: false
  plot_allgains: false

- !autostatsq.config.PSDConfig
  # settings for PSD test
  calc_psd: false
  tinc: 600  # [s]
  tpad: 200  # [s]
  dt_start: 60  # [s] start before arrival of first P phases
  dt_end: 120  # [s] end before arrival of Rayleigh waves
  n_poly: 25
  norm_factor: 50
  f_ign: 0.02  # [Hz]
  only_first: true # outputs only first "flat" frequency range
  plot_psds: false
  plot_ratio_extra: false
  plot_m_rat: false
  plot_flat_ranges: false

- !autostatsq.config.OrientConfig
  # settings for orientation test
  orient_rayl: false
  bandpass: [3.0, 0.01, 0.05]  # [Hz]
  start_before_ev: 30.0  # start befor theo. Rayleigh wave arrival, [s]
  stop_after_ev: 480.0  # end after theo. Rayleigh wave arrival, [s]
  ccmin: 0.80
  # min. cross-correlation value. results below this value will not be
  # considered

  plot_heatmap: false
  # plot correction angle vs. cross-correlation value as imshow heatmap
  # usually distibution plot is better.
  plot_distr: false
  # plot correction angle vs. cross-correlation value
  # usually distibution plot is better.
  plot_orient_map_fromfile: false
  # plot a map with correction angles as lines
  plot_angles_vs_events: false
  # plot angle vs single events, one plot for each station

- !autostatsq.config.maps
  # settings for all output maps
  map_size: [30.0, 30.0]
  pl_opt: [46, 11.75, 800000]
  # mid point of map (lat, lon) and radius [m]
  pl_topo: false
  # plotting topography can be very slow,
  # topographic data will be downloaded first
```


Station list:
-------------

Lists of stations as input can be in station-xml format or as comma-spread-file with columns: network code, station code, latitude (float), longitude(float), station elevation [km], station depth [km].

Step-by-step instructions:
--------------------------

- make a working directory
- prepare station list for input
- generate a template config file (`autostatsq --generate_config GENERATE_CONFIG`) and adjust the settings
- it might be helpful to not run the toolbox in one run, but go though it step-by-step, setting the current step to `true` and the others to `false`... (`autostatsq --config my_config_file --run RUN`)

Output:
-------

Gain test:
- yaml files with median, mean and standard deviation of Ai,j/Ai,ref; i: event, j: station
- csv files with Ai,j/Ai,ref for all events and all stations
- optional: maps showing median log. Ai,j/Ai,ref for all stations and components
- optional: ...

Orientation test:
- yaml file with median, mean and standard deviation of obtained correction angle
- yaml file with polarity errors
- yaml file containing all results for all stations and single events
- optional: map showing the median correction angle; figures showing cc value vs correction angle for each event and station; figure showing correction angle over time for each station

PSD test:
- flat freq ranges in yaml file for each station and component
- optional: synth. and real PSDs
- optional: plots showing fit through PSD ratios 


References:
-----------

Wessel, P., Smith, W. H.~F., Scharroo, R., Luis, J.~F., and Wobbe, F. (2013).
Generic Mapping Tools: Improved version released. EOS Trans. AGU, 94:409--410.

Heimann, S., Kriegerowski, M., Isken, M., Cesca, S., Daout, S., Grigoli, F.,
Juretzek, C., Megies, T., Nooshiri, N., Steinberg, A., Sudhaus, H.,
Vasyura-Bathke, H., Willey, T., and Dahm, T. (2017).
Pyrocko - an open-source seismology toolbox and library. v. 0.3. GFZ Data Services.
http://doi.org/10.5880/GFZ.2.1.2017.001.