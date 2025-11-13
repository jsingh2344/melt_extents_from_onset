# Generating Melt Extents and Snowlines from Onsets

## 11/6 - 11/13 update:


- Main issue was loading enough images from the datacube into memory at the same time to run functions on. Google Colab RAM goose chase was not fruitful. Currently chunking the datacube into 500x500 pixel sections, which equate to ~450 MB each.
  Some glaciers will be left out if they are on the boundary of chunks. Could clip directly to glacier bounding boxes and iterate through each glacier, or
  have the chunks overlap enough so that each glacier will be entirely contained in at least one chunk.
- Melt onsets are being triggered by a pixel's backscatter exceeding 3 dB from the winter mean. Melt extent is then calculated at any given
  time index by finding the number of pixels n that have onset up until that point in the current year, and then indexing into a flattened DEM with that n. Every pixel
  below that elevation is then deemed to be melting.
- Figs comparing this melt extent estimate to the previous workflow estimates are in me_cmp_figs. Each glacier, with above caveats, in path-frame 14-387 is included.
- A pixel triggers its 'snowline onset' by 1) having already begun its melt onset, and 2) by exceeding 4 dB from the 5th percentile summer backscatter.
  Will edit the 1) logic so that a pixel must only have been deemed 'melting'. Currently, some pixels at lower elevation anomalously never hit the 3 dB trigger,
  which incorrectly prevents them from ever having its 'snowline onset'.
  
